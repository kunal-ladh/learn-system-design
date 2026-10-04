# API Protocols Deep Dive — SDE2 Interview Guide

> **Goal:** master every modern API protocol you'll meet in an SDE2 interview — what it is, when to use it, how it works on the wire, trade-offs, code in JS/TS and Java/Spring Boot, and the interview gotchas senior engineers love to probe.
>
> **Prereq:** you know REST (verbs, status codes, JSON). Everything below is built relative to that.

---

## Table of contents

1. [The mental model: what an "API protocol" actually is](#1-the-mental-model)
2. [REST refresher (so we can compare against it)](#2-rest-refresher)
3. [GraphQL](#3-graphql)
4. [gRPC](#4-grpc)
5. [WebSocket](#5-websocket)
6. [Webhook](#6-webhook)
7. [Server-Sent Events (SSE)](#7-server-sent-events-sse)
8. [HTTP/2, HTTP/3 and QUIC](#8-http2-http3-and-quic)
9. [MQTT](#9-mqtt)
10. [SOAP (the museum exhibit)](#10-soap-brief)
11. [Comparison matrix](#11-comparison-matrix)
12. [Decision framework: which protocol when](#12-decision-framework)
13. [Cross-cutting concerns (caching, idempotency, versioning, auth)](#13-cross-cutting-concerns)
14. [SDE2 interview questions with deep answers](#14-sde2-interview-questions)

---

## 1. The mental model

Every API protocol is just three choices stacked on top of each other:

| Layer | Choices | Examples |
|---|---|---|
| **Transport** | TCP, UDP, QUIC | TCP for REST/gRPC/WebSocket, UDP for QUIC, MQTT over TCP |
| **Framing / wire format** | Text vs binary, request/response vs streams vs frames | JSON, XML, Protobuf, MQTT control packets |
| **Semantics** | RPC, resource-CRUD, query, pub/sub, push | REST = resource; gRPC = RPC; MQTT = pub/sub |

When someone asks "REST vs gRPC?" they're really asking: *given my latency/throughput/ergonomics needs, what stack do I want?*

A second axis worth keeping in mind:

- **Who initiates?** Client → server (REST, GraphQL, gRPC unary, SOAP). Server → client (Webhook, SSE, gRPC server-streaming, MQTT publish). Both ways at any time (WebSocket, gRPC bidi, MQTT).
- **Connection lifecycle:** short-lived (REST, GraphQL, webhook) vs long-lived (WebSocket, SSE, gRPC streams, MQTT).
- **Cardinality:** 1:1 (REST), 1:N broadcast (WebSocket rooms, SSE, pub/sub), N:M (MQTT topics).

Hold this mental model — every protocol below maps onto it.

---

## 2. REST

### What it is

**REST** (Representational State Transfer) was defined in Roy Fielding's 2000 PhD dissertation as an *architectural style* (not a protocol). The mental model: everything is a **resource** identified by a URI; clients exchange **representations** of resources (JSON, XML, HTML) using a **uniform interface** of HTTP verbs; servers are **stateless** (each request carries everything needed); responses are **cacheable**; the system is **layered** (proxies, CDNs, gateways are invisible to clients).

In practice "REST" in 2026 means: JSON over HTTP/1.1+, with HTTP verbs mapping to CRUD on URI-addressed resources. Most "REST APIs" violate at least one of Fielding's constraints (especially HATEOAS) — Fielding himself has griped about this. Use the term pragmatically.

You know REST, but interviews will probe how *deeply* you know it. The bits that catch SDE1s off guard:

### Verbs vs CRUD ≠ the same thing

- `POST` ≠ "create." It's "non-idempotent action on a collection." Useful for searches with complex bodies (`POST /searches`), batch ops, etc.
- `PUT` is **full replacement** and **idempotent**. Sending the same `PUT` twice must produce the same end state.
- `PATCH` is **partial update**. Not necessarily idempotent (e.g., `{"counter": {"increment": 1}}` is not idempotent; `{"counter": 5}` is).
- `DELETE` is idempotent. Deleting an already-deleted resource returns 204 or 404, but the *state* doesn't change.

### Status code traps

| Code | When |
|---|---|
| 200 | Success with body |
| 201 | Created — include `Location` header |
| 202 | Accepted (async — work queued, not done) |
| 204 | Success, no body |
| 400 | Client validation error |
| 401 | Not authenticated (missing/expired token) |
| 403 | Authenticated but not authorized |
| 404 | Doesn't exist (or hide a 403 for security) |
| 409 | Conflict (e.g., version mismatch, duplicate) |
| 410 | Gone — explicitly deleted, won't come back |
| 422 | Semantically wrong (well-formed JSON but invalid business rules) |
| 429 | Rate limited — must include `Retry-After` |
| 500 | Server bug |
| 502 | Bad gateway (upstream returned garbage) |
| 503 | Service unavailable (overload, maintenance) |
| 504 | Gateway timeout |

Don't conflate 401 vs 403 — interviewers love this.

### Caching is what makes REST cheap

- `Cache-Control: public, max-age=300, s-maxage=600` (browser 5 min, CDN 10 min).
- `ETag` / `If-None-Match` for conditional GETs → 304 Not Modified.
- `Last-Modified` / `If-Modified-Since` (weaker — second precision).
- `Vary: Accept-Encoding, Authorization` so caches don't mix users.

> **Gotcha:** GraphQL and gRPC lose HTTP-cache for free because everything goes through one URL/POST. You have to re-build caching at the application or transport layer.

### HATEOAS (rarely used in practice, often asked)

Level 3 of the Richardson Maturity Model: responses include links to next actions.

```json
{
  "id": 42,
  "status": "shipped",
  "_links": {
    "self":   { "href": "/orders/42" },
    "cancel": { "href": "/orders/42/cancel" },
    "track":  { "href": "/orders/42/tracking" }
  }
}
```

Interviewers ask about it. Real APIs (Stripe, GitHub) implement *some* HATEOAS for pagination but rarely fully. Be honest about that.

### Versioning

- **URI** (`/v1/users`) — simple, ugly.
- **Header** (`Accept: application/vnd.acme.v2+json`) — clean, harder for clients.
- **Query param** (`?v=2`) — hacky but pragmatic.

Stripe pins versions per API key. GitHub uses `Accept` headers. Pick a strategy and *stick* to it.

### Resource modeling (the real REST skill)

Naming resources is where most REST APIs go wrong. The rules:

| Rule | Good | Bad |
|---|---|---|
| Plural nouns for collections | `GET /orders` | `GET /getOrders` |
| Hierarchies for ownership | `GET /users/42/orders` | `GET /orders?user=42` (also fine, see below) |
| Verbs only for non-CRUD actions | `POST /orders/42/cancel` | `POST /cancelOrder` |
| Lower-case, hyphens for words | `/billing-accounts` | `/billingAccounts` or `/billing_accounts` |
| Stable IDs in path | `/orders/{id}` | `/orders/{shipping-status}/{id}` |

**Nested vs flat:** nest when the child can't exist without the parent (`/orders/{id}/items` — an item has no meaning outside its order). Use flat with filters when both are independent (`/comments?postId=42&userId=7`).

**Action endpoints** (non-CRUD verbs on resources) are fine and pragmatic:
```
POST /orders/42/cancel        # transition
POST /accounts/9/transfer     # operation with parameters
POST /pdfs/123/sign           # side-effect-y action
```
Some shops insist on representing actions as sub-resources (`PUT /orders/42/status` with `{"status":"cancelled"}`). Both are defensible — be ready to explain your call.

### Filtering, sorting, sparse fieldsets, search

These belong in the query string, not the path:

```
GET /orders?status=paid&min_total=100         # filter
GET /orders?sort=-created_at,total            # sort: minus = desc
GET /orders?fields=id,total,status            # sparse fieldset
GET /orders?include=customer,items.product    # eager-loaded relations
GET /orders?q=watch                           # full-text search (or use /search?q=)
```

JSON:API and Stripe's API both standardize these conventions. If you're greenfield, lean on JSON:API for filter/sort/include grammar — it saves you bikeshedding.

### Pagination (REST-specific patterns)

The cross-cutting section has the algorithm comparison. Here's the **API surface** for each:

```http
# Offset
GET /orders?page=3&size=20
Link: </orders?page=4&size=20>; rel="next", </orders?page=1&size=20>; rel="first"

# Cursor (Stripe, Twitter)
GET /orders?starting_after=ord_42&limit=20
→ { "data": [...], "has_more": true, "next_cursor": "ord_98" }

# Keyset (best for big tables)
GET /orders?after_id=42&after_created_at=2026-01-01T00:00:00Z&limit=20

# Link header (RFC 5988 — GitHub style)
Link: <https://api.github.com/repos?page=2>; rel="next",
      <https://api.github.com/repos?page=5>; rel="last"
```

Default to **cursor or keyset** for write-heavy collections — offset pagination silently skips/repeats rows when concurrent inserts happen.

### Conditional requests & optimistic concurrency

Avoid lost-update races with `ETag` + `If-Match`:

```http
GET /orders/42
→ HTTP/1.1 200 OK
  ETag: "abc123"
  { "id": 42, "status": "pending", ... }

PATCH /orders/42
If-Match: "abc123"
{ "status": "shipped" }

→ 412 Precondition Failed     # someone else updated since you read
→ 200 OK + new ETag            # your update wins
```

Also: `If-None-Match: *` on `PUT /orders/42` = "create only if doesn't exist." `If-None-Match: "abc"` on `GET` = "send body only if changed" → 304 Not Modified.

This is the textbook fix to "the read-modify-write race." Interviewers love this for SDE2.

### Bulk operations

REST has no bulk-update standard. Three pragmatic options:

```http
# 1. Custom action endpoint
POST /orders/bulk-update
[{ "id": 1, "status": "shipped" }, { "id": 2, "status": "cancelled" }]

# 2. PATCH with JSON Patch (RFC 6902)
PATCH /orders/42
Content-Type: application/json-patch+json
[
  { "op": "replace", "path": "/status", "value": "shipped" },
  { "op": "add",     "path": "/tags/-", "value": "priority" }
]

# 3. Async batch endpoint (returns 202)
POST /batches
{ "operations": [...] }
→ 202 Accepted + Location: /batches/b_42
GET /batches/b_42  → poll for result
```

For bulk reads, use filtered list endpoints. Avoid `/users?ids=1,2,3,4,...` past ~50 IDs — URL length limits + cache poisoning risk; switch to `POST /users/lookup` with a body.

### Content negotiation

```http
GET /orders/42
Accept: application/json
Accept-Language: en-US, en;q=0.9
Accept-Encoding: gzip, br

→ Content-Type: application/json; charset=utf-8
  Content-Language: en-US
  Content-Encoding: gzip
  Vary: Accept, Accept-Encoding   # tells caches to key on these
```

Less common but useful: `Accept: application/csv` for exports, `application/xml` for legacy clients, `application/problem+json` (RFC 7807) for structured errors:

```json
{
  "type": "https://acme.com/errors/insufficient-balance",
  "title": "Insufficient balance",
  "status": 402,
  "detail": "Account 9 has balance 12.50 but requires 100.00",
  "instance": "/accounts/9/transfers/tx_42"
}
```

RFC 7807 is the closest thing REST has to a "standard error envelope." Use it.

### Sample code

#### JS/TS (Express + Zod) — full CRUD with validation, errors, pagination

```ts
// schema.ts
import { z } from 'zod';

export const CreateOrder = z.object({
  userId: z.string().uuid(),
  total: z.number().positive(),
  items: z.array(z.object({
    sku: z.string().min(1),
    qty: z.number().int().positive(),
  })).min(1),
});
export type CreateOrder = z.infer<typeof CreateOrder>;

export const UpdateOrder = z.object({
  status: z.enum(['pending', 'paid', 'shipped', 'cancelled']).optional(),
  total: z.number().positive().optional(),
}).strict();
```

```ts
// orders.routes.ts
import { Router } from 'express';
import { z } from 'zod';
import crypto from 'crypto';
import { CreateOrder, UpdateOrder } from './schema';
import { db } from './db';

const r = Router();

// LIST with cursor pagination + filters + sparse fieldsets
r.get('/orders', async (req, res, next) => {
  try {
    const q = z.object({
      status: z.enum(['pending','paid','shipped','cancelled']).optional(),
      after:  z.string().optional(),
      limit:  z.coerce.number().int().min(1).max(100).default(20),
      fields: z.string().optional(),
    }).parse(req.query);

    const rows = await db.orders.list({
      status: q.status, after: q.after, limit: q.limit + 1,
    });
    const hasMore = rows.length > q.limit;
    const data = (hasMore ? rows.slice(0, -1) : rows)
      .map(o => q.fields ? pick(o, q.fields.split(',')) : o);

    res.set('Cache-Control', 'private, max-age=10');
    res.json({
      data,
      has_more: hasMore,
      next_cursor: hasMore ? data[data.length - 1].id : null,
    });
  } catch (e) { next(e); }
});

// GET one — with ETag for conditional requests
r.get('/orders/:id', async (req, res, next) => {
  try {
    const order = await db.orders.findById(req.params.id);
    if (!order) return next({ status: 404, type: 'order-not-found' });

    const etag = `"${crypto.createHash('sha1')
      .update(JSON.stringify(order)).digest('hex')}"`;
    if (req.header('If-None-Match') === etag) return res.status(304).end();

    res.set('ETag', etag);
    res.set('Cache-Control', 'private, max-age=60');
    res.json(order);
  } catch (e) { next(e); }
});

// CREATE — with Idempotency-Key support
r.post('/orders', async (req, res, next) => {
  try {
    const body = CreateOrder.parse(req.body);
    const idemKey = req.header('Idempotency-Key');

    if (idemKey) {
      const cached = await db.idempotency.get(idemKey);
      if (cached) return res.status(cached.status).json(cached.body);
    }

    const order = await db.orders.create(body);

    if (idemKey) {
      await db.idempotency.set(idemKey, { status: 201, body: order }, 86400);
    }

    res.status(201)
       .set('Location', `/orders/${order.id}`)
       .json(order);
  } catch (e) { next(e); }
});

// UPDATE — partial, with optimistic concurrency
r.patch('/orders/:id', async (req, res, next) => {
  try {
    const body = UpdateOrder.parse(req.body);
    const ifMatch = req.header('If-Match');

    const current = await db.orders.findById(req.params.id);
    if (!current) return next({ status: 404, type: 'order-not-found' });

    const currentETag = `"${crypto.createHash('sha1')
      .update(JSON.stringify(current)).digest('hex')}"`;
    if (ifMatch && ifMatch !== currentETag) {
      return next({ status: 412, type: 'precondition-failed' });
    }

    const updated = await db.orders.update(req.params.id, body);
    const newETag = `"${crypto.createHash('sha1')
      .update(JSON.stringify(updated)).digest('hex')}"`;
    res.set('ETag', newETag).json(updated);
  } catch (e) { next(e); }
});

// REPLACE — idempotent full replacement
r.put('/orders/:id', async (req, res, next) => {
  try {
    const body = CreateOrder.parse(req.body);
    const order = await db.orders.upsert(req.params.id, body);
    res.json(order);
  } catch (e) { next(e); }
});

// DELETE — idempotent
r.delete('/orders/:id', async (req, res, next) => {
  try {
    await db.orders.delete(req.params.id);
    res.status(204).end();
  } catch (e) { next(e); }
});

// Action endpoint — verbs are fine for non-CRUD ops
r.post('/orders/:id/cancel', async (req, res, next) => {
  try {
    const reason = z.object({ reason: z.string().max(500) }).parse(req.body);
    const order = await db.orders.cancel(req.params.id, reason.reason);
    res.json(order);
  } catch (e) { next(e); }
});

export default r;
```

```ts
// errors.ts — RFC 7807 Problem Details
export function problemHandler(err, req, res, _next) {
  if (err.name === 'ZodError') {
    return res.status(400).type('application/problem+json').json({
      type: 'https://api.acme.com/errors/validation',
      title: 'Request validation failed',
      status: 400,
      detail: 'See `errors` for field-level info.',
      instance: req.originalUrl,
      errors: err.issues.map(i => ({ path: i.path.join('.'), message: i.message })),
    });
  }
  const status = err.status ?? 500;
  res.status(status).type('application/problem+json').json({
    type:    `https://api.acme.com/errors/${err.type ?? 'internal'}`,
    title:   err.title ?? 'Server error',
    status,
    detail:  err.message,
    instance: req.originalUrl,
  });
}
```

```ts
// app.ts
import express from 'express';
import orders from './orders.routes';
import { problemHandler } from './errors';

const app = express();
app.use(express.json({ limit: '1mb' }));
app.use('/v1', orders);
app.use(problemHandler);
app.listen(3000);
```

#### Java / Spring Boot — full CRUD with DTOs, validation, ProblemDetail

`pom.xml` essentials: `spring-boot-starter-web`, `spring-boot-starter-validation`, `spring-boot-starter-data-jpa`.

```java
// dto/CreateOrderRequest.java
public record CreateOrderRequest(
    @NotNull UUID userId,
    @Positive BigDecimal total,
    @NotEmpty @Valid List<OrderItemDto> items
) {}

public record OrderItemDto(
    @NotBlank String sku,
    @Positive int qty
) {}

public record UpdateOrderRequest(
    @Pattern(regexp = "pending|paid|shipped|cancelled") String status,
    @Positive BigDecimal total
) {}

public record OrderResponse(
    UUID id, UUID userId, String status, BigDecimal total,
    List<OrderItemDto> items, Instant createdAt, Instant updatedAt,
    long version
) {}

public record PageResponse<T>(
    List<T> data, boolean hasMore, String nextCursor
) {}
```

```java
// entity/Order.java
@Entity @Table(name = "orders")
public class Order {
  @Id @GeneratedValue private UUID id;
  private UUID userId;
  private String status;
  private BigDecimal total;

  @Version            // JPA-managed optimistic lock → ETag source
  private long version;

  @CreatedDate  private Instant createdAt;
  @LastModifiedDate private Instant updatedAt;
  // ... items, getters
}
```

```java
// OrdersController.java
@RestController
@RequestMapping("/v1/orders")
@Validated
public class OrdersController {

  private final OrderService service;
  public OrdersController(OrderService s) { this.service = s; }

  @GetMapping
  public PageResponse<OrderResponse> list(
      @RequestParam(required = false) String status,
      @RequestParam(required = false) String after,
      @RequestParam(defaultValue = "20") @Min(1) @Max(100) int limit) {
    return service.list(status, after, limit);
  }

  @GetMapping("/{id}")
  public ResponseEntity<OrderResponse> get(
      @PathVariable UUID id,
      @RequestHeader(value = "If-None-Match", required = false) String ifNoneMatch) {
    var order = service.findById(id);
    var etag = "\"" + order.version() + "\"";
    if (etag.equals(ifNoneMatch)) {
      return ResponseEntity.status(HttpStatus.NOT_MODIFIED).eTag(etag).build();
    }
    return ResponseEntity.ok().eTag(etag)
        .cacheControl(CacheControl.maxAge(60, TimeUnit.SECONDS).cachePrivate())
        .body(order);
  }

  @PostMapping
  public ResponseEntity<OrderResponse> create(
      @Valid @RequestBody CreateOrderRequest req,
      @RequestHeader(value = "Idempotency-Key", required = false) String idemKey) {
    var order = service.create(req, idemKey);
    return ResponseEntity
        .created(URI.create("/v1/orders/" + order.id()))
        .eTag("\"" + order.version() + "\"")
        .body(order);
  }

  @PatchMapping("/{id}")
  public ResponseEntity<OrderResponse> patch(
      @PathVariable UUID id,
      @Valid @RequestBody UpdateOrderRequest req,
      @RequestHeader(value = "If-Match", required = false) String ifMatch) {
    var order = service.update(id, req, ifMatch);
    return ResponseEntity.ok().eTag("\"" + order.version() + "\"").body(order);
  }

  @PutMapping("/{id}")
  public ResponseEntity<OrderResponse> replace(
      @PathVariable UUID id,
      @Valid @RequestBody CreateOrderRequest req) {
    var order = service.upsert(id, req);
    return ResponseEntity.ok().eTag("\"" + order.version() + "\"").body(order);
  }

  @DeleteMapping("/{id}")
  public ResponseEntity<Void> delete(@PathVariable UUID id) {
    service.delete(id);
    return ResponseEntity.noContent().build();
  }

  @PostMapping("/{id}/cancel")
  public OrderResponse cancel(@PathVariable UUID id,
                              @Valid @RequestBody CancelRequest req) {
    return service.cancel(id, req.reason());
  }
}
```

```java
// OrderService.java — note the optimistic-lock translation
@Service
@Transactional
public class OrderService {

  private final OrderRepository repo;
  private final IdempotencyStore idem;

  public OrderResponse update(UUID id, UpdateOrderRequest req, String ifMatch) {
    var order = repo.findById(id)
      .orElseThrow(() -> new ResourceNotFoundException("order", id));

    if (ifMatch != null) {
      var current = "\"" + order.getVersion() + "\"";
      if (!current.equals(ifMatch)) {
        throw new PreconditionFailedException("ETag mismatch");
      }
    }

    if (req.status() != null) order.setStatus(req.status());
    if (req.total()  != null) order.setTotal(req.total());

    try {
      return toResponse(repo.save(order));
    } catch (OptimisticLockingFailureException e) {
      // Concurrent write detected by @Version
      throw new ConflictException("order was modified concurrently");
    }
  }

  public OrderResponse create(CreateOrderRequest req, String idemKey) {
    if (idemKey != null) {
      var cached = idem.get(idemKey);
      if (cached.isPresent()) return cached.get();
    }
    var saved = repo.save(toEntity(req));
    var resp  = toResponse(saved);
    if (idemKey != null) idem.put(idemKey, resp, Duration.ofHours(24));
    return resp;
  }
  // ...
}
```

```java
// GlobalExceptionHandler.java — Spring 6 / Boot 3 ProblemDetail (RFC 7807)
@RestControllerAdvice
public class GlobalExceptionHandler {

  @ExceptionHandler(ResourceNotFoundException.class)
  ProblemDetail notFound(ResourceNotFoundException e) {
    var pd = ProblemDetail.forStatusAndDetail(HttpStatus.NOT_FOUND, e.getMessage());
    pd.setType(URI.create("https://api.acme.com/errors/not-found"));
    pd.setTitle("Resource not found");
    return pd;
  }

  @ExceptionHandler(MethodArgumentNotValidException.class)
  ProblemDetail validation(MethodArgumentNotValidException e) {
    var pd = ProblemDetail.forStatusAndDetail(HttpStatus.BAD_REQUEST,
      "Request validation failed");
    pd.setType(URI.create("https://api.acme.com/errors/validation"));
    pd.setProperty("errors", e.getBindingResult().getFieldErrors().stream()
      .map(f -> Map.of("path", f.getField(), "message", f.getDefaultMessage()))
      .toList());
    return pd;
  }

  @ExceptionHandler(PreconditionFailedException.class)
  ProblemDetail preconditionFailed(PreconditionFailedException e) {
    return ProblemDetail.forStatusAndDetail(HttpStatus.PRECONDITION_FAILED, e.getMessage());
  }

  @ExceptionHandler(ConflictException.class)
  ProblemDetail conflict(ConflictException e) {
    return ProblemDetail.forStatusAndDetail(HttpStatus.CONFLICT, e.getMessage());
  }
}
```

A few things worth noting in the Spring code:

- `@Version` field doubles as both an optimistic-lock counter (JPA throws on concurrent writes) *and* the ETag value. Two birds.
- `ProblemDetail` is built into Spring 6 — no extra library needed for RFC 7807.
- `ResponseEntity.created(uri)` automatically sets `Location` (201 + Location is the textbook "I created it, here's where").
- `@Validated` + `@Valid` + `@NotNull`/`@Positive` use Jakarta Bean Validation — declarative and tested.

### Pros / cons

**Pros**
- **Ubiquitous tooling**: curl, Postman, browsers, CDNs, gateways, observability all assume REST.
- **HTTP caching is free** — ETag, Cache-Control, CDN-level reuse cuts backend load drastically.
- **Stateless** → scales horizontally with no session affinity.
- **Easy to debug** — JSON in, JSON out, plain text on the wire.
- **Polyglot clients** — every language has an HTTP client.
- **Layered architecture** — proxies, gateways, WAFs, rate limiters slot in transparently.

**Cons**
- **Over/under-fetching** — clients get exactly what the endpoint returns. Mobile apps either fetch fields they don't need or make N round trips. (Solved by GraphQL.)
- **Verbose for chatty workflows** — each call is a fresh request/response. (gRPC + streaming is faster.)
- **No native streaming** — long-poll, SSE, or WS bolted on.
- **Versioning is awkward** at scale — every public API faces v1 → v2 migration pain.
- **HATEOAS is rarely implemented** — clients hardcode URLs, breaking on path changes.
- **No type contract** by default — OpenAPI helps but isn't enforced at runtime unless you wire it.

### Interview gotchas

- **"PUT vs PATCH vs POST?"** PUT = full replace (idempotent), PATCH = partial (not always idempotent), POST = create-or-action (not idempotent unless you use Idempotency-Key).
- **"Is GET always cacheable?"** Yes by default (RFC 9111), unless `Cache-Control: no-store/private` says otherwise. POST is *not* cacheable by default, though it can be made so with `Cache-Control` — rarely worth doing.
- **"What's the difference between 401 and 403?"** 401 = "I don't know who you are" (missing/invalid creds). 403 = "I know who you are, you can't do this." Common bug: returning 401 for *both* leaks "this account exists" info.
- **"Why 422 instead of 400?"** 400 = malformed request (bad JSON, missing required header). 422 = well-formed but semantically invalid (price negative, status transition disallowed). Many APIs collapse these into 400 — fine, but be ready to defend.
- **"How do you fix the read-modify-write race?"** ETag + `If-Match` (optimistic concurrency). For DB-backed entities, use a `@Version` column. Mention both layers.
- **"Why does PATCH have its own RFCs?"** RFC 5789 defines PATCH but leaves the patch *format* open. RFC 6902 = JSON Patch (ops array), RFC 7396 = JSON Merge Patch (subset object). Different content types: `application/json-patch+json` vs `application/merge-patch+json`. Most teams use Merge Patch because it's intuitive; JSON Patch wins for array ops and conditional changes.
- **"How do you handle file uploads in REST?"** `multipart/form-data` for small files (≤ a few MB); for big files use pre-signed URLs (client uploads directly to S3/GCS with a signed URL, then PATCHes the resource with the object key). Avoid base64 in JSON — 33% bloat and OOM risk.
- **"How does gzip/Brotli fit in?"** `Accept-Encoding` request → `Content-Encoding` response. Brotli (`br`) beats gzip ~20% on JSON. Both are CPU-cheap on modern hardware. Always enable — usually a one-line config (`compression: true` in Express, `server.compression.enabled=true` in Boot).
- **"Why is HATEOAS not used in practice?"** Clients are typically generated from OpenAPI specs and hardcode endpoints. Discovery via hyperlinks adds bytes and complexity for marginal benefit. Stripe and GitHub use HATEOAS for pagination only (`Link` header, `_links.next`).
- **"What's the Richardson Maturity Model?"** L0 = one URL, RPC over POST (SOAP-ish). L1 = multiple URLs, one verb (still POST). L2 = multiple verbs, status codes (what most "REST" APIs are). L3 = HATEOAS. Knowing this lets you describe an API's design level concisely.

---

## 3. GraphQL

### What it is

A **query language and runtime** invented by Facebook (2012, OSS 2015) for *the client to ask for exactly the fields it wants, from one endpoint, in one round trip.*

```graphql
query {
  user(id: 42) {
    name
    email
    orders(last: 3) {
      id
      total
      items { sku, qty }
    }
  }
}
```

One POST to `/graphql` returns precisely that tree. No 7 REST round trips, no over-fetching of fields you didn't want.

### How it works

Three operation types:

| Op | Purpose | HTTP semantics |
|---|---|---|
| `query` | Read (idempotent) | POST (or GET with persisted queries) |
| `mutation` | Write (side effects) | POST |
| `subscription` | Real-time stream | WebSocket (`graphql-ws`) or SSE |

A **schema** (SDL — Schema Definition Language) declares types:

```graphql
type User {
  id: ID!
  name: String!
  email: String
  orders(last: Int = 10): [Order!]!
}

type Order {
  id: ID!
  total: Float!
  items: [Item!]!
}

type Query {
  user(id: ID!): User
  search(q: String!): [SearchResult!]!
}

type Mutation {
  createOrder(input: CreateOrderInput!): Order!
}

type Subscription {
  orderUpdated(id: ID!): Order!
}

union SearchResult = User | Order
```

**Resolvers** are functions that fetch data for each field. The runtime walks the query tree and calls resolvers in parallel where possible.

### The N+1 problem (the #1 GraphQL interview question)

Naive resolvers run separate queries per child:

```js
// Resolver for User.orders
orders: (user) => db.query('SELECT * FROM orders WHERE user_id = $1', [user.id])
```

Asking for 100 users fires 1 (users) + 100 (orders per user) = 101 queries. Boom.

**Fix: DataLoader** (batching + caching per request).

```js
import DataLoader from 'dataloader';

const ordersByUserId = new DataLoader(async (userIds) => {
  const rows = await db.query(
    'SELECT * FROM orders WHERE user_id = ANY($1)', [userIds]
  );
  // Group by user_id, preserve input order
  return userIds.map(id => rows.filter(r => r.user_id === id));
});

// In resolver:
orders: (user) => ordersByUserId.load(user.id)
```

Now 100 `.load()` calls coalesce into one `WHERE user_id IN (...)`. Always mention DataLoader if N+1 comes up.

### Caching is hard

REST gets HTTP cache free. GraphQL doesn't because:

1. Everything is POST to `/graphql` — no URL to key off.
2. The same logical entity can be requested with different field subsets.

Solutions:

- **GET with persisted queries**: client sends only a hash; server keeps the actual query text. Now the URL is cacheable. Apollo's "Automatic Persisted Queries" (APQ) does this.
- **Normalized client cache** (Apollo Client, Relay, urql) — stores entities by `__typename:id` and reuses fragments.
- **CDN-level cache** with persisted queries + hash-based URL.
- **Application-level cache** (Redis) keyed by entity ID.

### Federation & schema stitching

Big shops (Netflix, Airbnb) run a **federated** GraphQL: each team owns a subgraph (`users`, `payments`, `inventory`), and a **gateway** (Apollo Router) composes them into one supergraph. Solves the monorepo-of-resolvers problem.

### Pros / cons

**Pros**
- Client controls payload shape — no over/under-fetching.
- One endpoint, strongly typed schema, great tooling (GraphiQL, codegen).
- Schema becomes the contract — frontend devs love it.
- Field-level deprecation: `@deprecated(reason: "use newField")` — no v1/v2 mess.

**Cons**
- Caching is harder.
- File uploads need a multipart extension (`graphql-multipart-request-spec`).
- Easy to write expensive queries (depth, breadth, cycles) — need query cost analysis or depth limits.
- Auth/authz must be field-level, not endpoint-level.
- Errors come back as 200 OK with an `errors` array, which trips up naive HTTP-status-checking clients.

### Sample code

#### JS/TS: Apollo Server v4

```ts
// schema.ts
import { gql } from 'graphql-tag';

export const typeDefs = gql`
  type User {
    id: ID!
    name: String!
    orders: [Order!]!
  }
  type Order {
    id: ID!
    total: Float!
  }
  type Query {
    user(id: ID!): User
  }
  type Mutation {
    createOrder(userId: ID!, total: Float!): Order!
  }
  type Subscription {
    orderCreated: Order!
  }
`;
```

```ts
// resolvers.ts
import DataLoader from 'dataloader';
import { PubSub } from 'graphql-subscriptions';

const pubsub = new PubSub();
const ORDER_CREATED = 'ORDER_CREATED';

export const resolvers = {
  Query: {
    user: async (_, { id }, { db }) =>
      db.users.findById(id),
  },
  Mutation: {
    createOrder: async (_, { userId, total }, { db }) => {
      const order = await db.orders.create({ userId, total });
      pubsub.publish(ORDER_CREATED, { orderCreated: order });
      return order;
    },
  },
  Subscription: {
    orderCreated: {
      subscribe: () => pubsub.asyncIterator([ORDER_CREATED]),
    },
  },
  User: {
    orders: (user, _, { loaders }) => loaders.ordersByUserId.load(user.id),
  },
};

export const buildContext = ({ req }) => ({
  db: req.db,
  user: req.user,
  loaders: {
    ordersByUserId: new DataLoader(async (ids) => {
      const rows = await req.db.orders.findByUserIds(ids);
      return ids.map(id => rows.filter(r => r.userId === id));
    }),
  },
});
```

```ts
// server.ts
import { ApolloServer } from '@apollo/server';
import { startStandaloneServer } from '@apollo/server/standalone';
import { typeDefs } from './schema';
import { resolvers, buildContext } from './resolvers';

const server = new ApolloServer({ typeDefs, resolvers });
const { url } = await startStandaloneServer(server, {
  context: async ({ req }) => buildContext({ req }),
  listen: { port: 4000 },
});
console.log(`🚀 ${url}`);
```

Client (browser/Node):

```ts
import { ApolloClient, InMemoryCache, gql } from '@apollo/client';

const client = new ApolloClient({
  uri: 'http://localhost:4000/graphql',
  cache: new InMemoryCache(),
});

const { data } = await client.query({
  query: gql`
    query GetUser($id: ID!) {
      user(id: $id) {
        name
        orders { id total }
      }
    }
  `,
  variables: { id: '42' },
});
```

#### Java / Spring for GraphQL

Spring Boot starter: `spring-boot-starter-graphql`.

```graphql
# src/main/resources/graphql/schema.graphqls
type User {
  id: ID!
  name: String!
  orders: [Order!]!
}
type Order {
  id: ID!
  total: Float!
}
type Query {
  user(id: ID!): User
}
type Mutation {
  createOrder(userId: ID!, total: Float!): Order!
}
```

```java
// UserController.java
@Controller
public class UserController {

  private final UserRepository userRepo;
  private final OrderRepository orderRepo;

  public UserController(UserRepository u, OrderRepository o) {
    this.userRepo = u; this.orderRepo = o;
  }

  @QueryMapping
  public User user(@Argument String id) {
    return userRepo.findById(Long.valueOf(id)).orElse(null);
  }

  @MutationMapping
  public Order createOrder(@Argument String userId, @Argument Double total) {
    return orderRepo.save(new Order(Long.valueOf(userId), total));
  }

  // Batched resolver to avoid N+1
  @BatchMapping
  public Map<User, List<Order>> orders(List<User> users) {
    var ids = users.stream().map(User::getId).toList();
    var orders = orderRepo.findByUserIdIn(ids);
    return users.stream().collect(Collectors.toMap(
      u -> u,
      u -> orders.stream().filter(o -> o.getUserId().equals(u.getId())).toList()
    ));
  }
}
```

`@BatchMapping` is Spring's first-class DataLoader integration — it auto-batches one call per request.

### Interview gotchas

- **Errors are always 200**. The client must inspect `response.errors`.
- **Nullability**: `String!` = non-null. If a non-null resolver returns null, the *parent* gets nulled and the error bubbles. Design types with this in mind.
- **Subscriptions over WebSocket** use the `graphql-ws` protocol (replaces deprecated `subscriptions-transport-ws`).
- **Query cost limiting**: depth limit, complexity score, persisted queries only (for public APIs).
- **No file uploads natively** — use `Upload` scalar via `graphql-upload`, or use a REST endpoint for uploads.

---

## 4. gRPC

### What it is

**Google Remote Procedure Call** — a high-performance RPC framework using **Protocol Buffers** over **HTTP/2**. You define services in `.proto`, generate strongly-typed client and server stubs in 11+ languages, and call methods as if they were local.

Think: "RPC done right, in 2015."

### Why it's fast

| Reason | Detail |
|---|---|
| **HTTP/2** | Multiplexed streams (no head-of-line blocking), header compression (HPACK), persistent connections |
| **Protobuf** | Binary, schema-driven, ~3–10× smaller than JSON, far cheaper to parse |
| **Streaming** | First-class streams (server, client, bidi) without WebSocket hacks |
| **Codegen** | Static types — fewer runtime checks |

### The four RPC patterns

```proto
service OrderService {
  rpc GetOrder(GetOrderRequest) returns (Order);                  // unary
  rpc ListOrders(ListOrdersRequest) returns (stream Order);       // server-streaming
  rpc UploadEvents(stream Event) returns (UploadSummary);         // client-streaming
  rpc Chat(stream ChatMsg) returns (stream ChatMsg);              // bidirectional
}
```

| Pattern | Use case |
|---|---|
| Unary | Plain RPC: "get this user" |
| Server-streaming | Search results, large list, log tail |
| Client-streaming | Upload events, bulk insert, telemetry |
| Bidirectional | Chat, gaming, real-time collaboration |

### Sample `.proto`

```proto
syntax = "proto3";

package orders.v1;
option java_package = "com.acme.orders.grpc";
option java_multiple_files = true;

service OrderService {
  rpc GetOrder(GetOrderRequest) returns (Order);
  rpc StreamOrders(StreamOrdersRequest) returns (stream Order);
}

message GetOrderRequest {
  string id = 1;
}

message Order {
  string id = 1;
  string user_id = 2;
  double total = 3;
  repeated OrderItem items = 4;
  Status status = 5;
  google.protobuf.Timestamp created_at = 6;
}

message OrderItem {
  string sku = 1;
  int32 qty = 2;
  double price = 3;
}

enum Status {
  STATUS_UNSPECIFIED = 0;
  PENDING = 1;
  PAID = 2;
  SHIPPED = 3;
  CANCELLED = 4;
}

message StreamOrdersRequest {
  string user_id = 1;
  int32 limit = 2;
}
```

**Protobuf field-number rules** (interview bait):

- Field numbers 1–15 use 1 byte → use them for hot fields.
- Never reuse a field number — even after deletion, mark it `reserved`.
- Default values are zero — there's no "field not set" by default in proto3 unless you use `optional` (re-added in proto3 3.15+) or wrapper types.

### Sample code

#### JS/TS server (`@grpc/grpc-js` + `@grpc/proto-loader`)

```ts
import * as grpc from '@grpc/grpc-js';
import * as protoLoader from '@grpc/proto-loader';
import path from 'path';

const def = protoLoader.loadSync(path.join(__dirname, 'orders.proto'), {
  keepCase: true, longs: String, enums: String, defaults: true, oneofs: true
});
const proto = grpc.loadPackageDefinition(def) as any;

const server = new grpc.Server();

server.addService(proto.orders.v1.OrderService.service, {
  GetOrder: async (call: any, cb: any) => {
    const order = await db.orders.findById(call.request.id);
    if (!order) return cb({ code: grpc.status.NOT_FOUND, message: 'order not found' });
    cb(null, order);
  },
  StreamOrders: async (call: any) => {
    const orders = await db.orders.findByUser(call.request.user_id);
    for (const o of orders) call.write(o);
    call.end();
  },
});

server.bindAsync('0.0.0.0:50051', grpc.ServerCredentials.createInsecure(), () => {
  server.start();
});
```

Client:

```ts
const client = new proto.orders.v1.OrderService(
  'localhost:50051',
  grpc.credentials.createInsecure()
);

// Unary
client.GetOrder({ id: '42' }, (err: any, order: any) => {
  if (err) console.error(err.code, err.message);
  else console.log(order);
});

// Server stream
const stream = client.StreamOrders({ user_id: 'u1', limit: 100 });
stream.on('data', (o: any) => console.log(o));
stream.on('end', () => console.log('done'));
stream.on('error', (e: any) => console.error(e));
```

#### Java / Spring Boot (`grpc-spring-boot-starter`)

`build.gradle`:

```gradle
dependencies {
  implementation 'net.devh:grpc-spring-boot-starter:3.1.0.RELEASE'
  implementation 'io.grpc:grpc-stub'
  implementation 'io.grpc:grpc-protobuf'
}

// Codegen via protobuf-gradle-plugin reads src/main/proto/*.proto
```

Server:

```java
@GrpcService
public class OrderGrpcService extends OrderServiceGrpc.OrderServiceImplBase {

  private final OrderRepository repo;
  public OrderGrpcService(OrderRepository r) { this.repo = r; }

  @Override
  public void getOrder(GetOrderRequest req, StreamObserver<Order> responseObs) {
    var found = repo.findById(req.getId());
    if (found.isEmpty()) {
      responseObs.onError(Status.NOT_FOUND
        .withDescription("order " + req.getId() + " not found")
        .asRuntimeException());
      return;
    }
    responseObs.onNext(toProto(found.get()));
    responseObs.onCompleted();
  }

  @Override
  public void streamOrders(StreamOrdersRequest req, StreamObserver<Order> responseObs) {
    repo.findByUserId(req.getUserId(), req.getLimit())
        .forEach(o -> responseObs.onNext(toProto(o)));
    responseObs.onCompleted();
  }
}
```

Client (consumer service):

```java
@Service
public class OrderClient {

  @GrpcClient("orders")  // resolved from application.yml: grpc.client.orders.address
  private OrderServiceGrpc.OrderServiceBlockingStub stub;

  public Order get(String id) {
    return stub
      .withDeadlineAfter(2, TimeUnit.SECONDS)
      .getOrder(GetOrderRequest.newBuilder().setId(id).build());
  }

  public List<Order> stream(String userId) {
    var iter = stub
      .withDeadlineAfter(10, TimeUnit.SECONDS)
      .streamOrders(StreamOrdersRequest.newBuilder().setUserId(userId).build());
    var out = new ArrayList<Order>();
    iter.forEachRemaining(out::add);
    return out;
  }
}
```

### Cross-cutting features

- **Deadlines** (a.k.a. timeouts): `stub.withDeadlineAfter(2, SECONDS)`. The deadline propagates across hops — *huge* for distributed systems.
- **Cancellation**: cancelling a call cancels in-flight RPCs through the chain.
- **Metadata**: like HTTP headers — `Metadata.Key.of("authorization", ASCII_STRING_MARSHALLER)`.
- **Interceptors**: middleware for auth, logging, tracing, metrics.
- **Load balancing**: client-side LB via xDS or `dns:///`. No L7 proxy needed for service-to-service.
- **mTLS** is the default for production.
- **Reflection** (`grpc.reflection.v1alpha.ServerReflection`) — let `grpcurl` introspect your services.

### gRPC-Web and gRPC-Gateway

- **gRPC-Web** lets browsers talk gRPC via Envoy proxy that translates HTTP/1.1 chunked frames ⇄ HTTP/2. Bidi streaming is *not* supported (server-streaming only).
- **grpc-gateway** auto-generates a REST/JSON façade in front of gRPC services. You annotate `.proto` with HTTP rules; it spits out an OpenAPI spec. Great for "internal = gRPC, external = REST."

### Pros / cons

**Pros**
- Multi-language type safety from one schema.
- Fastest wire format and lowest CPU among mainstream protocols.
- First-class streaming and bidi.
- Deadlines + cancellation built in.

**Cons**
- Not browser-native (need gRPC-Web + proxy).
- Binary — harder to debug without tooling.
- Versioning: never reuse field numbers, never change types — design carefully.
- HTTP intermediaries (most CDNs, load balancers) need HTTP/2 + h2c or termination.
- Steeper learning curve for codegen + proto management in a monorepo vs multi-repo.

### Interview gotchas

- "How does gRPC handle errors?" — Uses status codes (`OK`, `INVALID_ARGUMENT`, `NOT_FOUND`, `DEADLINE_EXCEEDED`, etc.) plus optional `google.rpc.Status` with rich details (RetryInfo, BadRequest, ResourceInfo).
- "Why HTTP/2 specifically?" — Multiplexing means many parallel streams over one TCP connection (no head-of-line blocking at the *application* layer, though TCP HOL still exists — see HTTP/3).
- "What's the difference between proto2 and proto3?" — proto3 dropped `required` (lesson: it's a versioning trap), defaulted to optional, added `Any`, JSON mapping.
- "How do you evolve a `.proto` safely?" — never reuse numbers; only add fields; mark removed as `reserved`; never change wire types.

---

## 5. WebSocket

### What it is

A **full-duplex, persistent TCP connection** that begins life as an HTTP request and gets "upgraded" to its own wire protocol. After the handshake, server and client can send messages either way at any time, framed independently, with very low overhead.

RFC 6455. Browser-native (`new WebSocket(url)`). Designed for "chat, live trading, multiplayer games."

### The handshake

Client sends:

```http
GET /chat HTTP/1.1
Host: example.com
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Key: dGhlIHNhbXBsZSBub25jZQ==
Sec-WebSocket-Version: 13
Sec-WebSocket-Protocol: chat, soap   # optional subprotocol negotiation
Origin: https://example.com          # browser only
```

Server responds:

```http
HTTP/1.1 101 Switching Protocols
Upgrade: websocket
Connection: Upgrade
Sec-WebSocket-Accept: s3pPLMBiTxaQ9kYGzzhZRbK+xOo=  # SHA1(key + magic GUID)
Sec-WebSocket-Protocol: chat
```

After 101, the same TCP connection carries WebSocket frames instead of HTTP. Frame structure:

```
 0                   1                   2                   3
 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1 2 3 4 5 6 7 8 9 0 1
+-+-+-+-+-------+-+-------------+-------------------------------+
|F|R|R|R| op    |M| Payload len |   Extended payload length     |
|I|S|S|S| code  |A|   (7)       |          (16/63 bits)         |
|N|V|V|V|       |S|             |                               |
+-+-+-+-+-------+-+-------------+-------------------------------+
```

Op codes: 0x1 text, 0x2 binary, 0x8 close, 0x9 ping, 0xA pong.

**Frames from client to server must be masked** (XOR with 4-byte key) — defends against cache-poisoning in proxies. Server → client is unmasked.

### Subprotocols you'll meet

- **STOMP** — text-based pub/sub (used by Spring's `@MessageMapping`). Topics like `/topic/chat`, `/queue/inbox`.
- **graphql-ws** — for GraphQL subscriptions.
- **MQTT-over-WebSocket** — same MQTT semantics but tunneled for browser clients.
- **SignalR (custom)** — Microsoft's WebSocket-or-fallback transport.
- **JSON-RPC over WebSocket** — common in crypto exchanges (Binance, Coinbase).

### Scaling WebSockets (the interview hotseat)

This is *the* favorite WebSocket interview question. Issues:

1. **Sticky sessions.** Once a client picks server A, all its messages stay on A. Use connection ID → server affinity at the LB (HAProxy/NGINX/AWS ALB all support it). Some shops use a consistent-hash LB instead.
2. **Cross-server fan-out.** User on server A messages a user on server B. Solution: every server subscribes to a shared **pub/sub** (Redis Pub/Sub, NATS, Kafka, RabbitMQ) and republishes to its locally connected sockets.
3. **Backpressure.** A slow client can fill the server's send buffer. Bounded queues + drop-on-overflow + close-with-policy-violation.
4. **Heartbeats / health.** Send ping every 30 s; close if no pong within 60 s. Critical because TCP keepalive defaults are 2 hours.
5. **Reconnect with state.** Clients must reconnect with a *resume token* (last message ID) so server can replay missed messages. Slack, Discord, and almost every chat app do this.
6. **Authn at handshake.** You can't send custom headers from the browser (`new WebSocket` doesn't allow it). Workarounds: pass JWT in subprotocol field, query param (`?token=...` — *URL-leaks*), or set a cookie before connecting.

### Sample code

#### JS/TS server (`ws`)

```ts
import { WebSocketServer } from 'ws';
import http from 'http';
import { parse } from 'url';

const server = http.createServer();
const wss = new WebSocketServer({ noServer: true });

server.on('upgrade', (req, socket, head) => {
  const { query } = parse(req.url!, true);
  const token = query.token as string;
  if (!verifyJwt(token)) {
    socket.write('HTTP/1.1 401 Unauthorized\r\n\r\n');
    socket.destroy();
    return;
  }
  wss.handleUpgrade(req, socket, head, ws => {
    wss.emit('connection', ws, req);
  });
});

wss.on('connection', (ws, req) => {
  const userId = jwtUserOf(parse(req.url!, true).query.token as string);

  // Heartbeat
  ws.isAlive = true;
  ws.on('pong', () => { ws.isAlive = true; });

  ws.on('message', (data) => {
    const msg = JSON.parse(data.toString());
    // Fan out
    for (const client of wss.clients) {
      if (client.readyState === 1) client.send(JSON.stringify({ from: userId, ...msg }));
    }
  });

  ws.on('close', (code, reason) => {
    console.log(`closed ${userId} ${code} ${reason}`);
  });
});

// Heartbeat sweep
setInterval(() => {
  for (const ws of wss.clients) {
    if (!ws.isAlive) return ws.terminate();
    ws.isAlive = false;
    ws.ping();
  }
}, 30000);

server.listen(8080);
```

Browser client:

```js
const ws = new WebSocket(`wss://api.example.com/chat?token=${jwt}`, ['chat.v1']);

ws.addEventListener('open', () => ws.send(JSON.stringify({ type: 'hello' })));
ws.addEventListener('message', e => console.log('got', JSON.parse(e.data)));
ws.addEventListener('close', e => console.log('closed', e.code, e.reason));
ws.addEventListener('error', e => console.error(e));

// Reconnect with exponential backoff + jitter
function connect(retries = 0) {
  const sock = new WebSocket(url);
  sock.onclose = () => {
    const delay = Math.min(30000, 2 ** retries * 500) + Math.random() * 500;
    setTimeout(() => connect(retries + 1), delay);
  };
  sock.onopen = () => { retries = 0; };
  return sock;
}
```

#### Java / Spring Boot — raw WebSocket

```java
@Configuration
@EnableWebSocket
public class WsConfig implements WebSocketConfigurer {
  @Override public void registerWebSocketHandlers(WebSocketHandlerRegistry r) {
    r.addHandler(new ChatHandler(), "/chat").setAllowedOrigins("*");
  }
}

public class ChatHandler extends TextWebSocketHandler {
  private final Set<WebSocketSession> sessions = ConcurrentHashMap.newKeySet();

  @Override public void afterConnectionEstablished(WebSocketSession s) {
    sessions.add(s);
  }

  @Override public void handleTextMessage(WebSocketSession s, TextMessage msg) throws Exception {
    var payload = new TextMessage("[" + s.getId() + "] " + msg.getPayload());
    for (var other : sessions) {
      if (other.isOpen()) other.sendMessage(payload);
    }
  }

  @Override public void afterConnectionClosed(WebSocketSession s, CloseStatus st) {
    sessions.remove(s);
  }
}
```

#### Java / Spring Boot — STOMP (preferred for richer apps)

```java
@Configuration
@EnableWebSocketMessageBroker
public class StompConfig implements WebSocketMessageBrokerConfigurer {
  @Override public void registerStompEndpoints(StompEndpointRegistry r) {
    r.addEndpoint("/ws").setAllowedOriginPatterns("*").withSockJS();
  }
  @Override public void configureMessageBroker(MessageBrokerRegistry r) {
    r.enableSimpleBroker("/topic", "/queue"); // or .enableStompBrokerRelay() → RabbitMQ
    r.setApplicationDestinationPrefixes("/app");
  }
}

@Controller
public class ChatController {
  @MessageMapping("/chat.send")     // client sends → /app/chat.send
  @SendTo("/topic/messages")        // broadcast to subscribers
  public ChatMessage send(ChatMessage in, Principal user) {
    return new ChatMessage(user.getName(), in.text(), Instant.now());
  }

  @MessageMapping("/chat.dm")
  public void direct(DmMessage in, Principal user, SimpMessagingTemplate t) {
    t.convertAndSendToUser(in.to(), "/queue/dm",
      new ChatMessage(user.getName(), in.text(), Instant.now()));
  }
}
```

Browser side uses `@stomp/stompjs`:

```ts
import { Client } from '@stomp/stompjs';

const c = new Client({
  brokerURL: 'wss://api.example.com/ws',
  connectHeaders: { Authorization: 'Bearer ' + jwt },
  reconnectDelay: 5000,
});

c.onConnect = () => {
  c.subscribe('/topic/messages', m => console.log(JSON.parse(m.body)));
  c.publish({ destination: '/app/chat.send', body: JSON.stringify({ text: 'hi' }) });
};
c.activate();
```

### Pros / cons

**Pros**
- True bidirectional, low-latency, low-overhead after handshake.
- Browser-native.
- Works through HTTP infra (port 80/443).

**Cons**
- Stateful — scaling needs sticky sessions + pub/sub bus.
- Auth is awkward (no headers from browser).
- No built-in request/response correlation — you build your own message envelopes.
- Idle connections can be killed by NAT timeouts and proxies; heartbeats mandatory.
- No backpressure semantics — must enforce yourself.

---

## 6. Webhook

### What it is

The inverse of a client-pulled API: **the server you registered with calls you back** when something happens. "HTTP callback." Stripe → payment succeeded; GitHub → push event; Slack → slash command.

There is no spec — it's just *"someone POSTs JSON to a URL you gave them."* But there are very strong conventions because the security and reliability concerns are sharp.

### Anatomy of a robust webhook

```http
POST /webhooks/stripe HTTP/1.1
Host: api.acme.com
Content-Type: application/json
Stripe-Signature: t=1697123456,v1=5257a869e7ec...
Stripe-Webhook-Id: evt_1OqPxx2eZvKYlo2C...

{
  "id": "evt_1OqPxx",
  "type": "payment_intent.succeeded",
  "created": 1697123456,
  "data": { "object": { ... } }
}
```

### Five things every webhook receiver MUST do

1. **Verify the signature.** HMAC-SHA256 of `timestamp + "." + raw_body` using a shared secret. *Always against the raw body* — JSON.parse + re-serialize breaks signatures.
2. **Reject old timestamps** (≥5 min skew → drop). Prevents replay attacks.
3. **Be idempotent.** Sender will retry on 5xx (and sometimes on timeouts even after success). Use the event ID as a dedupe key in Redis/DB.
4. **Return 2xx fast** (< 5 s or sender retries). If work is expensive, enqueue it and return 200 immediately.
5. **Handle out-of-order delivery.** Use timestamps or sequence numbers; never assume order.

Other concerns:
- **IP allowlist** (Stripe publishes IP ranges).
- **mTLS** in B2B setups.
- **Rate-limit your own receiver** so a buggy sender doesn't DoS you.
- **Dead letter queue** for events you can't process.

### Sender side (delivering webhooks reliably)

If *you're* sending webhooks (you're Stripe), you have to handle:

- **Retries with exponential backoff + jitter**: e.g., 1s, 5s, 30s, 5m, 1h, ... up to 3 days.
- **Per-endpoint queue** so one slow customer doesn't block others.
- **Circuit breaker** on consistently-failing endpoints (auto-pause after N failures, email the owner).
- **Signing**: HMAC with a secret per endpoint.
- **Replayability**: GUI to manually re-send any past event.
- **Observability**: deliver attempts, status codes, latency.

### Sample code

#### JS/TS (Express) receiver — Stripe style

```ts
import express from 'express';
import crypto from 'crypto';
import { redis } from './redis';

const app = express();

// IMPORTANT: capture raw body before any JSON parser touches it
app.post('/webhooks/stripe',
  express.raw({ type: 'application/json' }),
  async (req, res) => {
    const sig = req.header('Stripe-Signature') ?? '';
    const parts = Object.fromEntries(sig.split(',').map(p => p.split('=')));
    const t = Number(parts.t);
    const v1 = parts.v1;

    if (Math.abs(Date.now()/1000 - t) > 300) {
      return res.status(400).send('stale');
    }

    const payload = req.body.toString('utf8');
    const expected = crypto
      .createHmac('sha256', process.env.STRIPE_SECRET!)
      .update(`${t}.${payload}`)
      .digest('hex');

    if (!crypto.timingSafeEqual(Buffer.from(v1), Buffer.from(expected))) {
      return res.status(401).send('bad signature');
    }

    const event = JSON.parse(payload);

    // Dedupe — set NX with 24h TTL
    const fresh = await redis.set(`wh:${event.id}`, '1', 'EX', 86400, 'NX');
    if (!fresh) return res.status(200).send('duplicate');

    // Hand off to a job queue, then ack fast
    await queue.add('stripe-event', event);
    res.status(200).send('ok');
  }
);
```

#### Java / Spring Boot receiver

```java
@RestController
@RequestMapping("/webhooks")
public class StripeWebhookController {

  private final StringRedisTemplate redis;
  private final JobQueue queue;
  private final String secret;

  public StripeWebhookController(StringRedisTemplate r, JobQueue q,
                                 @Value("${stripe.webhook-secret}") String s) {
    this.redis = r; this.queue = q; this.secret = s;
  }

  @PostMapping(value = "/stripe", consumes = MediaType.APPLICATION_JSON_VALUE)
  public ResponseEntity<String> stripe(
      @RequestHeader("Stripe-Signature") String sigHeader,
      @RequestBody byte[] rawBody) throws Exception {

    // Parse "t=...,v1=..." header
    var parts = Arrays.stream(sigHeader.split(","))
      .map(p -> p.split("=", 2))
      .collect(Collectors.toMap(a -> a[0], a -> a[1]));
    long t = Long.parseLong(parts.get("t"));
    if (Math.abs(Instant.now().getEpochSecond() - t) > 300) {
      return ResponseEntity.badRequest().body("stale");
    }

    String signed = t + "." + new String(rawBody, StandardCharsets.UTF_8);
    Mac mac = Mac.getInstance("HmacSHA256");
    mac.init(new SecretKeySpec(secret.getBytes(), "HmacSHA256"));
    String expected = HexFormat.of().formatHex(mac.doFinal(signed.getBytes()));
    if (!MessageDigest.isEqual(expected.getBytes(), parts.get("v1").getBytes())) {
      return ResponseEntity.status(401).body("bad signature");
    }

    var event = new ObjectMapper().readTree(rawBody);
    String id = event.get("id").asText();
    Boolean fresh = redis.opsForValue()
      .setIfAbsent("wh:" + id, "1", Duration.ofHours(24));
    if (Boolean.FALSE.equals(fresh)) return ResponseEntity.ok("duplicate");

    queue.enqueue("stripe-event", event);
    return ResponseEntity.ok("ok");
  }
}
```

### Pros / cons

**Pros**
- Push-based — no polling, no wasted requests.
- Simple HTTP — works through any infra.
- Loose coupling — the producer doesn't know who's listening.

**Cons**
- Receiver must be publicly reachable and stable (no local dev without `ngrok` / `cloudflared tunnel`).
- Delivery semantics are *at-least-once* — receiver must dedupe.
- Sender has all the reliability complexity.
- Hard to fan out to many receivers (each gets its own queue & retry policy).

### Interview gotchas

- "At-least-once vs exactly-once" → exactly-once is a lie; you dedupe to *simulate* it. The receiver's idempotency key is the truth.
- "How do you secure them?" → signed headers (HMAC), short replay window, IP allowlist, mTLS for B2B.
- "What about local development?" → tunnel tools, or for the truly weird, ngrok + per-developer URL.
- "Compare with polling" → webhooks reduce wasted requests but shift reliability burden to producer.

---

## 7. Server-Sent Events (SSE)

A long-overlooked protocol that's perfect when you want **server → client** push and don't need client → server. EventSource is a W3C spec, browser-native (every modern browser).

### How it works

The client opens a normal GET. The server keeps the connection open and writes `text/event-stream`:

```http
GET /stream HTTP/1.1
Accept: text/event-stream

HTTP/1.1 200 OK
Content-Type: text/event-stream
Cache-Control: no-cache
Connection: keep-alive

event: orderUpdate
id: 1
data: {"id": 42, "status": "shipped"}

event: orderUpdate
id: 2
data: {"id": 43, "status": "paid"}

: this is a comment / keepalive

retry: 10000

```

Each "event" is `event:` + `id:` + `data:` + blank line. The browser's `EventSource` auto-parses and auto-reconnects.

### Why it's underrated

- Works over plain HTTP — no upgrade dance, no special infra.
- Built-in **auto-reconnect** with `Last-Event-ID` (server can resume from there).
- Plays nicely with HTTP/2 (no connection-per-stream cost).
- Simpler than WebSocket if you only need one direction (server → client) — chat apps usually need both, but dashboards, log streams, AI completions, stock tickers don't.

### When NOT to use

- You need client → server pushes (other than via separate POSTs).
- You need binary frames.
- Lots of HTTP/1.1 connection-per-tab pressure (you can hit the 6-connection-per-origin browser limit; HTTP/2 fixes this).
- Many proxies buffer text/event-stream; you may need `X-Accel-Buffering: no` for NGINX.

### Sample code

JS/TS server (Node/Express):

```ts
app.get('/orders/stream', (req, res) => {
  res.set({
    'Content-Type': 'text/event-stream',
    'Cache-Control': 'no-cache',
    'Connection': 'keep-alive',
    'X-Accel-Buffering': 'no',
  });

  const lastId = Number(req.header('Last-Event-ID') ?? 0);
  const sub = orderBus.subscribe(lastId, (event) => {
    res.write(`event: orderUpdate\n`);
    res.write(`id: ${event.seq}\n`);
    res.write(`data: ${JSON.stringify(event)}\n\n`);
  });

  // Heartbeat
  const ping = setInterval(() => res.write(':\n\n'), 15000);

  req.on('close', () => { clearInterval(ping); sub.unsubscribe(); });
});
```

Client:

```js
const es = new EventSource('/orders/stream');
es.addEventListener('orderUpdate', e => console.log(JSON.parse(e.data)));
es.onerror = () => console.warn('SSE closed, browser will auto-retry');
```

Java / Spring Boot:

```java
@GetMapping(value = "/orders/stream", produces = MediaType.TEXT_EVENT_STREAM_VALUE)
public Flux<ServerSentEvent<Order>> stream(
    @RequestHeader(value = "Last-Event-ID", required = false) Long lastId) {
  return orderEvents.from(lastId)
    .map(e -> ServerSentEvent.<Order>builder()
       .event("orderUpdate")
       .id(String.valueOf(e.seq()))
       .data(e.order())
       .build())
    .mergeWith(Flux.interval(Duration.ofSeconds(15))
       .map(t -> ServerSentEvent.<Order>builder().comment("keepalive").build()));
}
```

### Interview gotcha

"WebSocket vs SSE vs long-polling?"

| | Long-poll | SSE | WebSocket |
|---|---|---|---|
| Direction | server→client | server→client | bidirectional |
| Reconnect | implicit (next request) | automatic (browser) | manual (you write it) |
| Binary | no | no | yes |
| Browser-native | yes (just polls) | yes | yes |
| Auth via header | yes | yes (custom HTTP) | no (workarounds) |
| Backpressure | natural | natural | manual |
| Use it for | legacy fallback | dashboards, AI streams | chat, gaming |

---

## 8. HTTP/2, HTTP/3 and QUIC

A protocol nerd interview can dive deep here. Here's the version landscape you need.

### HTTP/1.1 (1997)

- One request per connection at a time → browsers open 6 connections per origin.
- Pipelining was supposed to help but is widely broken in proxies.
- Headers sent in plain text every request.

### HTTP/2 (RFC 7540, 2015)

Solves three big HTTP/1.1 problems while staying compatible with the same semantics.

| Feature | What it does |
|---|---|
| **Binary framing** | All messages are frames over a single TCP connection |
| **Multiplexing** | Many streams per connection — no need for 6 sockets per origin |
| **HPACK header compression** | Dynamic table of headers; massive savings on repeated cookies/auth |
| **Server push** | Server sends resources client *will* need (deprecated in Chrome 106 — barely used) |
| **Stream priorities** | Hint to the server about urgency |

But: **TCP head-of-line blocking** still bites. If one TCP packet is lost, the OS won't deliver later packets even though they belong to a different HTTP/2 stream. Multiplexing helped application-level HOL, not transport-level.

### HTTP/3 + QUIC (RFC 9000 / 9114, 2022)

The breakthrough: **move the transport off TCP, onto QUIC, which runs on UDP and includes TLS 1.3 built-in.**

QUIC features:

| | Detail |
|---|---|
| **Runs on UDP** | TCP head-of-line blocking is gone — each stream has its own loss recovery |
| **TLS 1.3 baked in** | One handshake establishes connection + TLS together |
| **0-RTT resumption** | If client has talked to this server before, first packet can carry request data — sub-RTT page loads |
| **Connection migration** | Connection ID is independent of IP — survives Wi-Fi → cellular handoff without dropping |
| **Better congestion control** | Pluggable algorithms (BBR, CUBIC) at user-space, easy to evolve |
| **Always encrypted** | Even headers — middleboxes can't fiddle as much |

HTTP/3 is "HTTP semantics, over QUIC." The familiar verbs, statuses, headers — but the framing layer is QUIC streams.

### Adoption signs

- Cloudflare, Google, Facebook, Akamai, Fastly serve HTTP/3.
- Chrome/Firefox/Safari support HTTP/3.
- Detect via `alt-svc: h3=":443"; ma=86400` header — server tells client "HTTP/3 also lives here."

### Why an SDE2 should care

- **0-RTT is amazing for mobile.** Cuts a round trip.
- **Connection migration matters** for mobile apps where the IP changes.
- **UDP-based protocols need different firewall rules** — block-by-default networks may break things.
- **Middleware visibility goes down** — old "deep packet inspection" tools can't see inside QUIC. Good for users, harder for ops.

### gRPC and HTTP/3

gRPC is officially HTTP/2-based. There's experimental work on gRPC over HTTP/3 (`grpc.io` blog posts), but production-ready stacks (Java, Go) still default to H2. Brought up if you say "modern gRPC."

### Interview gotchas

- "Why move from TCP to UDP?" — TCP HOL blocking, slow handshake (~3 RTTs to TLS-up), and ossified middleboxes. UDP gives the wire flexibility.
- "QUIC's biggest enemy?" — corporate / mobile-carrier firewalls that block or rate-limit UDP. Hence Alt-Svc fallback.
- "Did HTTP/3 change HTTP semantics?" — No. URLs, verbs, status codes, headers are all the same. Only the wire below changed.

### Sample (Java/Spring Boot enabling HTTP/2 — HTTP/3 is still experimental in 2026)

```yaml
# application.yml
server:
  port: 8443
  http2:
    enabled: true
  ssl:
    enabled: true
    key-store: classpath:keystore.p12
    key-store-password: changeit
    key-store-type: PKCS12
```

Spring Boot 3 ships with Netty + Reactor Netty; HTTP/3 support is being staged behind the `reactor-netty-http-http3` artifact. For now, real-world HTTP/3 stacks in Java are Jetty 12 with QUIC plugin or Netty's `incubator-codec-http3`.

---

## 9. MQTT

### What it is

**Message Queuing Telemetry Transport** (1999, Andy Stanford-Clark @ IBM). A featherweight pub/sub protocol that runs over TCP (default port 1883 / TLS 8883) and was *built* for IoT: low bandwidth, intermittent connectivity, tiny devices, thousands of publishers.

Three roles:

- **Publisher** — sends messages to a topic.
- **Broker** — central exchange (Mosquitto, EMQX, HiveMQ, AWS IoT Core).
- **Subscriber** — listens to topics.

A topic is a slash-separated string: `home/livingroom/temperature`. Wildcards on subscribe:
- `+` — single-level wildcard: `home/+/temperature` → matches `home/kitchen/temperature` but not `home/floor1/kitchen/temperature`.
- `#` — multi-level wildcard at the end: `home/#` → everything under `home/`.

### QoS levels (the most-asked MQTT interview question)

| QoS | Semantics | Wire | Use case |
|---|---|---|---|
| 0 | At most once ("fire and forget") | 1 packet | Frequent sensor readings where loss is okay |
| 1 | At least once | 2 packets (PUBLISH + PUBACK) | Default — receiver may see duplicates |
| 2 | Exactly once | 4 packets (PUBLISH, PUBREC, PUBREL, PUBCOMP) | Billing events, critical commands |

QoS is end-to-end between *publisher↔broker* and *broker↔subscriber* separately. Effective QoS = min of the two legs. A publisher sending QoS 2 to a broker doesn't mean QoS 2 to a subscriber who only subscribed at QoS 1.

### Retained messages

`PUBLISH` with retain flag → broker keeps the *last* message on that topic. New subscribers get it immediately. Great for "device state" topics where a sensor publishes once an hour but a dashboard wants the latest reading on page load.

### Last Will and Testament (LWT)

When you connect, you tell the broker "if I die unexpectedly, publish *this* to *that* topic." On unclean disconnect (TCP drop, missed keepalive), the broker fires the LWT.

Classic pattern:

```
ConnectOptions: will={ topic: "device/sensor-42/status", payload: "offline", retain: true }
Then publish on connect: device/sensor-42/status = "online" (retain: true)
```

Now any subscriber knows the device's last seen state.

### MQTT 5 highlights (most production deployments are MQTT 5 now)

- **Reason codes** on every ack (richer error info).
- **Topic aliases** (cut bandwidth).
- **Shared subscriptions** — `$share/group1/topic/x` lets multiple subscribers split work like a queue.
- **User properties** (key-value headers).
- **Message expiry**, **session expiry**.

### Sample code

JS/TS with `mqtt`:

```ts
import mqtt from 'mqtt';

const client = mqtt.connect('mqtts://broker.example.com:8883', {
  clientId: 'svc-orders-' + Math.random().toString(16).slice(2),
  username: process.env.MQTT_USER,
  password: process.env.MQTT_PASS,
  clean: false,                              // resume previous session
  keepalive: 60,
  reconnectPeriod: 2000,
  will: {
    topic: 'service/orders/status',
    payload: 'offline',
    qos: 1,
    retain: true,
  },
});

client.on('connect', () => {
  client.publish('service/orders/status', 'online', { qos: 1, retain: true });
  client.subscribe('order/+/created', { qos: 1 });
});

client.on('message', (topic, payload, packet) => {
  console.log(topic, payload.toString());
});

// Publish a domain event
client.publish('order/42/created',
  JSON.stringify({ id: 42, total: 99.5 }),
  { qos: 1 });
```

Java with Eclipse Paho (Spring Boot uses `spring-integration-mqtt`):

```java
@Configuration
public class MqttConfig {

  @Bean
  public MqttPahoClientFactory mqttClientFactory() {
    var f = new DefaultMqttPahoClientFactory();
    var opts = new MqttConnectOptions();
    opts.setServerURIs(new String[]{ "tcp://broker.example.com:1883" });
    opts.setUserName("svc"); opts.setPassword("secret".toCharArray());
    opts.setCleanSession(false);
    opts.setKeepAliveInterval(60);
    f.setConnectionOptions(opts);
    return f;
  }

  // Inbound channel
  @Bean
  public IntegrationFlow mqttInFlow(MqttPahoClientFactory f) {
    var adapter = new MqttPahoMessageDrivenChannelAdapter(
      "consumer-" + UUID.randomUUID(), f, "order/+/created");
    adapter.setQos(1);
    return IntegrationFlow.from(adapter)
      .handle(m -> System.out.println(m.getHeaders().get("mqtt_receivedTopic")
        + " -> " + m.getPayload()))
      .get();
  }

  // Outbound channel
  @Bean
  @ServiceActivator(inputChannel = "mqttOut")
  public MessageHandler outbound(MqttPahoClientFactory f) {
    var h = new MqttPahoMessageHandler("publisher", f);
    h.setDefaultTopic("order/created");
    h.setDefaultQos(1);
    h.setAsync(true);
    return h;
  }
}
```

### Pros / cons

**Pros**
- Tiny on the wire (2-byte fixed header).
- Built-in QoS, retain, LWT, keepalive.
- Excellent for huge fan-out (one publisher → millions of subscribers).
- TLS support is solid; brokers handle authn/authz.

**Cons**
- Needs a broker (single dependency).
- Topic design is critical and hard to change later.
- Not request/response (you build that on top with `response_topic` and `correlation_data` in MQTT 5).
- Sub-topic ACLs can get unwieldy at scale.

### Interview gotchas

- "How does MQTT compare to Kafka?" — different category. MQTT is broadcast pub/sub with a fanout broker; Kafka is a distributed log with consumer groups and replay. MQTT is "last value matters" or live events. Kafka is "every event recorded forever, replay at will."
- "QoS 2 — is it really exactly-once?" — Yes, end-to-end on that leg, but your *application's* idempotency still matters across reconnects, broker restarts, etc.
- "Where would you NOT use MQTT?" — when you need analytical replay (use Kafka), when you need request/response RPC (use gRPC), when you need browser-native (use WebSocket — though MQTT-over-WSS works).

---

## 10. SOAP

### What it is

**Simple Object Access Protocol** (1998 at Microsoft + IBM; W3C-standardized as SOAP 1.2 in 2003). An XML-based messaging protocol with strict contracts (WSDL), built for **enterprise integration** — cross-org payments, government filing, B2B EDI, telco interconnect, healthcare. Predates REST as the canonical "web service" pattern.

You said it's old. It is. But it won't die because:
- Banking, healthcare, government, and telco have decade-spanning integrations they can't rewrite.
- SOAP message-level security (sign one field, encrypt another) is still hard to replicate with TLS-only stacks.
- It's transport-agnostic — the same SOAP message can travel HTTP, JMS, SMTP, or a private MQ.

If you join a bank, an insurer, or any enterprise integration team, you *will* meet SOAP. Don't freeze.

### The envelope

Every SOAP message is wrapped in an `<Envelope>` with an optional `<Header>` and a required `<Body>`:

```xml
POST /OrderService HTTP/1.1
Host: api.acme.com
Content-Type: text/xml; charset=utf-8                  <!-- SOAP 1.1 -->
SOAPAction: "http://acme.com/orders/getOrder"

<?xml version="1.0" encoding="UTF-8"?>
<soap:Envelope xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/"
               xmlns:ord="http://acme.com/orders"
               xmlns:wsa="http://www.w3.org/2005/08/addressing"
               xmlns:wsse="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-secext-1.0.xsd"
               xmlns:wsu="http://docs.oasis-open.org/wss/2004/01/oasis-200401-wss-wssecurity-utility-1.0.xsd">
  <soap:Header>
    <wsa:MessageID>uuid:8f2a-...</wsa:MessageID>           <!-- WS-Addressing -->
    <wsa:Action>http://acme.com/orders/getOrder</wsa:Action>
    <wsse:Security soap:mustUnderstand="1">                <!-- WS-Security -->
      <wsu:Timestamp>
        <wsu:Created>2026-06-02T10:00:00Z</wsu:Created>
        <wsu:Expires>2026-06-02T10:05:00Z</wsu:Expires>
      </wsu:Timestamp>
      <wsse:UsernameToken>
        <wsse:Username>svc-orders</wsse:Username>
        <wsse:Password Type="...#PasswordDigest">b64...</wsse:Password>
        <wsse:Nonce>aGVsbG8=</wsse:Nonce>
        <wsu:Created>2026-06-02T10:00:00Z</wsu:Created>
      </wsse:UsernameToken>
    </wsse:Security>
  </soap:Header>
  <soap:Body>
    <ord:GetOrderRequest>
      <ord:Id>42</ord:Id>
    </ord:GetOrderRequest>
  </soap:Body>
</soap:Envelope>
```

`mustUnderstand="1"` on a header element means: *if you don't know how to process this header, fault out — don't silently ignore.* Used heavily by WS-Security and WS-ReliableMessaging.

**SOAP 1.1 vs 1.2 — subtle interop trap:**

| | 1.1 | 1.2 |
|---|---|---|
| Namespace | `http://schemas.xmlsoap.org/soap/envelope/` | `http://www.w3.org/2003/05/soap-envelope` |
| Content type | `text/xml` | `application/soap+xml` (with `action` parameter) |
| SOAPAction header | Required (HTTP header) | Folded into Content-Type |
| Fault shape | `<faultcode>`, `<faultstring>`, `<detail>` | `<Code>`, `<Reason>`, `<Detail>` (nested subcodes) |
| HTTP GET | Not allowed | Allowed for safe ops |

Real-world systems mix both. Always check the WSDL — the binding namespace tells you which version.

### SOAP Faults (the equivalent of HTTP error codes)

A fault always returns **HTTP 500** — the real error lives in the SOAP body:

```xml
<soap:Envelope xmlns:soap="http://www.w3.org/2003/05/soap-envelope">
  <soap:Body>
    <soap:Fault>
      <soap:Code>
        <soap:Value>soap:Sender</soap:Value>              <!-- Sender / Receiver -->
        <soap:Subcode>
          <soap:Value xmlns:ord="http://acme.com/orders">ord:OrderNotFound</soap:Value>
        </soap:Subcode>
      </soap:Code>
      <soap:Reason>
        <soap:Text xml:lang="en">Order 42 not found</soap:Text>
      </soap:Reason>
      <soap:Detail>
        <ord:NotFoundFault>
          <ord:OrderId>42</ord:OrderId>
          <ord:RetryAfterSeconds>0</ord:RetryAfterSeconds>
        </ord:NotFoundFault>
      </soap:Detail>
    </soap:Fault>
  </soap:Body>
</soap:Envelope>
```

`soap:Sender` ≈ 4xx (caller's fault); `soap:Receiver` ≈ 5xx (server's fault). The `Subcode` chain carries domain-specific error names; `Detail` carries machine-readable typed payload — the SOAP equivalent of `application/problem+json`.

> **Gotcha:** monitoring tools that bucket on HTTP status code show *every* SOAP fault as 500. You need to parse the body to distinguish a real server crash from "order not found." Most SOAP-aware APMs (Dynatrace, AppDynamics) do this; generic ones don't.

### WSDL — the contract

**Web Services Description Language** — think of it as `.proto` + OpenAPI rolled into one XML file. A WSDL has five sections:

```xml
<definitions xmlns="http://schemas.xmlsoap.org/wsdl/"
             xmlns:xs="http://www.w3.org/2001/XMLSchema"
             xmlns:soap="http://schemas.xmlsoap.org/wsdl/soap/"
             xmlns:tns="http://acme.com/orders"
             targetNamespace="http://acme.com/orders">

  <!-- 1. types: XML Schema describing the request/response payloads -->
  <types>
    <xs:schema targetNamespace="http://acme.com/orders">
      <xs:element name="GetOrderRequest">
        <xs:complexType><xs:sequence>
          <xs:element name="Id" type="xs:string"/>
        </xs:sequence></xs:complexType>
      </xs:element>
      <xs:element name="GetOrderResponse">
        <xs:complexType><xs:sequence>
          <xs:element name="Order" type="tns:OrderType"/>
        </xs:sequence></xs:complexType>
      </xs:element>
      <xs:complexType name="OrderType">
        <xs:sequence>
          <xs:element name="Id"     type="xs:string"/>
          <xs:element name="Total"  type="xs:decimal"/>
          <xs:element name="Status" type="xs:string"/>
        </xs:sequence>
      </xs:complexType>
    </xs:schema>
  </types>

  <!-- 2. messages: input / output payload references -->
  <message name="GetOrderInput">
    <part name="parameters" element="tns:GetOrderRequest"/>
  </message>
  <message name="GetOrderOutput">
    <part name="parameters" element="tns:GetOrderResponse"/>
  </message>

  <!-- 3. portType: the abstract interface -->
  <portType name="OrderPortType">
    <operation name="GetOrder">
      <input  message="tns:GetOrderInput"/>
      <output message="tns:GetOrderOutput"/>
    </operation>
  </portType>

  <!-- 4. binding: concrete transport + encoding -->
  <binding name="OrderBinding" type="tns:OrderPortType">
    <soap:binding style="document"
                  transport="http://schemas.xmlsoap.org/soap/http"/>
    <operation name="GetOrder">
      <soap:operation soapAction="http://acme.com/orders/getOrder"/>
      <input>  <soap:body use="literal"/> </input>
      <output> <soap:body use="literal"/> </output>
    </operation>
  </binding>

  <!-- 5. service: where to call it -->
  <service name="OrderService">
    <port name="OrderPort" binding="tns:OrderBinding">
      <soap:address location="https://api.acme.com/OrderService"/>
    </port>
  </service>
</definitions>
```

The choices in `<binding>` open a famous Pandora's box:

| Style / use | Wire shape | Verdict |
|---|---|---|
| **document/literal wrapped** | Body has one element matching the operation; XSD validates it | **The 2026 default** — pick this |
| document/literal bare | Multiple top-level body elements | Breaks some toolkits; avoid |
| RPC/encoded | Body wraps the operation name; uses SOAP encoding (Section 5) | Legacy; WS-I forbids |
| RPC/literal | Wraps op name, XML Schema for body | Tolerable for legacy |

> **Senior-engineer answer:** "document/literal wrapped, per WS-I Basic Profile 1.x." Memorize that line — it instantly signals you've shipped SOAP services before.

### The WS-* universe (the famous "ws-deathstar")

A constellation of specs layered on top of SOAP. You don't need to recite specs — know what each does.

| Spec | What it does | When you'll see it |
|---|---|---|
| **WS-Security** | Sign + encrypt parts of the envelope; UsernameToken, X.509, SAML tokens | Almost every production SOAP service |
| **WS-Addressing** | Standard `MessageID`, `ReplyTo`, `FaultTo` headers — transport-independent routing | Async / message-bus integrations |
| **WS-ReliableMessaging** | Acks, deduplication, ordered delivery at the SOAP layer | When transport isn't reliable enough |
| **WS-AtomicTransaction + WS-Coordination** | 2-phase commit across SOAP services | Distributed XA transactions |
| **WS-Policy** | Declarative requirements ("body must be signed with X.509") | Used with WS-Security |
| **WS-Trust + WS-Federation** | Token issuance, federated identity | Cross-org SSO |
| **MTOM / XOP** | Binary attachments inlined by reference (`<xop:Include href="cid:..."/>`) | PDFs, images, scans in SOAP messages |
| **SAML** | Token format usually carried inside WS-Security | Enterprise auth |

WS-Security in one minute:

- **UsernameToken** — username + password digest (`SHA1(nonce + created + password)`). Easy, weak — only useful behind TLS.
- **X.509 token signing** — sign the body with the sender's cert; receiver verifies. Strong non-repudiation.
- **EncryptedData (XML Encryption)** — encrypt selected elements (you can encrypt `<CreditCard>` while leaving `<OrderId>` plaintext so intermediaries can route).
- **SAML assertions** — carry SAML tokens in the SOAP header for federated identity.

That last point — element-level encryption — is the SOAP feature TLS *can't* replicate. TLS encrypts the whole channel point-to-point; WS-Security encrypts specific elements and survives intermediaries that need to see *some* of the message but mustn't see others. This is why SOAP is sticky in banking: an FX gateway can route a payment by currency code while never seeing the account number.

### Transports (the surprising part)

SOAP is transport-agnostic. You'll see it on:

- **HTTP(S)** — by far the most common.
- **JMS** — banks running ActiveMQ / IBM MQ / Solace often wrap SOAP in JMS for guaranteed delivery + transactional semantics.
- **SMTP** — yes, SOAP-over-email. Rare but real (slow async B2B).
- **Raw TCP** — internal high-perf legacy integrations.

That's why WS-Addressing exists: HTTP makes request/response implicit, but JMS/SMTP need `ReplyTo` and `FaultTo` encoded *in* the envelope.

### Sample code

#### Java / JAX-WS (Jakarta XML Web Services) — contract-last server

JAX-WS is the standard Java SOAP stack. It was removed from JDK 11+ and lives on as Jakarta artifacts. Apache CXF is the most common runtime in production.

```xml
<!-- pom.xml essentials -->
<dependency>
  <groupId>jakarta.xml.ws</groupId>
  <artifactId>jakarta.xml.ws-api</artifactId>
  <version>4.0.2</version>
</dependency>
<dependency>
  <groupId>com.sun.xml.ws</groupId>
  <artifactId>jaxws-rt</artifactId>
  <version>4.0.3</version>
</dependency>
```

```java
// OrderService.java — the SEI (Service Endpoint Interface)
@WebService(targetNamespace = "http://acme.com/orders")
public interface OrderService {
  @WebMethod
  @WebResult(name = "Order")
  Order getOrder(@WebParam(name = "Id") String id) throws OrderNotFoundFault;
}

// OrderServiceImpl.java
@WebService(endpointInterface = "com.acme.orders.OrderService",
            targetNamespace   = "http://acme.com/orders",
            serviceName       = "OrderService",
            portName          = "OrderPort")
public class OrderServiceImpl implements OrderService {

  private final OrderRepository repo;
  public OrderServiceImpl(OrderRepository r) { this.repo = r; }

  @Override
  public Order getOrder(String id) throws OrderNotFoundFault {
    return repo.findById(id).orElseThrow(() ->
      new OrderNotFoundFault("Order " + id + " not found",
        new OrderNotFoundDetail(id)));
  }
}

// Standalone publish (usually you let Tomcat/CXF do this)
public class App {
  public static void main(String[] args) {
    Endpoint.publish("http://0.0.0.0:8080/OrderService",
      new OrderServiceImpl(new OrderRepository()));
  }
}
```

The WSDL is auto-generated from annotations — visit `http://localhost:8080/OrderService?wsdl`.

Client generation:

```bash
# Generate Java stubs from a remote WSDL
wsimport -keep -d src/main/java https://api.acme.com/OrderService?wsdl
```

```java
// Consume the generated stub
OrderService_Service svc = new OrderService_Service();
OrderService port = svc.getOrderPort();

BindingProvider bp = (BindingProvider) port;

// WS-Security UsernameToken via request-context properties
bp.getRequestContext().put(BindingProvider.USERNAME_PROPERTY, "svc");
bp.getRequestContext().put(BindingProvider.PASSWORD_PROPERTY, "secret");

// Timeouts (Metro/CXF — names differ by RI)
bp.getRequestContext().put("com.sun.xml.ws.connect.timeout", 2000);
bp.getRequestContext().put("com.sun.xml.ws.request.timeout", 5000);

try {
  Order order = port.getOrder("42");
} catch (OrderNotFoundFault f) {
  // Detail bean is the typed SOAP fault payload
  System.err.println("not found: " + f.getFaultInfo().getOrderId());
}
```

#### Spring-WS — contract-first server (the modern Java SOAP idiom)

Spring-WS makes you write the XSD/WSDL *first*, then implement endpoints — the opposite philosophy of JAX-WS reflection magic. The result is cleaner contracts that don't drift with refactors.

```xml
<!-- src/main/resources/orders.xsd -->
<xs:schema xmlns:xs="http://www.w3.org/2001/XMLSchema"
           xmlns:ord="http://acme.com/orders"
           targetNamespace="http://acme.com/orders"
           elementFormDefault="qualified">
  <xs:element name="GetOrderRequest">
    <xs:complexType><xs:sequence>
      <xs:element name="Id" type="xs:string"/>
    </xs:sequence></xs:complexType>
  </xs:element>
  <xs:element name="GetOrderResponse">
    <xs:complexType><xs:sequence>
      <xs:element name="Order" type="ord:OrderType"/>
    </xs:sequence></xs:complexType>
  </xs:element>
  <xs:complexType name="OrderType">
    <xs:sequence>
      <xs:element name="Id"     type="xs:string"/>
      <xs:element name="Total"  type="xs:decimal"/>
      <xs:element name="Status" type="xs:string"/>
    </xs:sequence>
  </xs:complexType>
</xs:schema>
```

```java
// WebServiceConfig.java
@EnableWs
@Configuration
public class WebServiceConfig extends WsConfigurerAdapter {

  @Bean
  public ServletRegistrationBean<MessageDispatcherServlet> dispatcher(
      ApplicationContext ctx) {
    var servlet = new MessageDispatcherServlet();
    servlet.setApplicationContext(ctx);
    servlet.setTransformWsdlLocations(true);
    return new ServletRegistrationBean<>(servlet, "/ws/*");
  }

  // Expose http://host/ws/orders.wsdl built from the XSD
  @Bean(name = "orders")
  public DefaultWsdl11Definition wsdl(XsdSchema schema) {
    var w = new DefaultWsdl11Definition();
    w.setPortTypeName("OrderPort");
    w.setLocationUri("/ws");
    w.setTargetNamespace("http://acme.com/orders");
    w.setSchema(schema);
    return w;
  }

  @Bean
  public XsdSchema schema() {
    return new SimpleXsdSchema(new ClassPathResource("orders.xsd"));
  }

  // WS-Security: validate UsernameToken + Timestamp on every request
  @Bean
  public Wss4jSecurityInterceptor securityInterceptor() {
    var i = new Wss4jSecurityInterceptor();
    i.setValidationActions("UsernameToken Timestamp");
    i.setValidationCallbackHandler(passwordHandler());
    i.setValidateRequest(true);
    i.setSecureResponse(true);
    return i;
  }

  @Bean
  public SimplePasswordValidationCallbackHandler passwordHandler() {
    var h = new SimplePasswordValidationCallbackHandler();
    h.setUsersMap(Map.of("svc", "secret"));
    return h;
  }

  @Override
  public void addInterceptors(List<EndpointInterceptor> interceptors) {
    interceptors.add(securityInterceptor());
    interceptors.add(new PayloadLoggingInterceptor());
    interceptors.add(payloadValidatingInterceptor());
  }

  @Bean
  public PayloadValidatingInterceptor payloadValidatingInterceptor() {
    var i = new PayloadValidatingInterceptor();
    i.setSchema(new ClassPathResource("orders.xsd"));
    i.setValidateRequest(true);
    i.setValidateResponse(true);
    return i;
  }
}
```

```java
// OrderEndpoint.java
@Endpoint
public class OrderEndpoint {

  private static final String NS = "http://acme.com/orders";
  private final OrderRepository repo;
  public OrderEndpoint(OrderRepository r) { this.repo = r; }

  @PayloadRoot(namespace = NS, localPart = "GetOrderRequest")
  @ResponsePayload
  public GetOrderResponse getOrder(@RequestPayload GetOrderRequest req) {
    var order = repo.findById(req.getId())
      .orElseThrow(() -> new OrderNotFoundException(req.getId()));

    var resp = new GetOrderResponse();
    resp.setOrder(mapToXmlType(order));
    return resp;
  }
}

// Map a domain exception → SOAP Fault
@SoapFault(faultCode = FaultCode.CLIENT,
           faultStringOrReason = "Order not found")
public class OrderNotFoundException extends RuntimeException {
  public OrderNotFoundException(String id) {
    super("Order " + id + " not found");
  }
}
```

JAXB classes (`GetOrderRequest`, `GetOrderResponse`, `OrderType`) are generated from the XSD by the `jaxb2-maven-plugin` (or `xjc` CLI) at build time.

Spring-WS client side:

```java
public class OrderClient extends WebServiceGatewaySupport {

  public Order getOrder(String id) {
    var req = new GetOrderRequest();
    req.setId(id);

    var resp = (GetOrderResponse) getWebServiceTemplate()
      .marshalSendAndReceive(
        "https://api.acme.com/ws/orders",
        req,
        msg -> ((SoapMessage) msg).setSoapAction(
          "http://acme.com/orders/getOrder")
      );
    return mapFromXmlType(resp.getOrder());
  }
}

@Bean
public OrderClient orderClient(Jaxb2Marshaller m) {
  var c = new OrderClient();
  c.setDefaultUri("https://api.acme.com/ws/orders");
  c.setMarshaller(m);
  c.setUnmarshaller(m);
  // WS-Security on the client side
  c.setInterceptors(new ClientInterceptor[]{
    clientSecurityInterceptor()
  });
  return c;
}

@Bean
public Wss4jSecurityInterceptor clientSecurityInterceptor() {
  var i = new Wss4jSecurityInterceptor();
  i.setSecurementActions("UsernameToken Timestamp");
  i.setSecurementUsername("svc");
  i.setSecurementPassword("secret");
  i.setSecurementPasswordType(WSConstants.PW_DIGEST);
  return i;
}
```

#### JS/TS — `soap` package (client + server)

The `soap` package on npm is the most-used SOAP library in Node. Reasonable for tooling, integration tests, and small services. (`strong-soap` is a fork; pick whichever's maintained when you start.)

```ts
// client.ts
import { createClientAsync, WSSecurity } from 'soap';

const wsdlUrl = 'https://api.acme.com/OrderService?wsdl';

const client = await createClientAsync(wsdlUrl, {
  // Add HTTP-level auth for the WSDL fetch itself
  wsdl_headers: { Authorization: 'Basic ' + Buffer.from('svc:secret').toString('base64') },
});

// WS-Security UsernameToken on every call
client.setSecurity(new WSSecurity('svc', 'secret', {
  passwordType: 'PasswordDigest',
  hasNonce:     true,
  hasTimeStamp: true,
}));

// SOAP 1.2 (default is 1.1)
// client.setSOAPVersion('1.2');

const [result, rawXml, soapHeader, rawRequest] =
  await client.GetOrderAsync({ Id: '42' });

console.log(result.Order);
```

```ts
// server.ts — expose a SOAP endpoint from Node
import express from 'express';
import { listen } from 'soap';
import fs from 'fs';

const wsdl = fs.readFileSync('./orders.wsdl', 'utf8');

const service = {
  OrderService: {
    OrderPort: {
      GetOrder: ({ Id }, callback, headers) => {
        // Headers contains the parsed wsse:Security info if present
        const order = db.findOrder(Id);
        if (!order) {
          // Return a SOAP fault by throwing a tagged object
          throw {
            Fault: {
              Code: {
                Value:    'soap:Sender',
                Subcode:  { value: 'ord:OrderNotFound' },
              },
              Reason: { Text: `Order ${Id} not found` },
              Detail: { OrderId: Id },
              statusCode: 500,
            }
          };
        }
        callback(null, { Order: order });
      },
    },
  },
};

const app = express();
const server = app.listen(8080);
listen(server, '/OrderService', service, wsdl);
```

### Pros / cons

**Pros**
- **Strict contracts** via WSDL + XML Schema — every field typed, validated, codegen-friendly across 10+ language stacks.
- **Message-level security** (WS-Security) — sign or encrypt specific elements; survives intermediaries.
- **Transport-agnostic** — HTTP, JMS, SMTP, raw TCP. The envelope doesn't care.
- **Reliable messaging** built in (WS-ReliableMessaging) — acks, deduplication, ordered delivery layered on unreliable transports.
- **Two-phase commit** across services (WS-AtomicTransaction + WS-Coordination) — almost no other web protocol offers this.
- **Formal compliance posture** — regulators (PCI/HIPAA/PSD2) understand and accept it; auditors have decades of tooling.

**Cons**
- **Verbose.** Same payload is 5–10× the bytes of JSON. CPU-heavy XML parsing.
- **WS-\* sprawl.** Each spec is 100+ pages with subtle interop holes between toolkits (CXF ↔ WCF ↔ Axis2 ↔ Spring-WS).
- **No browser story.** No native XML parsing for SOAP envelopes; if you want web clients, you wrap REST/JSON in front.
- **Tooling rot.** JAX-WS removed from the JDK; .NET deprecated WCF for new dev (CoreWCF is a community fork); Node libraries are sparsely maintained.
- **Errors are HTTP 500** by convention — confuses observability tools that bucket on HTTP status.
- **Steep learning curve** for the WS-* family.
- **No native streaming** — chunked transfer is possible but not idiomatic.

### Interview gotchas

- **"SOAP vs HTTP"** — SOAP is a *messaging protocol*; HTTP is a *transport*. SOAP usually runs on HTTP, but doesn't have to. Don't conflate them.
- **"SOAP 1.1 vs 1.2"** — Different namespaces, different content types (`text/xml` vs `application/soap+xml`), different fault shapes, `SOAPAction` moved from HTTP header into Content-Type parameter. Check the WSDL.
- **"What's in a WSDL?"** — Types (XSD), Messages (input/output payloads), PortType (abstract interface), Binding (transport + encoding), Service (endpoint URL). Tooling generates client/server stubs.
- **"What style and use?"** — "Document/literal wrapped, per WS-I Basic Profile 1.x." Signals you've shipped this before.
- **"How does WS-Security differ from TLS?"** — TLS encrypts the whole channel point-to-point. WS-Security encrypts/signs *specific elements* and persists through intermediaries. You can encrypt one field while letting a router see another.
- **"What's `mustUnderstand`?"** — Header attribute forcing receivers to either process the header or fault out. Prevents silent misinterpretation.
- **"Why does SOAP need WS-Addressing?"** — Because SOAP can run over async transports (JMS, SMTP) with no implicit reply channel; `ReplyTo`/`FaultTo` headers solve it.
- **"How do you attach binary files?"** — MTOM (Message Transmission Optimization Mechanism) with XOP. Binary parts ride as MIME attachments referenced from the SOAP body via `<xop:Include href="cid:..."/>`. Base64-in-CDATA also works but inflates ~33%.
- **"How do you version a SOAP API?"** — Namespace versioning: bump `targetNamespace` (`http://acme.com/orders/v2`). Old clients keep using the v1 namespace; you run both endpoints side by side. WSDL diffing tools like SoapUI help with backwards-compat audits.
- **"Contract-first vs contract-last?"** — Contract-first (write WSDL/XSD by hand → generate code) gives stable contracts and clean schema evolution; contract-last (annotate Java code → WSDL falls out) is faster to start but the contract drifts with refactors. Production shops prefer contract-first.
- **"How do you migrate SOAP → REST?"** — Strangler-fig pattern: stand up REST endpoints alongside SOAP, route progressively per consumer, deprecate WSDL versions one at a time, keep the SOAP endpoint as a thin façade until external partners migrate. Typically a 2–3 year program.
- **"Idempotency in SOAP?"** — No standard. Common pattern: `WS-Addressing MessageID` doubles as the idempotency key on the server. WS-ReliableMessaging adds explicit at-most-once / exactly-once delivery semantics if both sides support it.

---

## 11. Comparison matrix

| | REST | GraphQL | gRPC | WebSocket | SSE | Webhook | MQTT | SOAP |
|---|---|---|---|---|---|---|---|---|
| Transport | HTTP/1.1–3 | HTTP/1.1–3 | HTTP/2 | TCP (after HTTP upgrade) | HTTP/1.1–3 | HTTP | TCP / TLS | HTTP/SMTP/TCP |
| Format | JSON (usually) | JSON | Protobuf (binary) | Any (often JSON) | text (lines) | JSON | Binary | XML |
| Direction | C→S | C→S (+ Sub C↔S) | C↔S | C↔S | S→C | S→C | Pub/Sub | C→S |
| Streaming | No native | Subs only | All 4 modes | Yes | S→C only | No (single POST) | Yes | No |
| Schema | OpenAPI | SDL | .proto | None | None | None | Topic conventions | WSDL |
| Browser-native | Yes | Yes | gRPC-Web only | Yes | Yes | Receiver only | Via WebSocket bridge | Yes (heavy) |
| Caching | HTTP cache 🟢 | App-level / persisted queries | Per-RPC, custom | None | Limited | None | Retained messages | None |
| Versioning | URI / header | Field-level / @deprecated | Field numbers | Custom | Custom | Custom | Topic versioning | Namespaces |
| Best for | CRUD over the web | Client-specific shapes, FE-driven | Service-to-service, polyglot | Real-time bidi | One-way push | Event delivery between orgs | IoT, fanout | Legacy enterprise |
| Worst for | Real-time, large field control | Public caching, file upload | Browsers, debugging | One-way only, scaling | Bidi | Reliability-on-receiver | Request/response | Greenfield 2026 work |

---

## 12. Decision framework

The interview line you want to be able to fire off: *"It depends on cardinality, latency budget, who initiates, payload shape, and the consumer environment. Walk me through your traffic and I'll pick."*

Then:

1. **Browser ⇄ public web?** REST or GraphQL. GraphQL if frontend devs need flexible queries / mobile bandwidth matters.
2. **Service ⇄ service inside your VPC?** gRPC. Polyglot, fast, streams.
3. **Need real-time push to browser?**
   - One-way (dashboards, AI streams): SSE.
   - Two-way (chat, multiplayer): WebSocket.
4. **Need to notify *another company's* server when something happens?** Webhook.
5. **Lots of devices, intermittent net, fan-out broadcast?** MQTT.
6. **Big legacy financial / government integration?** Probably SOAP.
7. **Mobile-heavy traffic, lossy networks?** HTTP/3 / QUIC underneath whichever protocol you chose.

A nuance to mention: protocols are *layered*. GraphQL subscriptions run over WebSocket. gRPC runs over HTTP/2. MQTT can run over WebSocket. So "WebSocket vs MQTT" isn't a clean dichotomy — they sit at different layers.

---

## 13. Cross-cutting concerns

### Idempotency

Critical for any non-GET that may be retried.

**Pattern: client supplies an `Idempotency-Key`** (UUID). Server stores `(key → result)` for ~24h. On retry, returns the cached result instead of double-charging.

```http
POST /payments HTTP/1.1
Idempotency-Key: 4a3c-...

{"amount": 100, "currency": "USD"}
```

Stripe popularized this; you'll see it asked in interviews. Be ready to draw the storage model:

```sql
CREATE TABLE idempotency_keys (
  key TEXT PRIMARY KEY,
  request_hash TEXT NOT NULL,
  response_status INT,
  response_body JSONB,
  created_at TIMESTAMPTZ DEFAULT now()
);
-- TTL via background job or PostgreSQL pg_cron
```

If the same key arrives with a *different* body → 422 (Stripe returns this).

### Retries & backoff

Standard: **exponential backoff with full jitter**.

```js
const delay = Math.min(MAX, base * 2 ** attempt);
const sleep = Math.random() * delay; // full jitter — best for thundering herd
```

Always cap; always combine with circuit breaker; never retry on 4xx (except 408, 425, 429).

### Versioning strategies

| Strategy | Where it lives | Example | Notes |
|---|---|---|---|
| URI versioning | `/v1/users` | Twitter, GitHub | Simple, ugly, easy CDN cache |
| Header versioning | `Accept: application/vnd.x.v2+json` | GitHub V3 | Clean, harder for clients |
| Query param | `?v=2` | Hacky | Acceptable for internal |
| Date-pinned | `Stripe-Version: 2024-04-10` | Stripe | Excellent: each API key pins; backend can transform old → new |
| Field-level deprecation | `@deprecated` in GraphQL | Facebook | No big-bang versioning |
| Field numbers | proto3 | gRPC | Add fields, never remove/repurpose |

### Pagination

| Style | Pros | Cons |
|---|---|---|
| Offset (`?page=2&size=20`) | Easy, jumpable | Slow on big tables, inconsistent across writes |
| Cursor (`?after=eyJ...&limit=20`) | Stable, fast | No "jump to page 5" |
| Keyset (`?after_id=42&limit=20`) | Index-friendly | Sort key must be stable & unique |
| Token-based (opaque) | Server controls semantics | Less debuggable |

For SDE2 interviews, recommend **cursor/keyset** by default, mention `Link: <...>; rel="next"` headers for REST.

### Authentication overview

| Scheme | Where | Notes |
|---|---|---|
| Bearer + JWT | REST/GraphQL | Stateless, signed; revocation needs allowlist |
| Bearer + opaque + introspect | REST | DB roundtrip; easy revocation |
| OAuth 2.0 + PKCE | Public clients | Standard for SPA/mobile |
| API keys | B2B | Per-key rate-limits, scopes |
| HMAC | Webhooks | Sign payload with shared secret |
| mTLS | Service-to-service | Strong, ops-heavy |
| Cookie + CSRF | Browser flows | Old, still common |

### Rate limiting

Algorithms:
- **Fixed window** — simple, prone to bursts at boundary.
- **Sliding window** — smoother, more state.
- **Token bucket** — most common (allows bursts).
- **Leaky bucket** — fixed-rate output.

Always return `429` with `Retry-After` and ideally `X-RateLimit-Limit`, `X-RateLimit-Remaining`, `X-RateLimit-Reset` headers.

### Observability

Every protocol must be traceable. Standard: **OpenTelemetry** spans propagated via `traceparent`/`tracestate` headers (REST/GraphQL/gRPC metadata/MQTT user properties). Mention this — interviewers love it.

---

## 14. SDE2 interview questions

Each question below is the kind your interviewer will lob *casually*. The answer model is: **definition → trade-off → concrete example → edge case**.

### Conceptual & comparison

**Q1. When would you pick gRPC over REST?**

> gRPC shines for internal service-to-service comms where you control both sides. The big wins are: schema-first contracts via .proto with generated stubs in every language, ~3–10× smaller payloads with Protobuf, multiplexed HTTP/2 streams (so a chatty service-mesh doesn't burn TCP connections), and first-class deadlines + cancellation that propagate across hops.
>
> I'd avoid gRPC for browser-facing APIs (need gRPC-Web + proxy and bidi streaming is unsupported), for public APIs where consumers expect REST + curl, and for cases where binary debuggability hurts more than throughput helps.
>
> Concrete: order-service ↔ inventory-service in a Spring Boot mesh → gRPC. Mobile app ↔ order-service → REST or GraphQL.

**Q2. REST vs GraphQL trade-offs?**

> REST gives you free HTTP caching, simple debugging, and obvious resource boundaries. GraphQL gives clients the exact shape they need in one round trip, strong typing via SDL, no v1/v2 versioning (use `@deprecated`), and great mobile bandwidth characteristics.
>
> REST struggles with mobile clients that need 7 resources to render a screen — N round trips, or over-fetch. GraphQL struggles with HTTP caching (single POST endpoint), file uploads (need multipart extension), and runaway queries (need depth limits and persisted queries).
>
> Practical hybrid that some teams pick: REST for resource CRUD and uploads, GraphQL on top for read-heavy view-models.

**Q3. WebSocket vs SSE vs long-polling?**

> All three give server → client push. Long-poll is the lowest-common-denominator — the client opens a GET, server holds until something happens or a timeout, then client immediately re-polls. SSE is HTTP-native one-way push with auto-reconnect using Last-Event-ID. WebSocket is full-duplex with its own frame layer after an HTTP upgrade.
>
> If I only need server → client (dashboards, log streams, AI token streaming), I'd pick SSE — it's simpler, plays with HTTP infra, and the browser handles reconnects. If I need bidi (chat, collab editing), WebSocket. Long-poll is a fallback for hostile networks where WS/SSE are blocked.

**Q4. How do you scale a WebSocket service to 1M concurrent connections?**

> Three concerns: connection capacity per server, message routing, and statefulness.
>
> 1. **Per-server capacity.** Tune file-descriptor limits, kernel net.core.somaxconn, and use an event-loop server (Node, Netty, Vert.x). 100k–250k connections per box is achievable.
> 2. **Sticky sessions** at the LB so reconnects find their state. Use `ip_hash` or cookie-based affinity.
> 3. **Cross-server fan-out** via Redis Pub/Sub, NATS, or Kafka — every server subscribes to interesting topics and forwards to its local sockets.
> 4. **Reconnect with resume tokens** — clients carry last-message-id so I can replay missed messages from a short-lived Redis stream.
> 5. **Heartbeats** every 30s + close on missed pong, otherwise NAT timeouts silently kill connections and the server keeps state forever.
> 6. **Backpressure**: bounded send queue per connection, drop or disconnect slow clients.
> 7. **Auth at handshake**, not on each message, via JWT in subprotocol field or pre-set cookie.

**Q5. Idempotency — how would you implement it?**

> Two pieces: a server-side store `(idempotency_key → response_or_state)` with a TTL (~24h), and clients sending an `Idempotency-Key` header on POSTs that could be retried.
>
> First request: insert key with `IN_PROGRESS`, do the work, write the result, return it. Retry with the same key + same body → return the cached result. Retry with the same key + *different* body → 422 (semantic mismatch). Concurrent requests with the same key → second blocks or returns conflict.
>
> Watch out for: race on first request (use `INSERT ... ON CONFLICT` or Redis `SET NX`), TTL eviction colliding with very slow retries, and storing the request hash so we can detect mismatched bodies.

**Q6. How do MQTT and Kafka differ?**

> Both pub/sub, very different use cases.
>
> **MQTT** is a wire protocol designed for lightweight push to many subscribers, often IoT. Brokers (Mosquitto, EMQX) handle delivery, QoS 0/1/2, retained messages, and last-will. Messages are typically transient — once delivered (or expired), gone.
>
> **Kafka** is a distributed *log*: every message lands on disk in a partition, retained for days/weeks/forever. Consumers track their own offsets and can replay history. Designed for analytical and high-throughput pipelines.
>
> One liner: MQTT is "broadcast me now"; Kafka is "store every event so anyone can replay it."

**Q7. How does HTTP/3 differ from HTTP/2?**

> Semantics are the same — same verbs, same headers, same status codes. The transport changed:
>
> - HTTP/2 runs on TCP. One TCP packet loss stalls all multiplexed streams (TCP head-of-line blocking).
> - HTTP/3 runs on QUIC, which is built on UDP. Each stream has its own loss recovery → no transport-level HOL.
> - QUIC bundles TLS 1.3 in the handshake — one round trip to encrypted, 0-RTT on resumption.
> - QUIC uses connection IDs decoupled from IP → connection migration when your phone switches Wi-Fi → cellular.
>
> Trade-off: UDP is sometimes throttled/blocked, so HTTP/3 servers advertise via `Alt-Svc` header and fall back to HTTP/2.

**Q8. How do you secure webhooks?**

> Six layers I'd put in:
> 1. HMAC-SHA256 signature header with a per-endpoint secret, signed over `timestamp + raw_body`.
> 2. Reject timestamps older than 5 minutes (anti-replay).
> 3. Idempotency on event ID so retries don't double-process.
> 4. IP allowlist if the sender publishes ranges (Stripe, GitHub).
> 5. mTLS for B2B integrations where it matters.
> 6. Constant-time signature comparison (`timingSafeEqual`/`MessageDigest.isEqual`) to dodge timing attacks.
>
> Implementation pitfall: verify signature against *raw* body. JSON.parse → JSON.stringify rewrites whitespace and breaks HMAC.

**Q9. Walk me through designing a real-time chat app.**

> Anchor on WebSocket for client ↔ server. Then:
> - **Connection layer**: WS gateway service behind a sticky LB. Auth at handshake.
> - **Message bus**: Redis Streams or Kafka for cross-server fan-out + short replay window.
> - **Storage**: Cassandra/DynamoDB for messages (write-heavy, time-keyed); Postgres for users/rooms/membership.
> - **Presence**: Redis with TTL per user (refresh on heartbeat).
> - **Delivery semantics**: at-least-once + client-side dedupe by message ID. Server assigns monotonically increasing IDs per room.
> - **Push notifications** for offline users via APNs/FCM workers consuming the same bus.
> - **Read receipts**: dedicated topic or last-read-id per (user, room).
> - **Scaling**: shard by `room_id` so a hot room stays on one Kafka partition.
>
> Talking points for depth: backpressure, reconnect with resume token, encryption (end-to-end with double ratchet vs server-stored).

**Q10. Walk me through designing an IoT ingest pipeline for 1M devices.**

> - **Edge → broker**: MQTT over TLS 8883. QoS 1 by default; QoS 2 for critical commands. Last-Will tells the broker "I'm offline."
> - **Broker tier**: HiveMQ or EMQX cluster. Use shared subscriptions (MQTT 5 `$share/...`) to load-balance consumer workers.
> - **Stream out**: broker → Kafka bridge for downstream analytics, ML, and audit trail.
> - **Command-down**: dashboards publish to `device/<id>/commands`; device subscribes at QoS 1.
> - **State store**: retained message on `device/<id>/state` keeps the last-known state for new subscribers.
> - **Identity**: per-device certificate, broker authn via mTLS, topic ACLs scoped to device ID.
> - **Observability**: every PUBLISH carries OpenTelemetry trace ID in user properties (MQTT 5).

**Q11. Solve N+1 in a GraphQL resolver.**

> The default resolver fires one DB query per parent → 1 + N queries. Fix with DataLoader: each batch of `.load(id)` within a single tick is coalesced into one `WHERE id IN (...)` query, results mapped back by key.
>
> Two implementation notes:
> 1. Create one DataLoader instance *per request* so caching doesn't leak across users.
> 2. The returned array order must match the input keys order — easy to mess up if you map by ID.
>
> In Spring for GraphQL the equivalent is `@BatchMapping`. In Hibernate-land you can also fix it upstream with `JOIN FETCH` or entity graphs, but DataLoader is the right answer for the GraphQL layer.

**Q12. How do you do API versioning without breaking everyone?**

> Three principles: (a) additive changes are free, (b) breaking changes cost a version bump, (c) deprecate-before-remove with telemetry.
>
> For REST/JSON I'd use header versioning (Accept media type) for clean URLs, and pin API keys to a default version on first use (like Stripe). Every breaking version sets up a transformation layer: old request → canonical internal model → new response.
>
> For GraphQL I'd avoid v1/v2 entirely and lean on field-level `@deprecated(reason: ...)` with usage metrics so I know when it's safe to remove.
>
> For gRPC, evolve fields by adding new field numbers — never reuse, never change wire types. Mark removed numbers `reserved`.

**Q13. Compare WebSocket scaling vs SSE scaling.**

> SSE: each connection is an open HTTP/1.1 or HTTP/2 request. Browsers cap 6 HTTP/1.1 connections per origin — use HTTP/2 to multiplex many SSE streams over one TCP. No sticky-session requirement if all server data flows through a shared bus; clients just reconnect with Last-Event-ID and the new server replays from there.
>
> WebSocket: each connection is stateful on its server, sticky sessions are usually needed because you don't want to redo authentication and channel state on every reconnect. Scaling looks the same as I described in Q4: pub/sub bus + sticky LB + resume tokens.
>
> Key win for SSE in scaling: no sticky-session requirement → easier blue-green deploys and dynamic scale-down. WebSocket wins where bidi is non-negotiable.

**Q14. Caching trade-offs across protocols.**

> REST gets HTTP caching for free: ETags, Cache-Control, CDN-level reuse. Best caching story by far.
>
> GraphQL: out of the box, no HTTP cache (POST to one URL). Fixes: persisted queries (turn into deterministic GETs), normalized client cache (Apollo/Relay), CDN-level cache with query hash in URL.
>
> gRPC: no HTTP semantics consumers can rely on. Caching is per-RPC, application-defined, often Redis-backed.
>
> WebSocket / SSE / MQTT: caching doesn't fit the model — they're streams, not idempotent reads. You'd cache *upstream* of them (the data they emit).
>
> When evaluating a protocol switch from REST → GraphQL, "we'll lose CDN cache" is a real cost to surface.

**Q15. Bonus: difference between Kafka consumer groups and MQTT shared subscriptions.**

> Both load-balance messages across multiple workers. Kafka does it at the *partition* level — each partition is owned by exactly one consumer in the group, so message order is preserved per partition. MQTT 5 shared subscriptions (`$share/<group>/<topic>`) load-balance arbitrary subscribers with no per-key affinity guarantees by default. If you need MQTT with ordering per key, you either pick a broker that adds key-based routing (EMQX has features) or move to Kafka.

### Deeper / curveball questions

**Q16. Why does WebSocket have a magic GUID in the handshake?**

> RFC 6455 specifies that `Sec-WebSocket-Accept` is `Base64(SHA1(client_key + "258EAFA5-E914-47DA-95CA-C5AB0DC85B11"))`. The GUID is a constant defined in the RFC. Its purpose is *not* security (it's not a secret — it's in the RFC). It's to make sure both ends are speaking WebSocket and that an HTTP cache or proxy isn't accidentally completing the handshake.

**Q17. Why does proto3 not support `required` fields anymore?**

> Because `required` is a contract you can't safely loosen later. Once you mark a field required, removing it breaks every reader. Google's internal experience showed it caused more outages than it prevented. Proto3 makes everything optional-by-default (zero values used as "unset"), and adds `optional` keyword back in 3.15+ for genuine three-state needs (set / unset / explicitly null).

**Q18. What's the practical difference between gRPC streaming and a WebSocket?**

> Both give you bidi over one connection. gRPC streaming is *typed* — the schema declares message types and direction — and uses HTTP/2 frames managed by the gRPC library. WebSocket is *untyped* — you frame whatever bytes you want. gRPC bidi gives you flow control, deadlines, and structured error codes; WebSocket gives you nothing of those (you reinvent each).
>
> Use gRPC streaming for service-to-service. Use WebSocket for browser clients (gRPC-Web doesn't do client/bidi streaming).

**Q19. Can you do request/response over MQTT?**

> Yes, with MQTT 5's `response_topic` and `correlation_data` properties. Publisher sends a request to `commands/x` with `response_topic = responses/<client-id>/<correlation>`. Replier subscribes to `commands/x`, processes, and publishes to the response topic. Correlation data uniquely matches the response back. Before MQTT 5 you'd encode this in payload yourself.
>
> That said: if your dominant pattern is request/response, MQTT is the wrong tool. gRPC or REST will give you better tooling.

**Q20. What's HTTP/2 server push and why is it deprecated?**

> The idea: server can push resources the client is *about to* request (e.g., CSS the index page will need). In practice, browsers couldn't tell which pushes were useful vs already-cached → either over-pushed (waste) or under-pushed (wasted RTT). Chrome 106 (2022) removed support. Preload `Link` headers replaced it. Mention this if asked about HTTP/2 — shows you stay current.

**Q21. How does QUIC handle 0-RTT, and what are the security caveats?**

> If client and server have talked before, the client can send application data in the *first* packet (alongside the TLS resumption ticket). Massive latency win — sub-RTT page loads.
>
> Security caveat: 0-RTT data is *replayable*. An attacker who captures the encrypted first packet can resend it. So 0-RTT must only carry idempotent requests (GETs). POSTs go in 1-RTT or later. TLS 1.3 spec is explicit about this; HTTP/3 servers reject non-idempotent verbs over 0-RTT.

**Q22. Compare REST's HATEOAS to GraphQL's introspection.**

> HATEOAS embeds links in responses to tell clients what they can do next (`_links`). Introspection (GraphQL) exposes the *whole schema* via a `__schema` query — clients can discover types and fields at runtime. They both aim for "self-describing API" but at different layers: HATEOAS is per-resource hyperlinks at runtime; introspection is a global type catalog at design time.

**Q23. How do you implement fair multi-tenant rate limiting on a public REST API?**

> Token bucket per API key + global token bucket per tenant + IP-based fallback for unauthenticated. Stored in Redis with `INCR` + `EXPIRE`. Use sliding-window-counter to smooth boundary bursts. Return `429` with `Retry-After`, plus headers describing remaining budget. Don't forget *expensive endpoint weighting* — searches cost 10 tokens, cached GETs cost 1. This rewards efficient clients.

**Q24. Talk about cancellation and deadlines in gRPC.**

> Caller sets a deadline (`stub.withDeadlineAfter(2s)`). The deadline propagates as gRPC metadata to the callee. The callee's stub knows the deadline and can refuse to forward downstream calls that would exceed it.
>
> When the client cancels (or hits the deadline), the gRPC library raises `CANCELLED`/`DEADLINE_EXCEEDED` on the server side via `Context.isCancelled()`. Server-side code should poll this in long loops and bail early — otherwise it does work that no one will read.
>
> This is *the* feature REST clients reinvent badly. AbortController in fetch is the JS equivalent but doesn't propagate across hops.

**Q25. When would you choose long-polling in 2026?**

> Almost never as a first choice, but as a fallback for environments where WebSocket and SSE are blocked (corporate proxies, ancient firewalls). Libraries like Socket.IO and SignalR auto-negotiate to long-polling. Also useful for AWS API Gateway's HTTP integration if you want server push without setting up the WebSocket gateway.

---

## Quick study cheat-sheet

If you have 30 minutes before an interview, memorize this:

- **REST** — JSON over HTTP, verbs map to CRUD, HTTP cache is free (ETag + Cache-Control), versioning via URI/header/date-pin. Mention HATEOAS, status codes (401 vs 403, 412 for ETag mismatch, 422 vs 400), idempotency keys, ProblemDetail (RFC 7807), JSON Patch vs Merge Patch, cursor/keyset pagination over offset.
- **GraphQL** — single endpoint, schema-first, client picks shape, N+1 → DataLoader, subscriptions on WS, errors come back 200.
- **gRPC** — Protobuf over HTTP/2, 4 streaming modes, deadlines + cancellation propagate, codegen across languages. Not browser-friendly without gRPC-Web.
- **WebSocket** — HTTP upgrade → bidi frames, scaling needs sticky sessions + pub/sub fan-out + heartbeats + resume tokens.
- **SSE** — server → client only, browser-native EventSource, Last-Event-ID for auto-resume. Underrated for AI/dashboards.
- **Webhook** — server-to-server callback, signed HMAC, short replay window, dedupe on event ID, sender does exponential backoff.
- **HTTP/3 + QUIC** — UDP-based, TLS baked in, 0-RTT, no TCP HOL blocking, connection migration. Same HTTP semantics.
- **MQTT** — pub/sub with broker, QoS 0/1/2, retained + LWT, perfect for IoT.
- **SOAP** — XML envelopes (1.1 vs 1.2 namespace + content-type differ), WSDL contract = Types/Messages/PortType/Binding/Service, doc/lit-wrapped style, WS-Security (element-level sign+encrypt, beats TLS for intermediaries), WS-Addressing (`ReplyTo` for async), WS-ReliableMessaging, MTOM for attachments, faults always return HTTP 500, contract-first via XSD + Spring-WS / JAX-WS. Alive in banking/gov/telecom/healthcare.
- **Cross-cutting** — idempotency keys, exp-backoff + jitter, OpenTelemetry, cursor pagination, OAuth 2.0 + PKCE.

Write each of these on a flashcard. Be able to draw a sequence diagram of *each protocol's first 3 wire interactions* on a whiteboard. That single skill — sketching protocol diagrams from memory — separates SDE2 from SDE1 in interviews.

Good luck.
