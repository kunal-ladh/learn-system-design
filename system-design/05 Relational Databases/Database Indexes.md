---
title: Database Indexes
tags: [system-design, databases]
type: note
status: evergreen
created: 2026-10-01
updated: 2026-10-01
summary: How B+-tree indexes work, how to design composite and covering indexes, why every index taxes writes, and when the optimizer will ignore your index anyway.
related: ["[[How Databases Store Data]]", "[[B-Tree vs LSM-Tree]]", "[[Secondary Indexes on Sharded Data]]"]
---

# Database Indexes

An index is a sorted, separately-maintained copy of some of a table's columns, organized so the database can find matching rows by touching a handful of pages instead of scanning all of them. Indexes are the single highest-leverage tool in database performance — and the most commonly misused, because people add them by superstition instead of by reading query plans. This note gives you the B+-tree mental model, the composite-index rules, and the habit (`EXPLAIN`) that replaces superstition.

## The problem that created it

Picture a young system: a `transactions` table with ten thousand rows. `SELECT * FROM transactions WHERE user_id = 42` scans the whole table — and at ten thousand rows (a few hundred pages, mostly cached) nobody notices; it returns in a millisecond.

The table grows. At ten million rows the same query reads hundreds of thousands of pages. From [[How Databases Store Data]] you know cost is *pages touched*: the query now takes seconds, hammers the buffer pool (evicting everyone else's hot pages), and every concurrent copy of it multiplies the damage. The workload didn't change; the data volume made linear scans unaffordable. What's needed is a structure that finds the few relevant pages in logarithmic time — without re-sorting the table for every different query shape.

## What it solves

- **Point lookups** (`WHERE id = ?`) and **range scans** (`WHERE created_at > ?`) in logarithmic + sequential time instead of linear.
- **Sorted retrieval** (`ORDER BY`) and efficient `JOIN`s without sorting at query time — the index *is* the sort.
- **Uniqueness enforcement** — a unique constraint is implemented as an index.
- **Covering reads**: serving a query entirely from the index without visiting the table at all.

## How it works

### B+-tree mechanics, concept level

The default index structure in PostgreSQL and InnoDB is the **B+-tree**: a tree whose nodes are database pages. Internal pages hold only separator keys and child pointers; **all actual entries live in the leaf pages, which are linked left-to-right in sorted order**.

Two numbers make the structure click:

- A page holds **hundreds of keys** (fanout in the hundreds), so the tree is extremely shallow: 3–4 levels comfortably cover hundreds of millions of rows.
- A lookup therefore costs **3–4 page reads**, and the top levels are almost always resident in the buffer pool — so in practice a lookup is often a single disk read, or none.

A range scan descends once to the first matching leaf, then walks the leaf chain sequentially — which is why `BETWEEN`, `ORDER BY ... LIMIT`, and "latest N" queries love B+-trees. Inserts go to the correct leaf; a full leaf **splits** into two, occasionally propagating a split upward. The tree stays balanced by construction. (The write-optimized alternative to all this is the LSM-tree — see [[B-Tree vs LSM-Tree]].)

What a leaf entry points at differs by engine, and it matters: in PostgreSQL it holds the row's physical address in the heap; in InnoDB a secondary index leaf holds the **primary key value**, so the engine does a second B+-tree descent to fetch the row ([[How Databases Store Data]] covers why).

### Composite indexes and the leftmost-prefix rule

An index on `(a, b, c)` is a single sort: by `a`, then `b` within equal `a`, then `c`. It is *not* three indexes. The consequence is the **leftmost-prefix rule**: the index supports seeking on `a`, on `a AND b`, on `a AND b AND c` — but a predicate on `b` alone cannot seek, because rows with equal `b` are scattered throughout the sort order. (Think of a phone book sorted by last name, then first name: finding every "Priya" without a last name means reading the whole book.)

Ordering rules that follow directly:

- **Equality columns first, range column last.** For `WHERE user_id = ? AND created_at > ?`, index `(user_id, created_at)`. The range predicate "uses up" the index: columns after the first range column can no longer be seeked, only filtered.
- **An index on `(a, b)` makes a separate index on `(a)` redundant** (the prefix does its job) — but not one on `(b)`.
- `ORDER BY` can be satisfied for free if it matches the index order after the equality prefix: `WHERE user_id = ? ORDER BY created_at DESC LIMIT 20` is a dream query for `(user_id, created_at)`.

### Covering indexes

If the index contains *every* column a query needs, the database can answer from the index alone — an **index-only scan** — skipping the table fetch entirely. That table fetch (random heap hop or second B+-tree descent) is often the dominant cost, so covering a hot query can be a 10x win.

- PostgreSQL: either put the columns in the key, or use `INCLUDE (col)` (v11+) to carry non-searchable payload columns; index-only scans additionally depend on the visibility map being reasonably fresh (autovacuum).
- InnoDB: every secondary index implicitly "includes" the primary key, so `SELECT id FROM t WHERE email = ?` is covered by an index on `(email)` for free.

The habit: for the handful of truly hot queries, look at the `SELECT` list, not just the `WHERE` clause.

### Selectivity

**Selectivity** is the fraction of rows a predicate matches. Indexes shine when the predicate is highly selective (matches few rows) and are useless or harmful when it isn't: fetching 40% of a table through an index means millions of random page hops, which is *slower* than one sequential scan. This is why indexing a `status` column with three values rarely helps on its own — but `(status, created_at)` might, if queries always combine them and the interesting status is rare. The optimizer estimates selectivity from **statistics** (histograms of column values), which is why stale statistics produce bizarre plans.

### Every index taxes writes

An index is a redundant sorted copy that must be maintained *synchronously, inside the writing transaction*:

- Every `INSERT` and `DELETE` touches **every index** on the table.
- An `UPDATE` touches every index whose columns changed — and in PostgreSQL, a non-HOT update (any update where an indexed column changed, or the page is full) inserts a new entry into *all* indexes, changed or not.
- More indexes → more pages dirtied per write → more WAL, more buffer-pool pressure, slower writes, bigger storage, longer vacuum/maintenance.

A table with eight indexes pays roughly eight extra page modifications per insert. Reads pay for writes; the design question is always *which* reads justify *which* write tax. Unused indexes are pure tax — both Postgres (`pg_stat_user_indexes`) and MySQL (`sys.schema_unused_indexes`) can tell you which ones have never been read.

### When the optimizer ignores your index

The optimizer compares estimated costs; your index loses when:

- **Low selectivity** — predicted to match a large fraction of the table, so a sequential scan is genuinely cheaper. The optimizer is usually right about this.
- **A function or expression wraps the column**: `WHERE lower(email) = ?` or `WHERE DATE(created_at) = ?` cannot use a plain index on the column. Fix: index the expression (Postgres expression index, MySQL functional index) or rewrite the predicate as a range.
- **Type mismatch / implicit cast**: comparing a `VARCHAR` column to a number (or mismatched collations in joins) forces a cast on the column side — same effect as a function.
- **Leading wildcard**: `LIKE '%gmail.com'` can't seek a B+-tree (no sorted prefix to descend by). `LIKE 'abc%'` can.
- **Leftmost prefix violated** — filtering only on a non-leading column of a composite index.
- **Stale statistics** after a bulk load or mass delete — the optimizer is costing a table that no longer exists. `ANALYZE` it.
- **Tiny tables** — a one-page table is cheapest to scan; that's fine.

### EXPLAIN as a habit

`EXPLAIN` shows the plan the optimizer chose; `EXPLAIN ANALYZE` (Postgres; MySQL 8.0.18+ has `EXPLAIN ANALYZE` too) *runs* the query and shows actual row counts and timings next to estimates. The two habits that matter:

1. **Never add an index without looking at the plan before and after.** Otherwise you are guessing, and the write tax is permanent.
2. **Compare estimated vs actual rows.** A plan that estimates 10 rows and finds 2 million is a statistics problem, not an indexing problem — no amount of index-adding fixes it.

## Trade-offs

| You gain | You pay |
|---|---|
| Logarithmic lookups and cheap range scans | Every write updates every index, synchronously |
| Free `ORDER BY` / efficient joins | Storage — large indexes can rival the table |
| Index-only scans for covered queries | Buffer pool shared between table and index pages |
| Uniqueness enforcement | Slower bulk loads, more WAL, index bloat needing maintenance |

## Failure modes

- **The missing-index cliff**: a query that was "fine" for a year crosses a size threshold and starts scanning its way through the buffer pool, evicting hot pages and degrading *unrelated* queries. Detected via slow-query logs and a sudden rise in disk reads.
- **The over-indexed write path**: insert-heavy table accretes 10+ indexes over years of "quick fixes"; write latency climbs, replication lag follows ([[Replication]]). Nobody remembers which indexes are load-bearing.
- **Plan flips**: data distribution drifts, statistics update, and the optimizer suddenly picks a different plan at 3 a.m. — classic cause of "nothing deployed, everything slow". Mitigation: monitoring on plan changes, timely `ANALYZE`, and in extremis plan pinning/hints.
- **Index bloat** (Postgres): heavy update/delete churn leaves half-empty pages; the index grows and slows until `REINDEX`/`pg_repack`.
- **Scatter-gather on sharded data**: once a table is [[Sharding|sharded]], a secondary index is either local to each shard (queries fan out to all shards) or global (writes fan out) — see [[Secondary Indexes on Sharded Data]]. Indexing decisions stop being single-node decisions.

## When NOT to use it

- **Write-dominated, rarely-read tables** — audit trails, raw event dumps. Index only what the (rare) reads actually need; consider whether the data belongs in an analytics store instead ([[Time Series and Analytics Stores]]).
- **Low-cardinality single columns** (`status`, booleans, enums) on their own — the selectivity isn't there. Partial indexes (`WHERE status = 'PENDING'`) are the honest version when one rare value is hot.
- **Speculative indexes** ("we might query by this someday") — permanent write tax for a hypothetical read.
- **As a substitute for query redesign** — an index that rescues `SELECT *` over a six-way join is often treating the symptom; sometimes the fix is a projection, a cache ([[Caching Strategies]]), or a precomputed table ([[Normalization vs Denormalization]]).

## In your stack

- Everything here transfers to MongoDB nearly verbatim: WiredTiger secondary indexes are B+-trees, the leftmost-prefix rule is the "ESR rule" (equality, sort, range) in Mongo index design, and `explain()` is the same habit — see [[MongoDB Architecture]].
- In Spring/JPA services the write tax is invisible in code review: `@Index` annotations and Flyway migrations add indexes, nothing ever removes them. Auditing unused indexes is cheap and almost always finds dead weight.
- For fintech list screens — "last 20 transactions for user X" — the canonical index is `(user_id, created_at)` with matching `ORDER BY ... LIMIT`; verify with `EXPLAIN` that pagination isn't scanning past offset rows (prefer keyset pagination over `OFFSET`).
- Watch for ORM-generated predicates that defeat indexes: implicit casts from mismatched Java/SQL types, and `upper()`/`lower()` wrappers for case-insensitive search without a matching expression index. [[N+1 Queries and ORM Pitfalls]] covers the neighbouring traps.

## Interview angle

The 60-second version: *"An index is a sorted B+-tree copy of chosen columns; lookups descend 3–4 cached pages instead of scanning the table, and range scans walk linked leaves. Composite indexes are one sort order, so only leftmost prefixes can seek — equality columns first, range last. If the index covers all selected columns the table fetch is skipped entirely. The price is that every write maintains every index synchronously, so indexes are bought with write latency. And the optimizer only uses an index when selectivity and statistics justify it — so the habit is EXPLAIN before and after, never superstition."*

Classic probes: why doesn't an index on `(b)` help `WHERE a = ? AND b = ?` as much as `(a, b)`? Why is `LIKE '%foo'` un-indexable? Why did adding an index slow down inserts? Design follow-up: pick indexes for a given query mix and justify the write tax — and the sharded version, where they'll steer you to [[Secondary Indexes on Sharded Data]].

## Check yourself

> [!question]- Q: Explain the leftmost-prefix rule to a junior engineer using the phone-book analogy.
> A composite index on `(last_name, first_name)` is a phone book: sorted by last name, and by first name only *within* each last name. Finding "Sharma, Priya" or all "Sharma" entries is a direct seek. Finding every "Priya" regardless of last name means reading the entire book, because Priyas are scattered — the sort order gives you no way to jump to them. An index on `(a, b)` therefore serves `a` and `a+b` queries, but not `b` alone.

> [!question]- Q: What happens if you add `(user_id, created_at, amount, status)` as an index to cover one report query, on a table doing 2,000 inserts/sec?
> Every insert now writes an additional entry into this index: more dirty pages, more WAL volume, and more buffer-pool pressure — at 2,000/sec this is a measurable write-latency and replication-lag cost, paid continuously, for one report. Better options: run the report on a read replica, schedule it off-peak, or maintain a small aggregate table. If the query is genuinely hot and interactive, the covering index may still be worth it — but that's a decision to make with numbers from EXPLAIN, not by default.

> [!question]- Q: A query filtered on an indexed column is doing a sequential scan. List four plausible reasons before blaming the optimizer.
> (1) Low selectivity — the predicate matches a big fraction of the table, so a scan is actually cheaper. (2) A function/expression or implicit type cast wraps the column, making the index inapplicable. (3) The filter is on a non-leading column of a composite index. (4) Statistics are stale after a bulk change, so the optimizer's row estimates are wrong — check estimated vs actual in EXPLAIN ANALYZE, then ANALYZE the table.

> [!question]- Q: Mini design: a payments table with queries "all payments for account X newest first", "payment by idempotency key", and a nightly full scan for reconciliation. Propose indexes.
> `(account_id, created_at)` — serves the list query including its ORDER BY/LIMIT via leftmost prefix. A unique index on `(idempotency_key)` — point lookup and it *enforces* [[Idempotency]], which is the real requirement in a payment system. The nightly reconciliation wants no index: a sequential scan is the right plan for reading everything, ideally on a replica so it doesn't fight the buffer pool of the primary.

## Sources

- Use The Index, Luke — Markus Winand's free book on SQL indexing; the definitive treatment of B+-tree anatomy, composite ordering, and function-defeated indexes (also published as *SQL Performance Explained*).
- *Designing Data-Intensive Applications*, Kleppmann — Chapter 3 for B-trees vs LSM-trees and the storage context.
- PostgreSQL documentation: "Indexes" chapter (multicolumn indexes, index-only scans, expression and partial indexes) and "Using EXPLAIN".
- MySQL 8.0 Reference Manual: "Multiple-Column Indexes", "EXPLAIN Output Format", and "Optimizing InnoDB Queries".
