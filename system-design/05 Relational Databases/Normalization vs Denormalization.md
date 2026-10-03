---
title: Normalization vs Denormalization
tags: [system-design, databases]
type: note
status: evergreen
created: 2026-10-01
updated: 2026-10-01
summary: Normalization as "one fact, one place", the update anomalies it prevents, why read-heavy systems denormalize on purpose, and how denormalized copies are kept consistent — events, CDC, materialized views.
related: ["[[Database Scaling Patterns]]", "[[Change Data Capture]]", "[[Caching Strategies]]", "[[MongoDB Architecture]]"]
---

# Normalization vs Denormalization

Normalization is the discipline of storing **each fact exactly once** and deriving everything else with joins; denormalization is the deliberate re-introduction of redundant copies to make reads cheap. Neither is a virtue by itself: normalization optimizes for *write correctness*, denormalization for *read performance*, and every real system at scale runs a mix. The craft is knowing which facts to copy, and — the part people skip — engineering how the copies stay honest.

## Why this matters

This trade-off is quietly the backbone of half of system design. A cache is a denormalized copy of query results ([[Caching Strategies]]). A read replica is a denormalized copy of the whole database ([[Replication]]). A Twitter timeline precomputed per follower is denormalization at planetary scale ([[Twitter Timeline Design]]). A MongoDB document embedding its line items is denormalization as a data model ([[MongoDB Architecture]]). In every case the same two questions decide success: *how stale may a copy be?* and *what mechanism updates it?* Learn the questions here, at single-database scale, and you can reuse them everywhere — including in interviews, where "how do you keep the copy consistent?" is the follow-up that separates levels.

## How it works

### Normalization: one fact, one place

Skip the pedantic march through normal forms; the normal forms are increasingly strict formalizations of a single idea: **a fact should live in exactly one row of one table, identified by a key.** "Customer 42's email address" is one fact. If it appears in `customers`, and again in every row of `orders`, and again in `invoices`, you no longer store the fact — you store four opinions about it.

Concretely, the before/after of normalizing one table:

```
-- Denormalized: one wide table, facts repeated per order
orders(order_id, amount, customer_id, customer_name, customer_email, customer_city)
-- "customer 42's email" appears once PER ORDER

-- Normalized: each fact once, joined on demand
customers(customer_id, name, email, city)
orders(order_id, customer_id, amount)
-- "customer 42's email" appears exactly once, whatever the order count
```

Redundant copies create the three classic **update anomalies**:

- **Update anomaly**: changing the customer's email requires finding and updating every copy; miss one and the database now *contains a contradiction* — and no query can tell you which copy is true.
- **Insert anomaly**: you can't record a new customer's email until they place an order, because the email only lives on order rows.
- **Delete anomaly**: deleting a customer's last order deletes your only record of their email.

A reasonably normalized schema (informally: every non-key column states a fact about the key, the whole key, and nothing but the key) eliminates all three by construction. Writes become small and safe — update one row, every reader everywhere sees the new truth. Constraints and foreign keys can actually enforce integrity because each fact has one enforcement point. This is why **normalization is the correct default for the system of record**, especially where money and identity are involved: correctness-critical writes want one place to be right.

The price is paid at read time: real screens aggregate many facts, so queries grow joins — five, eight, twelve tables — and each join costs CPU, index lookups ([[Database Indexes]]) and planner complexity. At modest scale this price is near zero; well-indexed joins are fast, and "joins are slow" is mostly folklore at OLTP sizes. The price becomes real at high read volume, wide fan-outs (one query touching thousands of rows across tables), or when the join spans [[Sharding|shards]] — cross-shard joins are where normalization's read cost stops being theoretical.

### Denormalization: paying write-side costs to buy reads

When a hot read path can't afford its joins or aggregations at query time, you precompute: store the answer, or the expensive parts of it, redundantly.

Typical forms, roughly in order of increasing commitment:

