# API Design 101 — Design, Performance, and Optimization

> **Who this is for**: Engineers who build and maintain APIs and need to go beyond "just make it work" — covering REST design, latency reduction, payload optimization, query tuning, caching, security, and reliability. Every concept is practical and immediately applicable.

---

## Table of Contents

1. [API Design Fundamentals](#1-api-design-fundamentals)
2. [HTTP Deep Dive](#2-http-deep-dive)
3. [REST API Design — The Right Way](#3-rest-api-design--the-right-way)
4. [Request and Response Design](#4-request-and-response-design)
5. [Pagination, Filtering, and Sorting](#5-pagination-filtering-and-sorting)
6. [API Versioning](#6-api-versioning)
7. [Error Handling](#7-error-handling)
8. [GraphQL](#8-graphql)
9. [gRPC and Protocol Buffers](#9-grpc-and-protocol-buffers)
10. [Webhooks and Async APIs](#10-webhooks-and-async-apis)
11. [Latency Optimization](#11-latency-optimization)
12. [Payload Optimization and Compression](#12-payload-optimization-and-compression)
13. [HTTP Caching](#13-http-caching)
14. [Query Optimization for APIs](#14-query-optimization-for-apis)
15. [Connection Management](#15-connection-management)
16. [Idempotency and Retries](#16-idempotency-and-retries)
17. [Rate Limiting and Throttling](#17-rate-limiting-and-throttling)
18. [API Security](#18-api-security)
19. [API Reliability Patterns](#19-api-reliability-patterns)
20. [API Gateway Patterns](#20-api-gateway-patterns)
21. [API Documentation and Contracts](#21-api-documentation-and-contracts)
22. [Observability for APIs](#22-observability-for-apis)
23. [API Design Checklist](#23-api-design-checklist)
24. [Glossary](#24-glossary)

---

## 1. API Design Fundamentals

### What Makes an API Good?

A good API is:

| Property | What it means |
|---|---|
| **Correct** | Does what it says, no surprises |
| **Consistent** | Same conventions everywhere — naming, errors, pagination |
| **Predictable** | Knowing one endpoint teaches you all others |
| **Minimal** | No unnecessary fields, parameters, or endpoints |
| **Evolvable** | Adding new features doesn't break existing clients |
| **Secure** | Exposes only what clients need, validates everything |
| **Performant** | Fast enough for the use case, doesn't leak queries to caller |

### API Styles Comparison

| | REST | GraphQL | gRPC | WebSocket |
|---|---|---|---|---|
| Transport | HTTP | HTTP | HTTP/2 | TCP |
| Format | JSON/XML | JSON | Protocol Buffers | Any |
| Schema | Optional (OpenAPI) | Mandatory | Mandatory (proto) | None |
| Over-fetching | Common | None (client selects fields) | None | N/A |
| Under-fetching | Common (multiple round trips) | None (one query) | None | N/A |
| Caching | Easy (HTTP cache) | Hard (all POST) | Hard | None |
| Browser support | Native | Native | Needs proxy | Native |
| Streaming | SSE/WebSocket | Subscriptions | Native bidirectional | Native |
| Best for | Public APIs, CRUD | Mobile, complex data graphs | Internal service-to-service | Real-time bidirectional |

**Decision guide**:
- Public API consumed by many client types → REST
- Mobile app with complex, shaped data needs → GraphQL
- Internal microservices, high throughput, streaming → gRPC
- Real-time chat, live collaboration, gaming → WebSocket
- Server→client push only (feeds, live scores) → SSE

---

## 2. HTTP Deep Dive

### HTTP Methods — Semantics Matter

| Method | Idempotent | Safe | Body allowed | Use |
|---|---|---|---|---|
| GET | Yes | Yes | No | Retrieve resource |
| HEAD | Yes | Yes | No | Get headers only (check existence, Last-Modified) |
| OPTIONS | Yes | Yes | No | Preflight for CORS, list allowed methods |
| POST | **No** | No | Yes | Create resource, trigger action |
| PUT | Yes | No | Yes | Replace entire resource |
| PATCH | **No** | No | Yes | Partial update |
| DELETE | Yes | No | No/Yes | Delete resource |

**Idempotent** = calling N times = calling 1 time. Network retries are safe.
**Safe** = no side effects on server.

```
GET  /users/123      → returns user, no state change
PUT  /users/123      → replaces user (repeated PUT → same result)
POST /orders         → creates order (repeated POST → duplicate orders!)
DELETE /orders/456   → deletes order (delete again → 404 or 204, state unchanged)
PATCH /users/123     → updates email field only
```

### Status Codes — Use Them Right

**2xx Success**:
```
200 OK             → GET successful, PUT/PATCH successful (returns body)
201 Created        → POST successful (include Location header with new resource URL)
202 Accepted       → Request received, processing async (return 202 for long-running ops)
204 No Content     → DELETE successful, PUT with no body response needed
206 Partial Content → Range request served (video streaming, resumable downloads)
```

**3xx Redirect**:
```
301 Moved Permanently  → Resource moved, update your bookmark. Browser caches forever.
302 Found              → Temporary redirect (don't cache)
304 Not Modified       → ETag/Last-Modified matched, use your cached version (no body sent)
307 Temporary Redirect → Like 302, but preserve HTTP method (POST stays POST)
308 Permanent Redirect → Like 301, but preserve HTTP method
```

**4xx Client Error**:
```
400 Bad Request         → Malformed JSON, invalid parameter type
401 Unauthorized        → Not authenticated (must login)
403 Forbidden           → Authenticated but not authorized for this resource
404 Not Found           → Resource doesn't exist
405 Method Not Allowed  → Correct URL but wrong method (GET on a POST-only endpoint)
409 Conflict            → State conflict (optimistic locking failure, duplicate creation)
410 Gone                → Resource permanently deleted (stronger than 404)
412 Precondition Failed → If-Match header didn't match (ETag conflict)
415 Unsupported Media   → Content-Type not accepted
422 Unprocessable Entity→ Syntactically valid but semantically wrong (validation failure)
429 Too Many Requests   → Rate limited (include Retry-After header)
```

**5xx Server Error**:
```
500 Internal Server Error → Unhandled exception — should never leak stack traces
502 Bad Gateway           → Upstream service returned invalid response
503 Service Unavailable   → Server overloaded or in maintenance (include Retry-After)
504 Gateway Timeout       → Upstream service timed out
```

**Critical distinction**:
- 400 = client's fault → don't retry the same request
- 500 = server's fault → client CAN retry (with backoff)
- 503/504 → transient, retry with backoff

### HTTP Headers You Must Know

**Request headers**:
```
Authorization: Bearer <jwt_token>
Content-Type: application/json
Accept: application/json, application/xml;q=0.9
Accept-Encoding: gzip, br
Accept-Language: en-US,en;q=0.9
If-None-Match: "abc123"         (conditional GET — return 304 if unchanged)
If-Modified-Since: Wed, 01 Jan 2025 00:00:00 GMT
Idempotency-Key: uuid-abc-123   (safe retries for non-idempotent ops)
X-Request-ID: uuid-xyz-789      (correlation ID for distributed tracing)
X-Forwarded-For: 192.168.1.1   (real client IP behind proxy/LB)
X-API-Version: 2                (version negotiation via header)
Range: bytes=0-1023             (request first 1 KB for resumable downloads)
```

**Response headers**:
```
Content-Type: application/json; charset=utf-8
Content-Encoding: gzip
Content-Length: 1234
Cache-Control: public, max-age=3600
ETag: "abc123def456"
Last-Modified: Wed, 01 Jan 2025 12:00:00 GMT
Location: /api/v1/orders/456   (response to 201 Created or 3xx redirect)
X-Request-ID: uuid-xyz-789     (echo back for tracing)
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 42
X-RateLimit-Reset: 1704106800  (epoch timestamp when limit resets)
Retry-After: 30                (seconds to wait — for 429 and 503)
Strict-Transport-Security: max-age=31536000; includeSubDomains
X-Content-Type-Options: nosniff
X-Frame-Options: DENY
Access-Control-Allow-Origin: https://app.niyo.com
```

### Keep-Alive and Connection Reuse

HTTP/1.1 defaults to persistent connections (`Connection: keep-alive`). The server keeps the TCP connection open after the response. Client reuses it for the next request.

```
Without keep-alive (HTTP/1.0):
  TCP handshake (1.5 RTT) + TLS handshake (1-2 RTT) for EVERY request

With keep-alive (HTTP/1.1):
  TCP + TLS handshake once → multiple requests/responses on same connection
  
Typical savings: 20–100ms per request
```

Server configuration:
```
# Nginx
keepalive_timeout 65s;
keepalive_requests 100;  # max requests per connection

# Tomcat (Spring Boot)
server.tomcat.keep-alive-timeout=60000   # ms
server.tomcat.max-keep-alive-requests=100
```

---

## 3. REST API Design — The Right Way

### URL Design Principles

URLs identify **resources** (nouns), not actions (verbs). HTTP methods are the verbs.

```
# Bad: verbs in URLs
GET  /getUser?id=123
POST /createOrder
POST /deleteTransaction?id=456
GET  /fetchAllCorporates

# Good: nouns, hierarchical, plural
GET    /users/123
POST   /orders
DELETE /transactions/456
GET    /corporates
```

**Hierarchy = ownership**:
```
/corporates/{corpId}/employees          → employees of a corporate
/corporates/{corpId}/employees/{empId}  → specific employee
/corporates/{corpId}/payrolls           → payrolls for a corporate
/orders/{orderId}/items                 → items in an order
/orders/{orderId}/items/{itemId}        → specific item in order
```

**Max 2–3 levels of nesting**. Deeper than that → flatten with query params.
```
# Instead of:
/corporates/{corpId}/departments/{deptId}/employees/{empId}/transactions

# Do:
/transactions?employeeId=xxx&deptId=yyy&corpId=zzz
```

**Resource naming**:
```
# Plural nouns for collections
/users, /orders, /transactions, /corporates

# Singular for singleton resources
/users/{id}/profile      (user has one profile)
/system/health           (one system health)

# Use kebab-case for multi-word resources
/fund-transfers, /inward-remittances, /penny-checks

# Lowercase always — URLs are case-sensitive on Linux
GET /Users/123 ≠ GET /users/123
```

**Actions that don't fit CRUD**:
```
# Option 1: sub-resource for the action
POST /orders/{id}/cancel
POST /accounts/{id}/freeze
POST /corporates/{id}/approve
POST /transactions/{id}/retry

# Option 2: treat as an event/action resource
POST /password-resets      (creates a password reset event)
POST /email-verifications  (creates a verification event)

# Option 3: RPC-style (acceptable for operations, not state)
POST /bulk-payments/process
POST /payrolls/disburse
```

### URL Parameters: Path vs Query

**Path parameters** (`/users/{id}`):
- Required, identifies a specific resource
- Changes what resource you're looking at

**Query parameters** (`/users?role=ADMIN&status=ACTIVE`):
- Optional, modifies how the collection is returned
- Filtering, sorting, pagination, field selection

```
GET /transactions/{transactionId}          → path: identifies the resource
GET /transactions?status=PENDING           → query: filters collection
GET /transactions?page=2&size=20           → query: pagination
GET /transactions?sort=createdAt,desc      → query: sorting
GET /transactions?fields=id,amount,status  → query: field projection
```

---

## 4. Request and Response Design

### Request Design

**Be strict about what you accept**:
```json
// POST /orders — clear, typed, no ambiguity
{
  "corporateId": "corp-123",
  "amount": {
    "value": 50000.00,
    "currency": "INR"
  },
  "beneficiary": {
    "accountNumber": "1234567890",
    "ifsc": "HDFC0001234",
    "name": "Acme Pvt Ltd"
  },
  "reference": "INV-2024-001",
  "scheduledAt": "2024-01-15T14:30:00+05:30"   // ISO 8601 always
}
```

**Field naming**: pick one and never deviate.
```
camelCase:  { "createdAt": "...", "corporateId": "..." }  → JSON APIs, JavaScript clients
snake_case: { "created_at": "...", "corporate_id": "..." } → Python, Ruby clients
kebab-case: not for JSON keys (hyphens not valid identifiers)
```

Use `camelCase` for Java/JavaScript APIs. Be consistent everywhere.

**Use semantic types, not primitives**:
```json
// Bad: stringly-typed
{ "amount": "50000.00", "date": "2024-01-15", "status": "1" }

// Good: correct types
{ "amount": 50000.00, "date": "2024-01-15", "status": "PENDING" }

// Better: rich types for domain concepts
{ 
  "amount": { "value": 50000.00, "currency": "INR" },
  "scheduledAt": "2024-01-15T14:30:00+05:30",
  "status": "PENDING"
}
```

**Dates and times**: always ISO 8601, always include timezone.
```
"2024-01-15"                  → date only
"2024-01-15T14:30:00Z"       → UTC
"2024-01-15T14:30:00+05:30"  → IST
"2024-01-15T14:30:00.123Z"   → with milliseconds

Never return epoch timestamps to clients — not human-readable, timezone-ambiguous.
Store in UTC, return UTC or offset-aware ISO 8601.
```

**Amounts and money**: never use float for money.
```json
// Bad: floating point precision errors
{ "amount": 0.1 }  // stored as 0.10000000000000000555...

// Good: integer minor units or decimal string
{ "amount": 10000, "currency": "INR", "unit": "PAISE" }  // 100.00 INR
{ "amount": "100.00", "currency": "INR" }                  // string for exact decimal
// Or BigDecimal on the server, serialized to 2-decimal string
```

### Response Design

**Consistent envelope** — wrap all responses in a standard structure:

```json
// Success response
{
  "status": "success",
  "data": {
    "transactionId": "txn-abc-123",
    "amount": 50000.00,
    "status": "INITIATED"
  },
  "meta": {
    "requestId": "req-xyz-789",
    "timestamp": "2024-01-15T14:30:00Z"
  }
}

// Collection response
{
  "status": "success",
  "data": [ ... ],
  "pagination": {
    "page": 1,
    "size": 20,
    "total": 1547,
    "hasNext": true,
    "cursor": "eyJpZCI6IjEyMyJ9"
  },
  "meta": {
    "requestId": "req-xyz-789"
  }
}

// Error response
{
  "status": "error",
  "error": {
    "code": "INSUFFICIENT_BALANCE",
    "message": "Account balance (₹1,000) is insufficient for transfer of ₹5,000",
    "field": "amount",          // for validation errors
    "details": [                // for multiple validation errors
      { "field": "beneficiary.ifsc", "code": "INVALID_IFSC", "message": "..." }
    ]
  },
  "meta": {
    "requestId": "req-xyz-789"
  }
}
```

**What NOT to put in responses**:
```
Internal IDs from your database implementation (auto-increment integer IDs expose scale)
Stack traces (security risk — exposes internals)
Passwords, secrets, keys — even hashed ones
Server hostnames, internal IP addresses
Implementation details ("retry in N ms" is fine, "Cassandra partition key" is not)
```

**Use business IDs, not sequential integers**:
```
# Sequential integer: exposes how many records you have
/orders/1247  → caller knows you have ~1247 orders

# Use UUIDs or Snowflake IDs
/orders/01HN2K3P5QR8T6M7N9B4C6D7E2

# Or a prefixed ID for human readability
/orders/ord_01HN2K3P5QR8T6M7N9B4C6D7E2
/users/usr_4kR9mN2pQs
```

### HATEOAS — Hypermedia Links

Tell clients what they can do next. Self-documenting APIs.

```json
{
  "data": {
    "orderId": "ord-123",
    "status": "PENDING",
    "_links": {
      "self":   { "href": "/orders/ord-123", "method": "GET" },
      "cancel": { "href": "/orders/ord-123/cancel", "method": "POST" },
      "items":  { "href": "/orders/ord-123/items", "method": "GET" },
      "payment":{ "href": "/orders/ord-123/payment", "method": "GET" }
    }
  }
}
```

Include only links the current user is authorized to use. Client doesn't need to construct URLs — follows links. Eliminates URL hardcoding in clients.

---

## 5. Pagination, Filtering, and Sorting

### Offset Pagination

```
GET /transactions?page=3&size=20
SELECT * FROM transactions LIMIT 20 OFFSET 60;

Response:
{
  "data": [...],
  "pagination": {
    "page": 3,
    "size": 20,
    "total": 5420,
    "totalPages": 271
  }
}
```

**Problems**:
- Deep pagination is slow: `OFFSET 10000 LIMIT 20` scans 10,020 rows, discards 10,000
- Inconsistent under inserts: new items shift pages → items appear twice or get skipped
- `COUNT(*)` for total is expensive on large tables

**When to use**: Small datasets (< 10K rows), admin UIs where users jump to specific pages.

### Cursor-Based Pagination (Keyset)

```
GET /transactions?limit=20&after=eyJpZCI6IjEyMzQ1In0=

# Cursor is base64-encoded sort position, not an offset
# e.g., cursor = base64({"id": "12345", "createdAt": "2024-01-15T14:30:00Z"})

SELECT * FROM transactions
WHERE created_at < :cursorTimestamp
   OR (created_at = :cursorTimestamp AND id < :cursorId)
ORDER BY created_at DESC, id DESC
LIMIT 21;  -- fetch 21, if 21 returned hasNext=true, return 20

Response:
{
  "data": [...],
  "pagination": {
    "limit": 20,
    "hasNext": true,
    "hasPrev": true,
    "nextCursor": "eyJpZCI6IjEyMzQ2In0=",
    "prevCursor": "eyJpZCI6IjEyMzI1In0="
  }
}
```

**Advantages**:
- O(log n) with index — no full scan regardless of depth
- Stable — inserts/deletes don't affect other pages
- Infinite scroll pattern (mobile apps)

**Disadvantages**:
- Can't jump to page N directly
- Sort column must be part of cursor
- Harder to implement

**When to use**: Large datasets, real-time feeds, mobile infinite scroll, any dataset > 10K rows.

### Page Size Limits

Always enforce maximum page size:
```java
@GetMapping("/transactions")
public Page<TransactionDTO> getTransactions(
    @RequestParam(defaultValue = "20") @Max(100) int size,
    @RequestParam(defaultValue = "0") @Min(0) int page) { ... }

// Never allow: GET /transactions?size=1000000
// It will return 1M records, serialize to huge JSON, OOM the server
```

### Filtering

Design a consistent filter syntax:

```
# Simple equality
GET /transactions?status=PENDING
GET /transactions?corporateId=corp-123&status=SUCCESS

# Range filters (common approaches)
GET /transactions?amountMin=1000&amountMax=50000
GET /transactions?createdAfter=2024-01-01&createdBefore=2024-01-31

# Multiple values (IN clause)
GET /transactions?status=PENDING,FAILED
GET /transactions?status[]=PENDING&status[]=FAILED

# Operators (advanced, for power APIs)
GET /transactions?amount[gte]=1000&amount[lte]=50000
GET /transactions?createdAt[after]=2024-01-01T00:00:00Z
```

**In Spring Boot**:
```java
// Use a filter DTO with @RequestParam
public record TransactionFilter(
    String corporateId,
    TransactionStatus status,
    @DateTimeFormat(iso = ISO.DATE_TIME) ZonedDateTime createdAfter,
    @DateTimeFormat(iso = ISO.DATE_TIME) ZonedDateTime createdBefore,
    Double amountMin,
    Double amountMax
) {}

@GetMapping("/transactions")
public Page<TransactionDTO> list(@ModelAttribute TransactionFilter filter,
                                  @ModelAttribute PageRequest pageRequest) {
    return transactionService.findAll(filter, pageRequest);
}
```

### Sorting

```
GET /transactions?sort=createdAt,desc
GET /transactions?sort=amount,asc&sort=createdAt,desc  (multi-sort)

# Spring Data style
GET /transactions?sort=createdAt,desc&sort=amount,asc
```

**Always validate sort fields against allowlist** to prevent SQL injection or disclosure:
```java
private static final Set<String> ALLOWED_SORT_FIELDS = Set.of(
    "createdAt", "amount", "status", "corporateId"
);

if (!ALLOWED_SORT_FIELDS.contains(sortField)) {
    throw new InvalidSortFieldException("Invalid sort field: " + sortField);
}
```

### Search

```
GET /corporates?q=niyo             (full-text search)
GET /employees?name=kunal          (field-specific)
GET /transactions?reference=INV*   (wildcard prefix)

# Dedicated search endpoint for complex search
POST /search
{
  "query": "niyo banking",
  "filters": { "status": "ACTIVE" },
  "sort": [{ "field": "relevance" }],
  "page": { "limit": 20 }
}
```

---

## 6. API Versioning

Never break clients. Once published, an API is a contract.

### What Counts as a Breaking Change?

```
BREAKING (requires version bump):
  - Remove a field from response
  - Rename a field
  - Change field type (string → integer)
  - Change URL structure
  - Remove an endpoint
  - Add a required request field
  - Change HTTP method for an endpoint
  - Change status code for a scenario
  - Change error codes

NON-BREAKING (safe to ship without version bump):
  - Add new optional request field
  - Add new field to response (clients should ignore unknown fields)
  - Add a new endpoint
  - Add a new optional query parameter
  - Strengthen validation (make optional field validated more strictly)
```

**Tell clients to ignore unknown fields** — design clients for forward compatibility.

### Versioning Strategies

**URL path versioning** (recommended):
```
/api/v1/transactions
/api/v2/transactions

Pros: visible, easy to route at LB/gateway, curl-friendly, easy to deprecate
Cons: "v" in URL breaks REST purity (resources shouldn't know their version)
```

**Header versioning**:
```
GET /transactions
API-Version: 2

Pros: Clean URLs
Cons: Harder to test (needs curl -H), caches miss (key includes header)
```

**Accept header (content negotiation)**:
```
Accept: application/vnd.niyo.v2+json

Pros: Follows HTTP spec
Cons: Verbose, awkward to test, unfamiliar to most developers
```

**Query parameter** (avoid for production):
```
GET /transactions?version=2

Cons: Gets cached by mistake, pollutes URLs, not discoverable
```

**For SDE2-level APIs**: use URL path versioning. It's pragmatic and universally understood.

### Deprecation Policy

```
1. Add Deprecation header to old version responses:
   Deprecation: true
   Sunset: Wed, 01 Jan 2026 00:00:00 GMT
   Link: <https://docs.niyo.com/api/v2/migration>; rel="successor-version"

2. Document in changelog when deprecation starts
3. Give clients at least 6 months before sunset
4. Monitor usage — who is still on v1? Alert them directly
5. Return 410 Gone after sunset date
```

---

## 7. Error Handling

### Error Response Standard (RFC 9457 — Problem Details)

Spring Boot 3+ implements this natively.

```json
{
  "type": "https://docs.niyo.com/errors/insufficient-balance",
  "title": "Insufficient Balance",
  "status": 422,
  "detail": "Account balance ₹1,000 is insufficient for transfer of ₹5,000",
  "instance": "/transactions/txn-123",
  "traceId": "4bf92f3577b34da6a3ce929d0e0e4736",
  "timestamp": "2024-01-15T14:30:00Z",
  "errors": [
    {
      "field": "amount",
      "code": "INSUFFICIENT_BALANCE",
      "message": "Available balance: ₹1,000. Requested: ₹5,000"
    }
  ]
}
```

**Error code design**:
```
Use business error codes, not HTTP status alone:

INSUFFICIENT_BALANCE   → 422
DUPLICATE_TRANSACTION  → 409
CORPORATE_SUSPENDED    → 403
INVALID_IFSC           → 400
BANK_API_TIMEOUT       → 503
RATE_LIMIT_EXCEEDED    → 429

Why codes? HTTP status is coarse (multiple 422s can mean different things).
Clients can programmatically handle INSUFFICIENT_BALANCE vs INVALID_IFSC differently.
```

### Validation Errors — Return All at Once

```java
// Don't fail fast for validation — return ALL errors in one response
// Saves client a round trip per error
{
  "status": 400,
  "title": "Validation Failed",
  "errors": [
    { "field": "amount", "code": "MUST_BE_POSITIVE", "message": "Amount must be > 0" },
    { "field": "beneficiary.ifsc", "code": "INVALID_FORMAT", "message": "IFSC must be 11 characters" },
    { "field": "scheduledAt", "code": "MUST_BE_FUTURE", "message": "Scheduled date must be in the future" }
  ]
}
```

```java
// Spring Boot with @Valid
@PostMapping("/transactions")
public ResponseEntity<Transaction> create(@Valid @RequestBody TransactionRequest request) {
    // @Valid triggers Bean Validation, Spring's MethodArgumentNotValidException handler
    // catches and returns all field errors at once
}

@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ProblemDetail handleValidation(MethodArgumentNotValidException ex) {
        ProblemDetail pd = ProblemDetail.forStatus(400);
        pd.setTitle("Validation Failed");
        pd.setProperty("errors", ex.getBindingResult().getFieldErrors().stream()
            .map(fe -> Map.of(
                "field", fe.getField(),
                "code", fe.getCode(),
                "message", fe.getDefaultMessage()))
            .collect(Collectors.toList()));
        return pd;
    }
}
```

---

## 8. GraphQL

### The N+1 Problem and DataLoader

Without DataLoader, fetching a list of users and their orders creates N+1 queries:

```graphql
query {
  users {         # 1 query: SELECT * FROM users
    orders {      # N queries: SELECT * FROM orders WHERE userId=? (for each user!)
      total
    }
  }
}
```

**DataLoader** (Facebook's batching pattern): collect all IDs, batch into one query.

```java
// Without DataLoader: N+1 queries
public List<Order> orders(User user) {
    return orderRepository.findByUserId(user.getId()); // called once per user!
}

// With DataLoader: 1 batch query
@SchemaMapping(typeName = "User", field = "orders")
public CompletableFuture<List<Order>> orders(User user, DataLoader<String, List<Order>> loader) {
    return loader.load(user.getId()); // queued — batched after all User resolvers finish
}

// DataLoader batching function — called once with all accumulated IDs
@Bean
public DataLoaderRegistrar ordersLoader(OrderRepository repo) {
    return (registry, env) -> registry.register("ordersLoader",
        DataLoader.newDataLoader((List<String> userIds) ->
            CompletableFuture.supplyAsync(() ->
                repo.findByUserIdIn(userIds).stream() // one query for all
                    .collect(groupingBy(Order::getUserId))
                    .entrySet().stream()
                    .map(e -> userIds.stream().map(id -> e.getKey().equals(id) ? e.getValue() : List.of())
                        .collect(Collectors.toList()))
            )
        )
    );
}
```

### GraphQL Query Complexity

Without limits, a malicious query can crash your server:

```graphql
# Exponential complexity — deeply nested query
{
  users {
    friends {
      friends {
        friends {
          friends { id }  # 4 levels → potentially billions of DB calls
        }
      }
    }
  }
}
```

**Query complexity limiting**:
```java
// Assign cost to each field, reject if total exceeds threshold
@Bean
public Instrumentation complexityInstrumentation() {
    return new MaxQueryComplexityInstrumentation(100); // max complexity score
}

// Or limit depth
@Bean
public Instrumentation depthInstrumentation() {
    return new MaxQueryDepthInstrumentation(10); // max nesting depth
}
```

### GraphQL Caching

GraphQL is hard to cache because everything goes to `POST /graphql`. Approaches:

1. **Persisted queries**: client sends hash of known query; server executes pre-registered query. Hash-keyed CDN cache.
2. **GET for queries**: use GET with query as URL param (cacheable), POST only for mutations.
3. **Response cache on normalized client (Apollo)**: cache individual entities by ID, merge across queries.

---

## 9. gRPC and Protocol Buffers

### Protocol Buffer Design

```protobuf
syntax = "proto3";

package com.niyo.sme.transaction.v1;

service TransactionService {
  rpc CreateTransaction (CreateTransactionRequest) returns (Transaction);
  rpc GetTransaction (GetTransactionRequest) returns (Transaction);
  rpc StreamTransactions (StreamTransactionsRequest) returns (stream Transaction);
  rpc BatchGetTransactions (BatchGetTransactionsRequest) returns (BatchGetTransactionsResponse);
}

message CreateTransactionRequest {
  string corporate_id = 1;
  Money amount = 2;
  BeneficiaryDetails beneficiary = 3;
  string idempotency_key = 4;                 // always include
  string reference = 5;
  google.protobuf.Timestamp scheduled_at = 6; // use well-known types
}

message Money {
  int64 minor_units = 1;   // amount in paise (avoid float)
  string currency_code = 2; // INR, USD
}

message Transaction {
  string transaction_id = 1;
  TransactionStatus status = 2;
  Money amount = 3;
  google.protobuf.Timestamp created_at = 4;
  string corporate_id = 5;
}

enum TransactionStatus {
  TRANSACTION_STATUS_UNSPECIFIED = 0; // proto3: always have 0 default
  TRANSACTION_STATUS_INITIATED = 1;
  TRANSACTION_STATUS_PENDING = 2;
  TRANSACTION_STATUS_SUCCESS = 3;
  TRANSACTION_STATUS_FAILED = 4;
}
```

**Proto design rules**:
- Never change field numbers — they're the wire format
- Never reuse field numbers of removed fields (use reserved)
- Add `UNSPECIFIED = 0` for all enums
- Use well-known types: `google.protobuf.Timestamp`, `google.protobuf.Duration`
- Add idempotency keys to all mutating RPCs

```protobuf
// Mark removed fields as reserved
message Transaction {
  reserved 3, 7;            // field numbers removed
  reserved "old_field_name"; // field name reserved
}
```

### gRPC vs REST Performance

```
Payload size:
  JSON: { "transactionId": "abc123", "amount": 50000, "status": "PENDING" }
  → ~70 bytes
  
  Proto binary: field numbers + varint encoding
  → ~20 bytes (2–10x smaller)

Serialization speed:
  JSON: ~100ns/field (parse + serialize)
  Proto: ~10ns/field (binary decode)

For 10,000 transactions/sec: 200ms/req → 2,000 requests in-flight
  JSON: 10,000 × 70 bytes = 700 KB/s outbound
  Proto: 10,000 × 20 bytes = 200 KB/s — 3.5x bandwidth saving
```

---

## 10. Webhooks and Async APIs

### Webhook Design

Webhooks = your server calls the client's server when an event occurs.

```json
// Webhook payload — design principles
{
  "eventId": "evt_01HN2K3P5QR8T6M7N9B4C6D7E2",  // unique, idempotent
  "eventType": "transaction.completed",             // namespaced dot notation
  "apiVersion": "2024-01-15",                       // version of payload schema
  "timestamp": "2024-01-15T14:30:00Z",
  "data": {
    "transactionId": "txn-abc-123",
    "status": "SUCCESS",
    "amount": { "value": 50000, "currency": "INR" }
  },
  "metadata": {
    "corporateId": "corp-456"
  }
}
```

**Webhook security — verify sender**:
```java
// Sign payload with HMAC-SHA256 using shared secret
// Client verifies signature — proves payload came from you, not an attacker

// You send:
String signature = HMAC_SHA256(payload_body, webhook_secret);
// Header: X-Niyo-Signature: sha256=abc123def456...

// Client verifies:
String expectedSig = HMAC_SHA256(receivedBody, sharedSecret);
boolean valid = MessageDigest.isEqual(
    expectedSig.getBytes(), receivedSignature.getBytes()
);
// Use isEqual (constant time) to prevent timing attacks
```

**Webhook delivery**:
```
1. Event occurs
2. Save event to DB (outbox table) — durable
3. Worker reads outbox → sends HTTP POST to client endpoint
4. On success (2xx): mark delivered
5. On failure: retry with exponential backoff
   - Attempt 1: immediate
   - Attempt 2: 1 min
   - Attempt 3: 5 min
   - Attempt 4: 30 min
   - Attempt 5: 2 hours
   → After max retries: dead letter, alert ops
6. Client MUST respond within 5 seconds (don't do long processing in webhook handler)
```

**Transactional Outbox Pattern** — guarantee webhook delivery:
```
Instead of:                        Do:
DB.save(transaction)               BEGIN TRANSACTION;
Kafka.publish(event)               DB.save(transaction);
                                   DB.insert(outbox_events, event);
                                   COMMIT;
                                   
                                   // Separate process:
                                   // poll outbox → publish to Kafka → delete from outbox
// Problem: if process crashes      // Atomic: transaction + event saved together
// between DB and Kafka → lost event // Outbox ensures delivery even if Kafka is down
```

### Long-Running Operations — Async API Pattern

For operations that take > 5 seconds (KYC processing, report generation, bulk payments):

```
POST /bulk-payments/initiate
→ 202 Accepted
{
  "jobId": "job-abc-123",
  "status": "QUEUED",
  "checkStatusUrl": "/bulk-payments/jobs/job-abc-123",
  "estimatedDuration": "PT30S"  // ISO 8601 duration: 30 seconds
}

GET /bulk-payments/jobs/job-abc-123
→ 200 OK
{
  "jobId": "job-abc-123",
  "status": "PROCESSING",  // QUEUED → PROCESSING → COMPLETED / FAILED
  "progress": { "processed": 450, "total": 1000, "failed": 5 },
  "startedAt": "2024-01-15T14:30:00Z"
}

GET /bulk-payments/jobs/job-abc-123  (after completion)
→ 200 OK
{
  "jobId": "job-abc-123",
  "status": "COMPLETED",
  "result": {
    "downloadUrl": "https://s3.../report.csv?signed=...",
    "successCount": 990,
    "failureCount": 10
  },
  "completedAt": "2024-01-15T14:30:35Z"
}
```

Add webhook notification on completion so clients don't need to poll.

---

## 11. Latency Optimization

### Understanding Latency Sources

Measure each component before optimizing:
```
Total request latency = 
  Network (client → LB)       ~5ms
  + LB processing              ~1ms
  + Network (LB → service)    ~1ms
  + Service compute            ~5ms
  + DB query 1                ~10ms
  + DB query 2                ~8ms
  + External API call         ~80ms  ← biggest bottleneck
  + Serialization              ~2ms
  + Network (service → LB)    ~1ms
  + Network (LB → client)     ~5ms
  = ~118ms total
```

Always find the **actual bottleneck**. Optimizing serialization (2ms) when external API takes 80ms is wasted effort.

### Connection Pooling — Biggest Single Win

Every database/HTTP connection is expensive. Pool and reuse them.

**Database connection pool (HikariCP)**:
```yaml
spring:
  datasource:
    hikari:
      maximum-pool-size: 20       # max connections to DB
      minimum-idle: 5             # idle connections to maintain
      connection-timeout: 3000    # ms to wait for connection from pool (not DB timeout!)
      idle-timeout: 600000        # ms before idle connection is closed
      max-lifetime: 1800000       # ms max connection lifetime (prevents stale)
      leak-detection-threshold: 5000  # alert if connection held > 5s (find leaks)
```

**Pool sizing formula** (PostgreSQL/HikariCP rule of thumb):
```
pool size = number of CPU cores × 2 + number of disk spindles
For c5.xlarge (4 vCPU, SSD): 4 × 2 + 1 = 9 → use 10
For high-latency queries (external): larger pool is fine

Too large: DB runs out of connections (PostgreSQL default max_connections = 100)
Too small: requests queue waiting for pool connection
```

**HTTP client connection pool (OkHttp/Retrofit)**:
```java
// Don't create a new OkHttpClient per request — massive overhead
// Create once, share everywhere (same client = shared pool)
OkHttpClient client = new OkHttpClient.Builder()
    .connectionPool(new ConnectionPool(
        20,          // max idle connections
        5,           // keep-alive duration
        TimeUnit.MINUTES
    ))
    .connectTimeout(5, TimeUnit.SECONDS)
    .readTimeout(30, TimeUnit.SECONDS)
    .build();
```

### Parallel API Calls

If you need data from multiple sources, fetch them concurrently:

```java
// Sequential: 80ms + 60ms + 40ms = 180ms total
CorporateDetails corp = corporateService.get(corpId);         // 80ms
EmployeeList employees = employeeService.list(corpId);        // 60ms
PayrollSummary payroll = payrollService.getSummary(corpId);   // 40ms

// Parallel with CompletableFuture: max(80, 60, 40) = 80ms total
CompletableFuture<CorporateDetails> corpFuture = 
    CompletableFuture.supplyAsync(() -> corporateService.get(corpId), executor);
CompletableFuture<EmployeeList> empFuture = 
    CompletableFuture.supplyAsync(() -> employeeService.list(corpId), executor);
CompletableFuture<PayrollSummary> payrollFuture = 
    CompletableFuture.supplyAsync(() -> payrollService.getSummary(corpId), executor);

CompletableFuture.allOf(corpFuture, empFuture, payrollFuture).join();
CorporateDetails corp = corpFuture.get();
// etc.

// Java 21 Virtual Threads make this even simpler:
try (var executor = Executors.newVirtualThreadPerTaskExecutor()) {
    Future<CorporateDetails> corp = executor.submit(() -> corporateService.get(corpId));
    Future<EmployeeList> emp = executor.submit(() -> employeeService.list(corpId));
    Future<PayrollSummary> payroll = executor.submit(() -> payrollService.getSummary(corpId));
    // all run concurrently
}
```

### Async Non-Blocking Operations

Move non-essential work off the request thread:

```java
// Synchronous: user waits for audit log write
public TransactionResponse createTransaction(TransactionRequest req) {
    Transaction txn = transactionService.create(req);
    auditLogger.log(txn);          // user waits for this
    analyticsService.track(txn);   // and this
    notificationService.send(txn); // and this
    return TransactionResponse.from(txn); // total: 50ms + 20ms + 30ms = 100ms
}

// Async: user gets response immediately, background work happens after
public TransactionResponse createTransaction(TransactionRequest req) {
    Transaction txn = transactionService.create(req);
    eventBus.publishAsync(new TransactionCreatedEvent(txn)); // fire and forget
    return TransactionResponse.from(txn); // 50ms only
}

// @Async in Spring
@Async
public void sendNotification(Transaction txn) { ... } // runs in separate thread pool
```

### Defer Expensive Operations

```
Eager:  compute everything → respond
Lazy:   respond with minimal data → compute expensive things only when needed

Examples:
- Return list of transaction IDs → compute aggregates only when detail requested
- Return thumbnail URL → generate full report only on explicit download request
- Return status → compute analytics only when dashboard opened
```

### Timeout Strategy

Every outbound call needs a timeout. No exceptions.

```java
// OkHttp timeouts
OkHttpClient client = new OkHttpClient.Builder()
    .connectTimeout(3, TimeUnit.SECONDS)   // TCP + TLS handshake
    .readTimeout(10, TimeUnit.SECONDS)     // time to receive response body
    .writeTimeout(5, TimeUnit.SECONDS)     // time to send request body
    .callTimeout(15, TimeUnit.SECONDS)     // overall call timeout (includes retries)
    .build();

// Different timeouts for different SLAs:
// Critical path (payment initiation): connectTimeout=2s, readTimeout=5s
// Non-critical (analytics): connectTimeout=5s, readTimeout=30s
// Background jobs: connectTimeout=10s, readTimeout=60s
```

**Cascading timeouts**: If service A calls B calls C, each timeout must be smaller than the caller's:
```
Client: 30s timeout
  Service A: 25s timeout budget
    Service B: 20s timeout budget
      Service C: 15s timeout budget

Never let inner service have longer timeout than outer.
A 60s DB timeout inside a 10s HTTP timeout is useless.
```

### HTTP/2 and HTTP/3

**Switch to HTTP/2** for significant latency improvements (especially on mobile):
```yaml
# Spring Boot with embedded Tomcat
server:
  http2:
    enabled: true
  ssl:
    enabled: true  # HTTP/2 requires HTTPS in practice
```

Benefits over HTTP/1.1:
- **Multiplexing**: 100 requests over 1 TCP connection (vs 1 per connection or limited pipelining)
- **Header compression (HPACK)**: repeated headers (Authorization, Accept) compressed across requests
- **Server push**: send CSS/JS before browser asks (for web APIs serving SPAs)

**HTTP/3 (QUIC)**: eliminates TCP head-of-line blocking. Critical for mobile (high packet loss, IP changes). Available in Nginx 1.25+, Cloudflare, AWS CloudFront.

---

## 12. Payload Optimization and Compression

### Gzip, Brotli, Zstd — Response Compression

Compress response body. Text-based formats (JSON, XML) compress 5–10x.

```
100KB JSON response:
  Uncompressed: 100 KB
  gzip:         ~10–20 KB  (80–90% reduction)
  brotli:       ~8–15 KB   (slightly better than gzip)
  zstd:         ~8–12 KB   (fast + good ratio)

On 100Mbps connection: 100KB = ~8ms, 15KB = ~1.2ms
On mobile (10Mbps): 100KB = 80ms, 15KB = 12ms — massive difference
```

**Enable in Spring Boot**:
```yaml
server:
  compression:
    enabled: true
    min-response-size: 1024      # only compress > 1KB
    mime-types:
      - application/json
      - application/xml
      - text/html
      - text/plain
```

```java
// Client must advertise what it accepts
// Accept-Encoding: gzip, br, zstd

// Server responds with:
// Content-Encoding: gzip
// (server auto-decompressed by client)

// For OkHttp (automatic):
OkHttpClient client = new OkHttpClient.Builder()
    .addInterceptor(new GzipRequestInterceptor()) // compress requests too
    .build();
// OkHttp automatically adds Accept-Encoding: gzip and decompresses responses
```

**Compress requests too** (for large payloads like bulk operations):
```
POST /bulk-transactions
Content-Encoding: gzip
Content-Type: application/json
[body: gzip-compressed JSON]

Benefit: Sending 10MB payload → ~1–2MB after compression
```

### Field Selection (Sparse Fieldsets)

Don't return what the client didn't ask for:

```
GET /transactions?fields=id,amount,status

Response: only those fields
{
  "data": [
    { "id": "txn-123", "amount": 50000, "status": "SUCCESS" }
    // no corporateId, beneficiary, createdAt, metadata, etc.
  ]
}
```

**Implementation**:
```java
@GetMapping("/transactions")
public List<Map<String, Object>> getTransactions(
    @RequestParam(required = false) Set<String> fields) {
    
    List<Transaction> transactions = transactionService.findAll();
    
    if (fields == null || fields.isEmpty()) {
        return transactions.stream().map(TransactionDTO::from).collect(toList());
    }
    
    // Return only requested fields
    return transactions.stream()
        .map(t -> filterFields(TransactionDTO.from(t), fields))
        .collect(toList());
}

private Map<String, Object> filterFields(TransactionDTO dto, Set<String> fields) {
    ObjectMapper mapper = new ObjectMapper();
    Map<String, Object> all = mapper.convertValue(dto, Map.class);
    all.keySet().retainAll(fields);
    return all;
}
```

### Avoid Over-Fetching Anti-Patterns

```
# Anti-pattern: returning full object when only status is needed
GET /transactions/txn-123
→ returns 50 fields (full transaction, beneficiary details, audit trail, etc.)

# Better: provide lightweight views
GET /transactions/txn-123?view=summary   → 5 fields
GET /transactions/txn-123?view=full      → all fields
GET /transactions/txn-123/status         → just the status field

# Or proper field selection
GET /transactions/txn-123?fields=id,status,amount
```

### Streaming for Large Responses

For large datasets, stream instead of buffering in memory:

```java
// BAD: loads all into memory, then serializes
@GetMapping("/reports/transactions")
public List<TransactionDTO> getAllTransactions() {
    return transactionRepository.findAll(); // 100,000 records in RAM
}

// GOOD: stream directly to response
@GetMapping("/reports/transactions")
public void streamTransactions(HttpServletResponse response) throws IOException {
    response.setContentType("application/x-ndjson"); // newline-delimited JSON
    response.setHeader("Transfer-Encoding", "chunked");
    
    try (PrintWriter writer = response.getWriter()) {
        transactionRepository.streamAll().forEach(txn -> {
            writer.println(objectMapper.writeValueAsString(TransactionDTO.from(txn)));
            writer.flush(); // flush each line
        });
    }
}

// Spring WebFlux (reactive) — true streaming
@GetMapping(value = "/transactions/stream", produces = MediaType.APPLICATION_NDJSON_VALUE)
public Flux<TransactionDTO> streamTransactions() {
    return transactionRepository.findAll()  // Reactive repository
        .map(TransactionDTO::from);
}
```

### Binary Formats

For internal service-to-service communication where both sides control the schema:

| Format | Size vs JSON | Speed | Human-readable | Schema required |
|---|---|---|---|---|
| JSON | 1x | Slow | Yes | No |
| MessagePack | ~0.5x | Fast | No | No |
| Protocol Buffers | ~0.3x | Very fast | No | Yes (.proto) |
| Avro | ~0.3x | Very fast | No | Yes |
| FlatBuffers | ~0.4x | Fastest | No | Yes |

For SDE2 API work: JSON for external, Protocol Buffers for internal gRPC.

### Pagination as Payload Optimization

Forcing pagination is a payload optimization, not just UX:

```
Without pagination: 
  GET /transactions → returns 100,000 records → 50MB response → OOM → client crashes

With pagination:
  GET /transactions?limit=100 → 100 records → 50KB → instant

Always default to a reasonable page size and cap max:
  default = 20, max = 100 (enforced server-side)
```

---

## 13. HTTP Caching

HTTP caching is the most underused latency optimization. Done right, it eliminates server calls entirely.

### Cache-Control Directives

```
Cache-Control: public, max-age=3600
  public     = can be cached by browser AND CDN/proxy
  private    = browser-only cache (user-specific data)
  no-cache   = can be stored but MUST revalidate with server before use
  no-store   = do NOT store any cache copy
  max-age=N  = cache for N seconds
  s-maxage=N = CDN/proxy TTL (overrides max-age for shared caches)
  must-revalidate  = once expired, must revalidate before serving stale
  stale-while-revalidate=N = serve stale for N seconds while fetching fresh in background
  immutable  = content will never change (safe to cache forever)
```

**Real-world examples**:
```
Static assets (versioned):
  Cache-Control: public, max-age=31536000, immutable
  → Cache forever (filename includes hash, changes on deploy)

Public API data (product catalog, IFSC details):
  Cache-Control: public, max-age=300, stale-while-revalidate=60
  → Cache 5 min, serve stale for 1 min while fetching fresh (zero latency on refresh)

Authenticated user data (account balance):
  Cache-Control: private, max-age=30
  → Cache in browser for 30s, never in CDN

Sensitive data (OTPs, tokens):
  Cache-Control: no-store
  → Never cached anywhere

Frequently changing public data (stock prices, live scores):
  Cache-Control: public, max-age=0, must-revalidate
  → Always revalidate, but CDN can serve while revalidating
```

### ETag and Conditional Requests

ETag = fingerprint of the response. Client caches it, sends it back next time. Server only transmits body if content changed.

```
Initial request:
Client → GET /corporates/corp-123
Server → 200 OK
         ETag: "abc123def456"
         Cache-Control: private, max-age=300
         [full response body]

Second request (after max-age expires):
Client → GET /corporates/corp-123
         If-None-Match: "abc123def456"
         
Server: check if corp-123 has changed since ETag "abc123def456" was generated
  Not changed → 304 Not Modified (no body!) → client uses cached copy
  Changed     → 200 OK, new ETag: "xyz789" [full body]
```

**Benefit**: 304 response has no body. 0 bytes of content transmitted. Just headers (~200 bytes).

**Implementation in Spring**:
```java
@GetMapping("/corporates/{id}")
public ResponseEntity<CorporateDTO> getCorporate(
    @PathVariable String id,
    @RequestHeader(value = "If-None-Match", required = false) String ifNoneMatch) {
    
    Corporate corp = corporateService.findById(id);
    String etag = '"' + DigestUtils.md5Hex(corp.getUpdatedAt().toString()) + '"';
    
    if (etag.equals(ifNoneMatch)) {
        return ResponseEntity.status(304).build(); // Not Modified
    }
    
    return ResponseEntity.ok()
        .eTag(etag)
        .cacheControl(CacheControl.maxAge(300, TimeUnit.SECONDS).cachePrivate())
        .body(CorporateDTO.from(corp));
}

// Or use Spring's built-in ETag filter:
@Bean
public FilterRegistrationBean<ShallowEtagHeaderFilter> etagFilter() {
    FilterRegistrationBean<ShallowEtagHeaderFilter> bean = new FilterRegistrationBean<>();
    bean.setFilter(new ShallowEtagHeaderFilter()); // automatically computes ETag from response body
    return bean;
}
```

### Last-Modified

Alternative to ETag using timestamps:

```
Server → Last-Modified: Wed, 15 Jan 2024 14:30:00 GMT
Client → If-Modified-Since: Wed, 15 Jan 2024 14:30:00 GMT
Server → 304 Not Modified (if unchanged)
```

ETag is preferred (content-based, not time-based — time zone issues, same-second updates).

### Vary Header — Cache per Header

```
Vary: Accept-Encoding

Tells CDN: cache separately for each value of Accept-Encoding.
gzip clients get gzip version; br clients get brotli version.

Vary: Accept-Language
  Cache separately per language (English vs Hindi response)

Vary: Authorization
  Don't use! Every user gets their own cache entry → cache useless for authenticated endpoints
  Instead: don't cache authenticated responses, or use response body hash as key
```

### CDN + API Gateway Caching

```
Layer 1: Browser cache (Cache-Control: max-age=N)
Layer 2: CDN (CloudFront, Cloudflare) — caches at edge
Layer 3: API Gateway cache (Kong, AWS API Gateway)
Layer 4: Application cache (Redis)
Layer 5: Database query cache

Hit Layer 1: 0ms (instant, no network)
Hit Layer 2: 5ms (edge PoP near user)
Hit Layer 3: 20ms (regional)
Hit Layer 4: 50ms (application + Redis)
Hit Layer 5: 100ms (full roundtrip)
Miss all:    100ms+ (DB query)
```

What to cache at CDN vs Redis:
```
CDN: public, static-ish data (IFSC details, bank lists, product catalog, docs)
Redis: authenticated, user-specific, frequently accessed (session, user profile, account summary)
Application local (Caffeine): ultra-hot data per instance (config, feature flags)
```

---

## 14. Query Optimization for APIs

### The N+1 Problem

The single biggest performance killer in APIs backed by ORMs or manual queries.

```java
// N+1: 1 query for list + N queries for each item's related data
List<Corporate> corporates = corporateRepo.findAll();     // 1 query
for (Corporate corp : corporates) {
    List<Employee> emps = employeeRepo.findByCorporateId(corp.getId()); // N queries!
    corp.setEmployees(emps);
}
// For 100 corporates: 1 + 100 = 101 queries

// Solution 1: JOIN in one query
List<Corporate> corporates = corporateRepo.findAllWithEmployees(); // 1 query

// Solution 2: Batch loading
List<String> corpIds = corporates.stream().map(Corporate::getId).collect(toList());
Map<String, List<Employee>> empsByCorpId = employeeRepo.findByCorpIdIn(corpIds)
    .stream().collect(groupingBy(Employee::getCorporateId)); // 1 query
corporates.forEach(c -> c.setEmployees(empsByCorpId.getOrDefault(c.getId(), List.of())));
// Total: 2 queries regardless of N

// MongoDB equivalent
@Query(value = "{ 'status': ?0 }", fields = "{ 'name': 1, 'email': 1, 'status': 1 }")
List<Corporate> findByStatus(String status); // projection — don't fetch all fields
```

### Database Index Design for API Queries

Design indexes around your API's access patterns, not your schema:

```java
// API endpoint: GET /transactions?corporateId=X&status=Y&createdAt[after]=Z&limit=20

// Query:
// SELECT * FROM transactions
// WHERE corporate_id = 'corp-123'
// AND status = 'PENDING'
// AND created_at > '2024-01-01'
// ORDER BY created_at DESC
// LIMIT 20;

// Optimal index: covers filter + sort
// (corporate_id, status, created_at) — composite index in this order!
// Rule: equality filters first, range filter last, sort column last

// MongoDB:
db.transactions.createIndex(
    { corporateId: 1, status: 1, createdAt: -1 }
);

// PostgreSQL:
CREATE INDEX idx_transactions_corp_status_date
ON transactions(corporate_id, status, created_at DESC)
WHERE status != 'DELETED'; -- partial index: exclude deleted (smaller index)
```

**Explain queries and look at**:
```sql
EXPLAIN ANALYZE SELECT ...;
-- Look for: Seq Scan (bad for large tables), Index Scan (good), rows estimated vs actual

-- MongoDB
db.transactions.find({...}).explain("executionStats")
-- Look for: COLLSCAN (bad), IXSCAN (good), totalDocsExamined vs nReturned
```

### Projection — Select Only What You Need

```java
// Bad: fetch entire document, use 3 fields
List<Transaction> txns = transactionRepo.findAll();
return txns.stream().map(t -> new TxnSummary(t.getId(), t.getStatus(), t.getAmount()));

// Good: MongoDB projection — only fetch needed fields
@Query(value = "{ 'corporateId': ?0 }", fields = "{ '_id': 1, 'status': 1, 'amount': 1 }")
List<TransactionSummary> findSummariesByCorporateId(String corpId);

// JPA JPQL projection
@Query("SELECT new com.niyo.sme.dto.TxnSummary(t.id, t.status, t.amount) FROM Transaction t WHERE t.corporateId = :corpId")
List<TxnSummary> findSummaries(@Param("corpId") String corpId);

// Or JPA interface projection (type-safe)
public interface TxnSummary {
    String getId();
    String getStatus();
    BigDecimal getAmount();
}
List<TxnSummary> summaries = txnRepo.findByCorporateId(corpId, TxnSummary.class);
```

### Avoid COUNT for Pagination

`COUNT(*)` on large tables is slow. Alternatives:

```
1. Estimated count: use DB statistics (PostgreSQL reltuples)
   SELECT reltuples FROM pg_class WHERE relname = 'transactions';
   → fast, approximate (within 5–10%)

2. Cursor pagination: don't return total at all — just hasNext
   "hasNext": true  (fetch N+1, if got N+1, there's more)

3. Cached count: store count in Redis, update on writes
   SET transactions:count:corp123 5420 EX 300
   INCR transactions:count:corp123  -- on insert
   DECR transactions:count:corp123  -- on delete

4. Lazy count: only compute when explicitly requested
   GET /transactions?countTotal=true  -- default false
```

### Denormalization for Read Performance

For read-heavy APIs, store data in the shape you read it, not the shape you write it:

```
Write model (normalized):
  corporate: { id, name, gstin, status }
  account: { id, corpId, balance, ifsc }
  employees: [ { id, corpId, name, salary } ]

Read model (denormalized) — for dashboard API:
  corporateDashboard: {
    corporateId,
    corporateName,
    status,
    totalBalance,       ← pre-computed
    employeeCount,      ← pre-computed
    lastTransactionAt,  ← pre-computed
    accountDetails: { ... }
  }

Build read model via:
- Events: on corporate.created, employee.added → update dashboard document
- Scheduled job: rebuild every 5 min
- Change Data Capture: MongoDB change streams → rebuild affected document
```

### Read Replicas

```
Write traffic → Primary (writes + consistency-critical reads)
Read traffic  → Read replicas (bulk of API reads — eventually consistent)

Spring Data MongoDB:
  @ReadPreference: PRIMARY      // route to primary
  @ReadPreference: SECONDARY    // route to replica

Spring Data JPA with read replica:
  @Transactional(readOnly = true) // hint for replica routing
  public List<Transaction> findAll() { ... }
```

**When to use primary for reads**:
- Read-your-writes: after a write, you immediately read and must see your write
- Decisions based on latest data (balance check before debit)

**When replica is fine**:
- Dashboards and reports (eventual consistency is OK)
- Lists and searches (slightly stale is OK)
- Analytics (stale by design)

---

## 15. Connection Management

### Keep-Alive Tuning

```nginx
# Nginx → upstream (your service)
upstream backend {
    server app:8080;
    keepalive 64;  # max idle keepalive connections to upstream
}

server {
    location /api/ {
        proxy_http_version 1.1;
        proxy_set_header Connection "";  # remove Connection: close from upstream requests
        proxy_pass http://backend;
    }
    
    # Client → Nginx
    keepalive_timeout 65s;
    keepalive_requests 1000;
}
```

### Connection Pool Sizing Rules

```
Too small: requests queue, high wait time → high latency
Too large: DB overwhelmed with connections → connection thrashing → high latency

DB max_connections (PostgreSQL default): 100
Reserve some for admin/monitoring: 10
Available for app: 90

If 3 instances of your service:
  Per-instance pool max: 90 / 3 = 30

Formula: pool_size ≈ cores × 2, bounded by max_connections / instances
```

### Connection Timeouts vs Read Timeouts

```
connectTimeout: time to establish TCP + TLS connection
  Set to: 1–3 seconds (should be fast; if it takes longer, something is wrong)

readTimeout: time to wait for first byte after sending request
  Set to: 5–30 seconds (depends on your SLA and what the downstream does)

writeTimeout: time to send the request body
  Set to: 5 seconds (usually fast unless uploading large files)

callTimeout: total time for the entire call including retries
  Set to: slightly larger than your API's SLA budget

⚠️ Don't set timeouts too long — a 30s read timeout means requests can pile up for 30s
   during an outage before failing. With 100 concurrent users × 30s = 3000 stuck threads
```

---

## 16. Idempotency and Retries

### Why Idempotency Matters

Networks are unreliable. Clients retry. Without idempotency, retries cause duplicate operations (double payments, double emails).

```
Timeline:
t=0:    Client sends POST /payments (₹1000 to Acme)
t=5:    Network timeout — client doesn't know if server processed it
t=5:    Client retries POST /payments (₹1000 to Acme)

Without idempotency: ₹2000 debited
With idempotency key: server detects duplicate → returns same 201 Created → ₹1000 debited
```

### Implementing Idempotency

```java
@PostMapping("/payments")
public ResponseEntity<PaymentResponse> createPayment(
    @RequestHeader("Idempotency-Key") String idempotencyKey,
    @Valid @RequestBody PaymentRequest request) {
    
    // 1. Check if we've seen this key before
    Optional<PaymentResponse> cached = idempotencyStore.get(idempotencyKey);
    if (cached.isPresent()) {
        return ResponseEntity.status(200)  // or 201 — same as original
            .header("Idempotent-Replayed", "true")
            .body(cached.get());
    }
    
    // 2. Acquire lock on idempotency key (prevent concurrent duplicate)
    boolean locked = redisLock.acquire("idempotency:" + idempotencyKey, 30);
    if (!locked) {
        return ResponseEntity.status(409).build(); // concurrent request with same key
    }
    
    try {
        // 3. Double-check after acquiring lock
        cached = idempotencyStore.get(idempotencyKey);
        if (cached.isPresent()) return ResponseEntity.ok(cached.get());
        
        // 4. Process
        PaymentResponse response = paymentService.process(request);
        
        // 5. Store response
        idempotencyStore.set(idempotencyKey, response, Duration.ofHours(24));
        
        return ResponseEntity.status(201).body(response);
    } finally {
        redisLock.release("idempotency:" + idempotencyKey);
    }
}
```

**Idempotency key storage**:
```
Redis HASH: { key → serialized response, timestamp }
TTL: 24 hours (match payment window)
If Redis is down: let request through without idempotency (fail open vs fail closed — business decision)
```

### Retry Strategy

**On client side**:
```java
// Exponential backoff with jitter
int attempt = 0;
int maxAttempts = 3;
long baseDelayMs = 100;

while (attempt < maxAttempts) {
    try {
        return apiClient.call(request);
    } catch (RetryableException e) {
        attempt++;
        if (attempt == maxAttempts) throw e;
        long delay = (long) (baseDelayMs * Math.pow(2, attempt));  // 200ms, 400ms
        long jitter = ThreadLocalRandom.current().nextLong(0, delay / 2); // ±50%
        Thread.sleep(delay + jitter);  // 200±100ms, 400±200ms
    }
}
```

**What to retry**:
```
RETRY:
  Connection timeout (didn't reach server)
  Read timeout (request may not have processed — only if IDEMPOTENT)
  503 Service Unavailable
  502 Bad Gateway
  429 Too Many Requests (after Retry-After delay)

NEVER RETRY:
  400 Bad Request (same request will fail again)
  401 Unauthorized (fix auth first)
  403 Forbidden
  404 Not Found
  409 Conflict
  422 Unprocessable Entity
  500 Internal Server Error (may be non-idempotent — dangerous to retry POST)
```

---

## 17. Rate Limiting and Throttling

### Rate Limit Headers

Always tell clients their limit state:

```
X-RateLimit-Limit: 100          # max requests in window
X-RateLimit-Remaining: 42       # remaining in current window
X-RateLimit-Reset: 1704106800   # epoch when window resets
X-RateLimit-Window: 60          # window size in seconds
Retry-After: 18                 # seconds to wait (on 429)
```

### Rate Limit Dimensions

```
Per IP:       prevent DDoS, scraping (unauthenticated endpoints)
Per API key:  partner quotas (Finbox: 1000/hr, FreshDesk: 100/min)
Per user:     fair use (user: 60 writes/min, 600 reads/min)
Per endpoint: protect expensive ops (POST /bulk-payments: 10/min vs GET: 1000/min)
Per tenant:   multi-tenant SaaS (corporate: 10,000 API calls/day based on plan)
```

### Graceful Degradation Under Load

```
When load approaches capacity:
  1. Return 429 with Retry-After for excess requests (don't accept what you can't serve)
  2. Prioritize critical paths: payments > reports > analytics
  3. Shed low-priority read traffic before write traffic
  4. Serve stale cached responses before rejecting

Request priority queue:
  CRITICAL:  payment initiation, OTP, authentication
  HIGH:      account balance, transaction status
  NORMAL:    transaction history, reports
  LOW:       analytics, exports, bulk operations
```

---

## 18. API Security

### Authentication Patterns

**API Key** (service-to-service, partner APIs):
```
X-API-Key: ak_live_abc123xyz789

Store: hash of key in DB (never store plaintext)
Validate: hash incoming key, compare to stored hash
Scope: key tied to permissions (read-only, write, admin)
Rotation: support multiple active keys (grace period for rotation)
```

**JWT (user-facing APIs)**:
```
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...

Validate:
1. Signature (RS256: use public key — asymmetric, safer for multi-service)
2. Expiry (exp claim)
3. Issuer (iss claim)
4. Audience (aud claim — which service is this token for)

Short-lived access tokens (15 min) + refresh tokens (7 days)
Refresh tokens stored in DB (allows revocation)
```

**mTLS (mutual TLS)** — for high-security internal service-to-service:
```
Both client and server present certificates.
Server verifies client certificate — strongest auth for internal microservices.
No bearer token needed — identity proven by certificate.
```

### Input Validation — Never Trust Input

```java
// Validate at the boundary — every field from outside
@NotNull(message = "corporateId is required")
@Size(min = 5, max = 50, message = "corporateId must be 5-50 characters")
@Pattern(regexp = "^corp-[a-z0-9-]+$", message = "Invalid corporateId format")
private String corporateId;

@Positive(message = "Amount must be positive")
@DecimalMax(value = "10000000.00", message = "Amount cannot exceed ₹1 crore")
private BigDecimal amount;

@Valid  // cascade validation to nested objects
private BeneficiaryDetails beneficiary;

// Always validate:
// - Type (is this a number where we expect a number?)
// - Range (is amount between 1 and max-transfer-limit?)
// - Format (is IFSC 11 chars? Is PAN 10 chars?)
// - Referential integrity (does corporateId exist and belong to this user?)
```

### OWASP API Security Top 10

| # | Threat | Mitigation |
|---|---|---|
| 1 | **Broken Object Level Authorization** | Check if THIS user can access THIS object, not just if user is logged in |
| 2 | **Broken Authentication** | Strong JWT validation, short expiry, secure token storage |
| 3 | **Broken Object Property Level Auth** | Don't expose admin fields to regular users |
| 4 | **Unrestricted Resource Consumption** | Rate limiting, payload size limits, pagination enforcement |
| 5 | **Broken Function Level Authorization** | Admin endpoints need admin role check — not just authentication |
| 6 | **Unrestricted Access to Sensitive Business Flows** | Bot protection, CAPTCHA on sensitive operations |
| 7 | **Server-Side Request Forgery (SSRF)** | Validate URLs before fetching (allowlist domains) |
| 8 | **Security Misconfiguration** | Remove debug endpoints, don't expose stack traces, secure headers |
| 9 | **Improper Inventory Management** | Track all APIs, document deprecated ones, no shadow APIs |
| 10 | **Unsafe API Consumption** | Validate third-party API responses, don't trust blindly |

**BOLA (Broken Object Level Authorization) — the most common**:
```java
// BAD: endpoint checks if user is authenticated, but not if they OWN this corporate
@GetMapping("/corporates/{id}/transactions")
public List<Transaction> getTransactions(@PathVariable String id) {
    return transactionService.findByCorporateId(id); // any user can access any corporate!
}

// GOOD: verify ownership
@GetMapping("/corporates/{id}/transactions")
public List<Transaction> getTransactions(
    @PathVariable String id,
    @AuthenticationPrincipal User currentUser) {
    
    if (!corporateService.belongsToUser(id, currentUser.getId())) {
        throw new AccessDeniedException("You do not have access to corporate " + id);
    }
    return transactionService.findByCorporateId(id);
}
```

### Security Headers

```java
@Bean
public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http.headers(headers -> headers
        .contentTypeOptions(Customizer.withDefaults())          // X-Content-Type-Options: nosniff
        .frameOptions(FrameOptionsConfig::deny)                 // X-Frame-Options: DENY
        .xssProtection(Customizer.withDefaults())               // X-XSS-Protection: 1; mode=block
        .httpStrictTransportSecurity(hsts ->                    // HSTS: force HTTPS
            hsts.includeSubDomains(true).maxAgeInSeconds(31536000))
        .referrerPolicy(ref ->                                  // Referrer-Policy
            ref.policy(ReferrerPolicyHeaderWriter.ReferrerPolicy.STRICT_ORIGIN))
    );
    return http.build();
}

// Content-Security-Policy for APIs:
// Content-Security-Policy: default-src 'none'; frame-ancestors 'none'
```

### CORS

```java
@Configuration
public class CorsConfig implements WebMvcConfigurer {
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        registry.addMapping("/api/**")
            .allowedOrigins(
                "https://app.niyo.com",
                "https://workbench.niyo.com"
            )
            .allowedMethods("GET", "POST", "PUT", "PATCH", "DELETE", "OPTIONS")
            .allowedHeaders("Authorization", "Content-Type", "X-Request-ID", "Idempotency-Key")
            .exposedHeaders("X-RateLimit-Remaining", "X-Request-ID", "Location")
            .allowCredentials(true)
            .maxAge(3600); // preflight cache: 1 hour (reduces OPTIONS requests)
    }
}
```

---

## 19. API Reliability Patterns

### Circuit Breaker

```java
// Resilience4j configuration
@CircuitBreaker(name = "bankApi", fallbackMethod = "bankApiFallback")
@TimeLimiter(name = "bankApi")  // timeout
@Retry(name = "bankApi")        // retry
public BankResponse initiateTransfer(TransferRequest request) {
    return yblBankClient.initiate(request);
}

public BankResponse bankApiFallback(TransferRequest request, Exception ex) {
    // Log the failure, return a queued/pending response
    auditLog.error("Bank API failed, queuing for retry", ex);
    return BankResponse.queued(request.getTransactionId());
}
```

```yaml
resilience4j:
  circuitbreaker:
    instances:
      bankApi:
        sliding-window-size: 10         # last 10 calls
        failure-rate-threshold: 50      # open if >50% fail
        wait-duration-in-open-state: 30s
        permitted-calls-in-half-open-state: 3
        slow-call-duration-threshold: 5s
        slow-call-rate-threshold: 80    # open if >80% take >5s
  retry:
    instances:
      bankApi:
        max-attempts: 3
        wait-duration: 1s
        retry-exceptions:
          - java.net.SocketTimeoutException
          - java.io.IOException
        ignore-exceptions:
          - com.niyo.sme.exception.BusinessException
```

### Bulkhead — Isolate Resources

```yaml
resilience4j:
  bulkhead:
    instances:
      bankApi:
        max-concurrent-calls: 20      # only 20 threads for bank API
        max-wait-duration: 0ms        # fail immediately if full (don't queue)
      kycApi:
        max-concurrent-calls: 10
```

If bank API is slow and occupying 20 threads, KYC API still has its 10 threads — isolated.

### Health Checks

```java
// Spring Boot Actuator
# GET /actuator/health
{
  "status": "UP",
  "components": {
    "db": { "status": "UP" },
    "kafka": { "status": "UP" },
    "redis": { "status": "UP" },
    "bankApiCircuitBreaker": { "status": "UP", "details": { "state": "CLOSED" } }
  }
}

// Custom health indicator
@Component
public class ExternalApiHealthIndicator implements HealthIndicator {
    @Override
    public Health health() {
        try {
            bankClient.ping();
            return Health.up().withDetail("bankApi", "reachable").build();
        } catch (Exception e) {
            return Health.down().withDetail("bankApi", e.getMessage()).build();
        }
    }
}
```

**Liveness vs Readiness** (Kubernetes):
```
Liveness: Is the process alive? (should we restart it?)
  Fails → Kubernetes kills and restarts the pod
  Only fail this if the process is deadlocked, OOM, or truly broken

Readiness: Is the service ready to accept traffic? (should LB send it requests?)
  Fails → Kubernetes removes pod from Service endpoints (stops traffic)
  Fail this during startup, during deploys, when dependencies are down

# application.yml
management:
  endpoint:
    health:
      probes:
        enabled: true
  endpoints:
    web:
      exposure:
        include: health, info, metrics, prometheus
```

### Graceful Shutdown

```yaml
server:
  shutdown: graceful    # wait for in-flight requests before shutting down
  
spring:
  lifecycle:
    timeout-per-shutdown-phase: 30s  # max time to wait for requests to finish
```

During graceful shutdown:
1. Kubernetes sends SIGTERM
2. Pod stops receiving new requests (removed from Service)
3. In-flight requests finish (up to 30s)
4. Process exits

Without this: requests in progress get connection reset (500 to clients) during deploy.

---

## 20. API Gateway Patterns

### What API Gateway Does

```
Clients → API Gateway → Service A
                     → Service B
                     → Service C

Gateway handles:
  SSL termination       → services only see plain HTTP internally
  Auth/JWT validation   → services trust requests from gateway
  Rate limiting         → centralized, not per-service
  Request routing       → /payments/* → payment-service
  Request transformation→ rename fields, add headers, version translation
  Response aggregation  → combine Service A + B responses into one response
  Logging/tracing       → inject traceId, log all requests
  Circuit breaking      → gateway-level circuit breaker
  API versioning        → route /v1/* → v1 service, /v2/* → v2 service
  IP allowlisting       → block requests not from known IPs
```

### Request Aggregation / BFF Pattern

```
Mobile app needs: user profile + recent transactions + account balance
Without aggregation: 3 API calls from mobile
With aggregation: 1 call to BFF → BFF calls 3 services in parallel → 1 response

Mobile BFF (optimized for mobile screen):
GET /mobile/dashboard
→ {
    user: { name, avatar },
    balance: 12500.00,
    recentTransactions: [ ...last 5... ]  // only 5, not 20
  }

Web BFF (optimized for desktop):
GET /web/dashboard  
→ { 
    user: { name, avatar, email, phone, plan },
    balance: { current: 12500, reserved: 2000, available: 10500 },
    recentTransactions: [ ...last 20 with full details... ],
    charts: { monthlySpend: [...] }
  }
```

### Gateway-Level Caching

```yaml
# Kong API Gateway caching
plugins:
  - name: proxy-cache
    config:
      response_code: [200, 304]
      request_method: [GET, HEAD]
      content_type: [application/json]
      cache_ttl: 300                    # 5 minutes
      strategy: memory                  # or redis for distributed
      cache_control: true               # respect Cache-Control headers
      vary_headers: [Authorization]     # separate cache per auth token (careful — kills CDN efficiency)
```

---

## 21. API Documentation and Contracts

### OpenAPI / Swagger

```java
// Spring Boot with springdoc-openapi
@Operation(
    summary = "Initiate a fund transfer",
    description = "Creates a new NEFT/RTGS/IMPS transaction. Idempotent if same Idempotency-Key is used.",
    tags = { "Transactions" }
)
@ApiResponses({
    @ApiResponse(responseCode = "201", description = "Transaction initiated",
        headers = @Header(name = "Location", description = "URL of created transaction")),
    @ApiResponse(responseCode = "400", description = "Validation failed",
        content = @Content(schema = @Schema(implementation = ProblemDetail.class))),
    @ApiResponse(responseCode = "409", description = "Duplicate idempotency key",
        content = @Content(schema = @Schema(implementation = ProblemDetail.class))),
    @ApiResponse(responseCode = "422", description = "Insufficient balance or invalid IFSC")
})
@PostMapping("/transactions")
public ResponseEntity<TransactionResponse> createTransaction(
    @Parameter(description = "Idempotency key for safe retries", required = true)
    @RequestHeader("Idempotency-Key") String idempotencyKey,
    @Valid @RequestBody CreateTransactionRequest request) { ... }
```

**OpenAPI best practices**:
- Document every error response, not just happy path
- Include real examples in schema definitions
- Mark deprecated fields with `deprecated: true`
- Use `$ref` for reusable schemas (don't repeat MoneyAmount in every endpoint)
- Keep OpenAPI spec in version control alongside code

### API Changelog

Every breaking and non-breaking change documented with date and migration path:
```markdown
## 2024-01-15 — v2.0 (Breaking)
- BREAKING: `amount` field changed from float to object `{ value, currency }`
  Migration: update clients to send `{ "amount": { "value": 50000, "currency": "INR" } }`
- ADDED: `scheduledAt` optional field for future-dated transactions
- DEPRECATED: `v1` endpoints — sunset date: 2025-01-15

## 2024-01-01 — v1.3 (Non-breaking)
- ADDED: `?fields=` query parameter for field selection on all GET endpoints
- ADDED: cursor-based pagination via `?after=<cursor>` (offset pagination still works)
- FIXED: 500 error when `beneficiary.name` contained special characters
```

### Contract Testing

```java
// Consumer-driven contract testing with Pact
// Service B (consumer) defines what it expects from Service A (provider)

// In Service B's tests:
@Pact(consumer = "TransactionService", provider = "CorporateService")
RequestResponsePact getCorporatePact(PactDslWithProvider builder) {
    return builder
        .given("corporate corp-123 exists")
        .uponReceiving("a request for corporate details")
            .path("/corporates/corp-123")
            .method("GET")
        .willRespondWith()
            .status(200)
            .body(new PactDslJsonBody()
                .stringType("id", "corp-123")
                .stringType("name", "Niyo Corp")
                .stringType("status", "ACTIVE"))
        .toPact();
}

// Service A (provider) runs pact verification against this contract
// If Service A changes its response schema → pact verification fails → CI fails → breaking change detected before deployment
```

---

## 22. Observability for APIs

### The Four Golden Signals for APIs

```
1. Latency:     p50, p95, p99 per endpoint
2. Traffic:     requests per second per endpoint
3. Errors:      error rate (4xx, 5xx) per endpoint
4. Saturation:  thread pool utilization, DB connection pool, queue depth
```

### Structured Logging

```java
// Don't do this — unstructured, hard to query
log.info("User " + userId + " created transaction " + txnId + " for amount " + amount);

// Do this — structured JSON log
log.info("Transaction created",
    StructuredArguments.keyValue("userId", userId),
    StructuredArguments.keyValue("transactionId", txnId),
    StructuredArguments.keyValue("amount", amount),
    StructuredArguments.keyValue("corporateId", corporateId),
    StructuredArguments.keyValue("traceId", MDC.get("traceId")));

// Output:
{
  "timestamp": "2024-01-15T14:30:00.123Z",
  "level": "INFO",
  "message": "Transaction created",
  "userId": "user-123",
  "transactionId": "txn-456",
  "amount": 50000,
  "corporateId": "corp-789",
  "traceId": "4bf92f3577b34da6a3ce929d0e0e4736"
}
```

### Request Logging Middleware

```java
@Component
@Order(1)
public class RequestLoggingFilter extends OncePerRequestFilter {

    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain chain) throws IOException, ServletException {
        String requestId = request.getHeader("X-Request-ID");
        if (requestId == null) requestId = UUID.randomUUID().toString();
        
        MDC.put("requestId", requestId);
        MDC.put("method", request.getMethod());
        MDC.put("path", request.getRequestURI());
        
        response.setHeader("X-Request-ID", requestId);
        
        long startTime = System.currentTimeMillis();
        try {
            chain.doFilter(request, response);
        } finally {
            long duration = System.currentTimeMillis() - startTime;
            MDC.put("statusCode", String.valueOf(response.getStatus()));
            MDC.put("durationMs", String.valueOf(duration));
            
            log.info("API request completed");  // all MDC fields auto-appended
            MDC.clear();
        }
    }
}
```

### API Metrics with Micrometer

```java
// Automatic with Spring Boot Actuator + Micrometer
// Available at /actuator/prometheus

// http.server.requests — automatic per-endpoint metrics
// Tags: method, uri, status, exception
// Histogram: p50, p95, p99 latency

// Custom business metrics
@Service
public class TransactionService {
    private final Counter txnCreatedCounter;
    private final Timer txnLatencyTimer;
    
    public TransactionService(MeterRegistry registry) {
        txnCreatedCounter = Counter.builder("transactions.created")
            .tag("type", "NEFT")
            .description("Number of transactions created")
            .register(registry);
            
        txnLatencyTimer = Timer.builder("transactions.processing.time")
            .description("Time to process a transaction")
            .publishPercentiles(0.5, 0.95, 0.99)
            .register(registry);
    }
    
    public Transaction create(CreateTransactionRequest req) {
        return txnLatencyTimer.recordCallable(() -> {
            Transaction txn = doCreate(req);
            txnCreatedCounter.increment();
            return txn;
        });
    }
}
```

### Distributed Tracing

```java
// Add traceId to every log line — correlate across services
// With Spring Boot + OpenTelemetry Java Agent:

// application.yml
management:
  tracing:
    sampling:
      probability: 0.1  # sample 10% of traces (100% in dev)

# JAVA_TOOL_OPTIONS: -javaagent:/opentelemetry-javaagent.jar
# OTEL_EXPORTER_OTLP_ENDPOINT: http://collector:4317
# OTEL_SERVICE_NAME: sme-service-apis

// traceId is automatically propagated:
// incoming request → extract traceId from W3C Traceparent header
// outgoing Retrofit/RestTemplate calls → inject traceId header
// Kafka messages → inject traceId in headers
// MongoDB queries → MDC includes traceId
```

---

## 23. API Design Checklist

Use this before every API review.

### Design

- [ ] Resources are nouns, not verbs
- [ ] URLs are hierarchical and reflect ownership
- [ ] Correct HTTP method for each operation
- [ ] Correct status codes (201 for create, 204 for delete, 202 for async)
- [ ] Consistent field naming (camelCase)
- [ ] ISO 8601 timestamps with timezone
- [ ] No float for money — use integer minor units or BigDecimal
- [ ] Validation errors return ALL errors at once, not just first
- [ ] Error responses have business error codes, not just HTTP status
- [ ] No stack traces in error responses

### Pagination & Filtering

- [ ] Default page size set, max page size enforced
- [ ] Cursor-based pagination for large collections
- [ ] Filters are validated against allowlist
- [ ] Sort fields validated against allowlist

### Security

- [ ] Every endpoint requires authentication (unless explicitly public)
- [ ] Every endpoint checks object-level authorization (BOLA)
- [ ] Input validation on every field
- [ ] Rate limiting configured
- [ ] Security headers set (HSTS, CSP, X-Frame-Options)
- [ ] No sensitive data in logs (mask card numbers, PANs, amounts if needed)

### Performance

- [ ] Response compressed (gzip/brotli enabled)
- [ ] Cache-Control headers set appropriately
- [ ] No N+1 queries — batch loads verified
- [ ] DB queries have indexes for the access pattern
- [ ] Projection used — only fetch needed fields
- [ ] Connection pooling configured
- [ ] Timeouts set on all outbound calls

### Reliability

- [ ] Idempotency key support for non-idempotent operations
- [ ] Circuit breaker on all external API calls
- [ ] Retry strategy defined (what to retry, how many times, backoff)
- [ ] Health check endpoint (/actuator/health)
- [ ] Graceful shutdown enabled

### Observability

- [ ] Request ID in every response
- [ ] Structured logging with traceId
- [ ] Latency histogram metrics per endpoint
- [ ] Error rate metrics per endpoint
- [ ] Alerts for p99 latency and error rate SLOs

### Versioning

- [ ] Version in URL (/api/v1/)
- [ ] No breaking changes without version bump
- [ ] Deprecation headers on old versions
- [ ] Sunset date announced and documented

---

## 24. Glossary

| Term | Definition |
|---|---|
| **Backpressure** | Mechanism for a consumer to signal to producer to slow down |
| **BOLA** | Broken Object Level Authorization — accessing resources you don't own |
| **Brotli** | Modern compression algorithm; 15–20% better than gzip |
| **BFF** | Backend for Frontend — dedicated API per client type |
| **Cache-Control** | HTTP header controlling caching behavior |
| **CDN** | Content Delivery Network — edge cache near users |
| **Circuit Breaker** | Stops calling a failing dependency; recovers gradually |
| **Connection Pool** | Reuse established connections instead of creating new ones |
| **Content Negotiation** | Client specifies preferred format; server responds accordingly |
| **CORS** | Cross-Origin Resource Sharing — browser security mechanism for cross-domain requests |
| **Cursor Pagination** | Page by stable cursor position, not offset number |
| **DataLoader** | Batching pattern to solve N+1 in GraphQL |
| **ETag** | Entity Tag — fingerprint of response, used for conditional requests |
| **Field Selection** | Client specifies which fields to return (`?fields=id,name`) |
| **gRPC** | Google RPC framework using Protocol Buffers over HTTP/2 |
| **HATEOAS** | Hypermedia links in responses telling clients what to do next |
| **HTTP/2** | HTTP with multiplexing, header compression, server push |
| **HTTP/3** | HTTP over QUIC (UDP), eliminates TCP head-of-line blocking |
| **Idempotency** | Same request N times = same result as once |
| **Idempotency Key** | Client-provided UUID that makes non-idempotent ops safe to retry |
| **Keep-Alive** | Reuse TCP connection for multiple HTTP requests |
| **mTLS** | Mutual TLS — both client and server authenticate with certificates |
| **N+1 Problem** | 1 query for list + N queries for related data — use batch loading |
| **OWASP** | Open Web Application Security Project — API security standard |
| **Outbox Pattern** | Store events in same DB transaction as data — guarantees delivery |
| **Preflight** | OPTIONS request browser makes before CORS cross-origin request |
| **Projection** | Fetch only needed columns/fields from DB |
| **Protocol Buffers** | Binary serialization format — 3–10x smaller than JSON |
| **Rate Limiting** | Restrict requests per time window per user/IP/endpoint |
| **Read Replica** | DB copy for read traffic; lag may cause stale reads |
| **SSE** | Server-Sent Events — one-way server→client event stream |
| **SSRF** | Server-Side Request Forgery — server fetches attacker-controlled URL |
| **Stale-While-Revalidate** | Serve cached response immediately, refresh in background |
| **Throttling** | Slow down requests (queue) vs rate limiting (reject) |
| **Transactional Outbox** | Pattern ensuring events are published exactly once with the DB write |
| **Vary** | HTTP header telling caches to create separate entries per header value |
| **WebSocket** | Bidirectional persistent TCP connection for real-time communication |
| **Webhook** | HTTP callback — you call the client's URL when an event occurs |