| Form | Example | What it saves |
|---|---|---|
| Duplicated column | `orders.customer_name` copied from `customers` | a join on every order list |
| Precomputed aggregate | `accounts.balance` instead of `SUM(ledger_entries)` | an aggregation over the full history per read |
| Precomputed join table / read model | an `order_summary` table shaped exactly like the API response | the whole multi-join query |
| Embedded document | Mongo order document containing its items | all joins — the read is one document fetch |
| Materialized view | database-maintained stored query result | the query, with the DB managing storage |

Interest on the debt, which accrues forever:

- **Every write to the source fact must now also update the copies** — writes get slower, more complex, and grow failure modes.
- **The anomalies return**, exactly as the theory predicts: any path that updates the source but not the copy creates a contradiction. Denormalization converts a performance problem into a *consistency maintenance* problem.
- Storage grows, and — subtler — the schema now encodes *assumptions about read patterns*; when the product changes, denormalized shapes are harder to migrate than normalized ones.

The honest framing: **denormalize deliberately, per identified hot read, with a named consistency mechanism — never by habit or heap.**

Signals that a specific denormalization has earned its keep:

- The read is **hot and measured** — it shows up in the top queries by total time, not in someone's anxiety.
- Rungs below it on [[Database Scaling Patterns|the scaling ladder]] are exhausted for this path: the query is tuned, indexed, and a plain cache can't hold it (too many distinct keys, or invalidation is harder than maintaining a proper read model).
- The staleness budget is **stated and generous** — seconds or more. Zero-staleness denormalization (same-transaction copies) is only worth it when the read volume is extreme and the write volume is low.
- The read shape is **stable** — product screens that change weekly make precomputed shapes churn; stable API contracts make them cheap.
- Someone owns the **reconciliation story** — a copy nobody can verify is a liability wearing a performance costume.

### Keeping the copies consistent

This is the load-bearing engineering. Options, from strongest to loosest:

1. **Same transaction.** Update source and copy in one [[ACID Transactions|transaction]] (app code or trigger). Copies are never stale — but every write pays the full cost, contention rises (the famous example: a single `account.balance` row serializing all of an account's writes), and it only works inside one database.
2. **Events + async updaters.** The writer commits the source change and publishes an event; a consumer updates the read copies. Copies are eventually consistent — honest staleness of milliseconds to seconds. The dual-write trap applies in full: "commit DB + publish event" is not atomic, so route the event through [[The Outbox Pattern]], and make updaters idempotent ([[Idempotency]]) because delivery is at-least-once.
3. **Change Data Capture.** Instead of hand-written events, tail the database's own WAL ([[How Databases Store Data]]) with Debezium-style [[Change Data Capture]]: every committed change becomes a stream, and copy-maintainers subscribe. No application code can "forget" to emit; the log is the truth. This is the industrial-strength backbone for search indexes, caches, and read models — and it's [[CQRS]] when the read models get their own stores.
4. **Materialized views.** The database maintains a stored query result. PostgreSQL's are *manually* refreshed (`REFRESH MATERIALIZED VIEW`, optionally `CONCURRENTLY` with a unique index — it recomputes the whole view, so freshness is whatever your refresh schedule is). Some systems maintain views incrementally and transactionally (e.g. Oracle, SQL Server indexed views; specialized engines). Great for analytics-ish reads inside one DB; not a cross-system mechanism.
5. **Periodic rebuild / reconciliation.** Recompute copies on a schedule, and — in money systems — run reconciliation jobs that *diff* copies against the source and alert. Even with mechanisms 1–4, a reconciliation loop is the auditor-grade backstop, because bugs happen and copies drift.

The design question for every denormalized copy: **what is its maximum tolerable staleness, and which mechanism above delivers that at acceptable write cost?** Write the answer down next to the copy.

Choosing between the mechanisms at a glance:

| Mechanism | Staleness | Write cost | Crosses DB boundary? | Fails how |
|---|---|---|---|---|
| Same transaction | zero | every write pays; contention on hot copies | no | slow/contended writes |
| Events via outbox | ms–seconds | one extra insert per write | yes | consumer lag or bugs → drift until reconciled |
| CDC from the WAL | ms–seconds | ~zero on the write path | yes | pipeline lag; schema changes need care |
| Materialized view (Postgres-style) | refresh interval | periodic recompute | no | stale between refreshes; refresh cost grows with data |
| Periodic rebuild + reconciliation | minutes–hours | batch job | yes | everything between runs; it's the backstop, not the mechanism |

## Common misconceptions

- **"Joins are slow, so normalize less."** At OLTP scale with proper indexes, joins are fast; measure before denormalizing. The genuine join killers are huge fan-outs, cross-shard queries, and analytics-shaped scans — which have their own tools ([[Time Series and Analytics Stores]]).
- **"Denormalization is a NoSQL thing."** It's a data-design decision available (and common) in SQL; conversely, document stores can be modelled normalized with references. The engine doesn't decide; your read/write ratio and consistency budget do.
- **"We'll keep the copies in sync in app code."** Every write path, forever, including the batch job someone writes next year? Without an enforcing mechanism (transaction, trigger, outbox/CDC) plus reconciliation, drift is a *when*, not an *if*.
- **"3NF or bust" / normal-form pedantry.** The forms are a ladder of rigor for "one fact, one place", not a compliance regime. Knowing *why* update anomalies happen beats reciting BCNF's definition.
- **"The balance column is obviously fine."** A stored balance is a denormalized aggregate of the ledger. It's often the right call — but it needs a stated mechanism (same-transaction update) *and* reconciliation against the ledger, which is precisely how [[Stripe and Payment System Design|payment systems]] treat it.

## In your stack

- MongoDB makes this trade-off per collection: **embedding is denormalization, referencing is normalization.** Embed what is read together and owned together (order + items); reference what is shared and independently updated (customer ← orders). Unbounded growth inside a document (append-forever arrays) is the embed-trap to refuse — see [[MongoDB Architecture]].
- The ledger/balance pattern is your fintech home ground: ledger entries are the normalized source of truth; the account balance is a denormalized aggregate maintained in the same transaction, with nightly reconciliation. An auditor will ask to see the reconciliation, not the balance.
- Your event estate (Kafka → SQS → Redis Streams) is the natural transport for mechanism 2/3; the outbox relay pattern matters at Niyo specifically because "Mongo write + queue publish" is a dual write in every service that does it inline.
- Redis caches of API responses are denormalized copies with TTL-based consistency — the same staleness question, answered with expiry. [[Cache Invalidation]] is this note's problem wearing a different hat.
- Cross-service copies (service B caching service A's customer names) are denormalization *across ownership boundaries* — they need a contract: an event feed from the owner, or an accepted staleness. Ad-hoc nightly pulls are how contradictions enter regulated reports.

## Interview angle

The 60-second version: *"Normalization stores each fact once, so writes are small and can't create contradictions — that's why it's the default for the system of record. The price is joins at read time. When a hot read path can't afford its joins, you denormalize: duplicate columns, precomputed aggregates, read models shaped like the response. That re-opens update anomalies, so every denormalized copy needs a named consistency mechanism — same-transaction update, events via an outbox, CDC from the WAL, or a materialized view — chosen by the copy's staleness budget. And in money systems, reconciliation jobs as the backstop. The design question is never 'normalize or not', it's 'which facts get copies, how stale may each copy be, and what keeps it honest'."*

Classic probes:

- "What breaks when you store `customer_name` on orders?" — update anomaly; though for *invoices* the copy is a feature: a point-in-time snapshot, legally correct.
- "How would you keep a search index in sync with the DB?" — CDC from the WAL, with idempotent appliers and periodic rebuild as backstop.
- "Why is a stored balance dangerous, and how do you make it safe?" — it's a denormalized aggregate; same-transaction maintenance plus reconciliation against the ledger.
- "Snapshot or cache?" — the sharpest one-line test in this note: if the source changes, should the copy change?
- The scaling bridge: "your read path is dying under joins — walk the options" → this note plus [[Database Scaling Patterns]].

## Check yourself

> [!question]- Q: Explain the three update anomalies with one example each, and state the single principle that prevents all three.
> Update: customer email copied onto every order row — changing it means updating many rows, and missing one leaves two "truths". Insert: you can't store a new customer's email until their first order exists, because email lives only on orders. Delete: removing the last order erases the email entirely. All three vanish when each fact lives in exactly one place keyed by its identifier — the customer's email in one `customers` row — which is the whole content of normalization.

> [!question]- Q: What happens if a team adds an `account_balance` column maintained by application code on each transaction insert, with no other mechanism? Trace the failure.
> Some write path eventually updates the ledger without the balance (a bulk correction script, a new refund flow, a crash between the two statements if they're not in one transaction). The balance now disagrees with `SUM(ledger)` — silently, since reads use the column. Money decisions (overdraft checks, user-visible balance) are made on the wrong number until someone notices. Safe version: update balance and ledger in one [[ACID Transactions|transaction]] (accepting the contention on hot accounts), or compute balance from an evented projection — and in either case run a reconciliation job that diffs balance vs ledger and alarms on drift.

> [!question]- Q: When is copying `customer_name` onto an invoice row NOT an anomaly but a requirement?
> When the copy is semantically a *snapshot*, not a cache: an invoice must show the name (and address, and price) as they were at issuance, for legal and audit reasons. The "fact" being stored is "what the invoice said", which is itself a one-place fact. The test: if the source changes, *should* this copy change? No → snapshot, store it proudly. Yes → denormalized cache, needs a sync mechanism.

> [!question]- Q: A read model updated by events is found to be wrong for 0.1% of records. The consumer code looks correct. What are the likely drift sources, and what's the structural answer?
> Likely sources: events dropped or double-applied before the consumer was fully idempotent; a historical bug window whose damage was never repaired (fixing the code doesn't fix the rows); a write path that bypasses events (bulk script, manual correction); event reordering across partitions applied without version checks; or the dual-write race if the service publishes inline instead of via an outbox. The structural answer is twofold: prevention — outbox/CDC so events can't be forgotten, idempotent and order-tolerant consumers ([[Idempotency]], versioned upserts); and repair — a reconciliation job that periodically diffs the read model against the source and re-emits or patches divergent rows. Event-driven copies without reconciliation are copies whose drift you simply haven't measured yet.

> [!question]- Q: Mini design: product pages read product + seller + rating-aggregate at 20,000 reads/sec; ratings are written 50/sec. Normalize or denormalize, and how do copies stay fresh?
> Source of truth stays normalized: products, sellers, individual ratings. The read path gets a denormalized read model — product id → rendered page document including seller name and precomputed avg rating — in a cache or document store. Maintenance: rating writes update the aggregate asynchronously via events/CDC (staleness budget: minutes is fine for an average), seller-name changes likewise (rare). Backstop: periodic rebuild. This is a 400:1 read/write ratio — the canonical shape where paying 50 small async updates/sec to save 20,000 join-queries/sec is obviously right, and the mechanism (outbox → consumer → read model) is stated, not hoped for.

## Sources

- *Designing Data-Intensive Applications*, Kleppmann — Chapter 2 (relational vs document, normalization and when denormalized/embedded wins), Chapter 3 (materialized views, derived data framing); Part III's "systems of record and derived data" is this note generalized.
- Codd, E. F. (1970), "A Relational Model of Data for Large Shared Data Banks" — the origin of the relational model the normal forms formalize.
- PostgreSQL documentation: "Materialized Views" (`REFRESH MATERIALIZED VIEW [CONCURRENTLY]` semantics).
- MongoDB documentation: "Data Model Design" — embedding vs referencing, the document-world statement of this exact trade-off.
