---
title: Transaction Isolation Levels
tags: [system-design, databases]
type: note
status: evergreen
created: 2026-10-01
updated: 2026-10-01
summary: The four isolation levels, the exact anomaly each one readmits — dirty reads, non-repeatable reads, phantoms, lost updates, write skew — plus MVCC, SELECT FOR UPDATE, and what Postgres and MySQL actually default to.
related: ["[[ACID Transactions]]", "[[How Databases Store Data]]", "[[Consistency Models]]"]
---

# Transaction Isolation Levels

Isolation — the I of [[ACID Transactions]] — is not a boolean. It's a dial with four standard positions, and every position below the top readmits specific, named concurrency anomalies. Databases default to the middle of the dial because full serializability costs throughput, which means **the default behaviour of your database permits races that most developers believe are impossible**. This note names each anomaly, maps it to the level that stops it, and shows the practical tools (MVCC snapshots, `SELECT FOR UPDATE`) that let you buy exactly as much isolation as a given transaction needs.

## Why this matters

Two concurrent requests read a balance of 100, each deducts 70, both commit, and the account sits at −40 — at the *default* settings of PostgreSQL and MySQL, with every statement wrapped in a transaction, with zero bugs in your SQL. Isolation anomalies are the most common way "correct" code loses money under concurrency, and they don't show up in tests because tests rarely race. The vocabulary here — dirty read, lost update, write skew — is also a reliable interview discriminator between people who have used transactions and people who understand them.

## How it works

### The ladder and what each level readmits

The SQL standard defines four levels. Read them as "what am I still exposed to at this level":

| Anomaly | Read uncommitted | Read committed | Repeatable read | Serializable |
|---|---|---|---|---|
| **Dirty read** — seeing another transaction's uncommitted write | possible | prevented | prevented | prevented |
| **Non-repeatable read** — re-reading a row mid-transaction and getting a different value | possible | possible | prevented | prevented |
| **Phantom** — re-running a *query* and seeing new/vanished rows | possible | possible | possible (see engine notes) | prevented |
| **Lost update** — two read-modify-writes, one silently overwrites the other | possible | possible | engine-dependent (see below) | prevented |
| **Write skew** — two transactions read overlapping data, write disjoint rows, jointly break an invariant | possible | possible | possible | prevented |

The anomalies in words:

- **Dirty read**: transaction B reads a row A has modified but not committed; A then aborts — B acted on data that never existed. Essentially no one runs this level deliberately; PostgreSQL doesn't even implement it (asking for read uncommitted gives you read committed).
- **Non-repeatable read**: B reads a row, A updates and commits it, B reads the same row again and sees a different value inside one transaction. Breaks any logic that assumes a stable view (e.g. computing a report in multiple queries).
- **Phantom**: B runs `SELECT count(*) WHERE status = 'PENDING'`, A inserts a new pending row and commits, B re-runs the query and the count changed. Not a changed row — a changed *result set*; row locks on what you read can't stop rows that didn't exist yet.
- **Lost update**: both A and B read `counter = 10`, both write `11`, both commit. One increment vanished. The classic shape is any read-modify-write done in application code.
- **Write skew**: the subtle one. Invariant: at least one doctor on call. A checks "two doctors on call, fine", removes herself; B simultaneously checks "two on call, fine", removes himself; both commit; zero doctors on call. Neither touched the other's row, every individual write was valid — the *combination* broke the invariant. No row lock catches this, because each transaction wrote a row the other never read-locked. Only true serializability (or an explicit materialized lock) prevents it.

The lost update, drawn as a timeline — this exact interleaving runs fine at Postgres read committed and InnoDB repeatable read:

| Time | Transaction A | Transaction B | `balance` on disk |
|---|---|---|---|
| t1 | `BEGIN; SELECT balance` → 100 | | 100 |
| t2 | | `BEGIN; SELECT balance` → 100 | 100 |
| t3 | computes 100 − 70 = 30 | | 100 |
| t4 | | computes 100 − 70 = 30 | 100 |
| t5 | `UPDATE balance = 30; COMMIT` | | 30 |
| t6 | | `UPDATE balance = 30; COMMIT` | **30** |

Two successful 70-unit deductions; the account paid out 140 and shows 30 instead of −40 — one update is simply gone. Nothing errored, every statement was individually correct, and both commits were acknowledged. Every fix in this note (FOR UPDATE, atomic conditional update, optimistic version check, Postgres RR's abort, serializable) is a different way of making t5/t6 collide instead of silently stacking.

### What the big two actually do

**Defaults: PostgreSQL defaults to read committed; MySQL InnoDB defaults to repeatable read.** Same SQL, different races — worth knowing cold.

**PostgreSQL**
- *Read committed*: each **statement** sees a snapshot of data committed before that statement began. Two statements in one transaction can see different states.
- *Repeatable read*: the whole **transaction** sees one snapshot (taken at its first query). This also prevents phantoms in Postgres — stronger than the SQL standard requires. Lost updates are *detected*: if you update a row that a concurrent transaction updated and committed after your snapshot, your transaction aborts with a serialization error. **Your code must be prepared to retry.**
- *Serializable*: repeatable read plus **Serializable Snapshot Isolation (SSI)** — the database tracks read/write dependencies between concurrent transactions and aborts one whenever a cycle (a potential write-skew outcome) appears. Optimistic: no extra blocking, but a retry loop is mandatory.

**MySQL InnoDB**
- *Repeatable read* (default): plain `SELECT`s read from a **consistent snapshot** taken at the first read — so non-repeatable reads and (for plain reads) phantoms don't appear. But plain snapshot reads take no locks, so **lost updates are NOT prevented**: read-modify-write in app code still races. Locking reads (`SELECT ... FOR UPDATE`) read the *latest* committed data and take **next-key locks** (row + gap), which block other transactions from inserting into the scanned range — preventing phantoms for locking reads.
- *Serializable*: implicitly converts plain `SELECT`s into shared locking reads — pessimistic, deadlock-prone under contention, rarely used in OLTP practice.

### MVCC: why readers don't block writers

Both engines implement snapshot-based levels with **multi-version concurrency control**: an update doesn't overwrite the row in place; it creates a **new version**, and old versions remain available to transactions whose snapshot predates the change (PostgreSQL keeps old versions in the table heap and garbage-collects them with vacuum; InnoDB reconstructs them from undo logs). Each transaction's snapshot determines which version of each row it sees.

The payoff: **readers never block writers and writers never block readers** — long reads and concurrent updates coexist without lock queues. The costs: version bloat under churn (vacuum/purge lag), and the fact that a snapshot is a *view*, not a *lock* — seeing consistent data gives you no right to act on it unchallenged, which is exactly why lost updates survive MVCC.

### SELECT FOR UPDATE and friends: buying isolation per-query

Rather than raising the level globally, you can escalate specific reads:

- **`SELECT ... FOR UPDATE`** takes exclusive row locks on the rows read: concurrent writers (and other FOR UPDATE readers) block until you commit. This is the standard fix for read-modify-write — it turns "check then act" into a critical section. Pessimistic, simple to reason about; costs lock waits and deadlock risk (keep lock order consistent, keep transactions short).
- **Atomic single-statement writes** sidestep the read entirely: `UPDATE accounts SET balance = balance - 70 WHERE id = ? AND balance >= 70`, then check rows-affected. One statement, no race window, no lock held across application code. Prefer this shape when it fits.
- **Optimistic locking** (a version column; `UPDATE ... WHERE id = ? AND version = ?`) detects conflicts at write time instead of locking at read time — JPA's `@Version` does exactly this. Best under low contention; under high contention it degrades into a retry storm.

Rule of thumb: contention rare → optimistic; contention real and the row is the natural serialization point → `FOR UPDATE`; invariant spans *multiple* rows that no single lock covers → serializable isolation (with retries) or an explicit lock row.

### Choosing a level in practice

A workable decision sequence for an OLTP service:

1. **Stay at the engine default** for plain reads and single-row writes — it's what the engine is tuned for and what your team's intuitions were formed on.
2. **Identify every read-modify-write** on contended data (balances, inventory, counters, state machines). Each one gets an explicit mechanism: atomic conditional `UPDATE` where possible, `FOR UPDATE` where not, `@Version` optimistic locking where contention is rare.
3. **Identify multi-row invariants** (the write-skew shape: "at most N", "at least one", uniqueness-across-a-predicate). Each gets serializable isolation on that transaction only, or a designated lock row, or a database constraint that makes the violation impossible.
4. **Never raise the global default as a safety blanket** — serializable-everywhere multiplies aborts and deadlocks across code that didn't need it, and the retry storms arrive exactly at peak load.
5. **Write the retry handler before you need it**: Postgres RR/serializable both abort on conflict by design; an application without retry logic converts the safety mechanism into a user-facing error.

### Setting the level

Isolation is settable per transaction — which is exactly how it should be used (see the decision sequence above):

```sql
-- SQL: this transaction only
BEGIN;
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
...
COMMIT;
```

```java
// Spring: per method, with retries for the abort-on-conflict levels
@Transactional(isolation = Isolation.SERIALIZABLE)
@Retryable(retryFor = CannotSerializeTransactionException.class, maxAttempts = 3)
public void closeRota(...) { ... }
```

A session/global default also exists (`default_transaction_isolation` in Postgres, `transaction_isolation` in MySQL) — treat changing it as an architecture decision, not a hotfix.

## Common misconceptions

- **"We wrap everything in transactions, so we're safe from races."** Transactions at read committed happily permit lost updates and write skew. The wrapper is not the guarantee; the level is.
- **"Repeatable read means my read-modify-write is safe."** In InnoDB, plain reads at RR are snapshot reads with no locks — the lost update proceeds. In Postgres RR you get an abort instead of a silent loss, which is safety only if you actually retry.
- **"MVCC means no locks anywhere."** Writers still lock the rows they write; `FOR UPDATE` exists precisely because snapshots don't arbitrate conflicts.
- **"Serializable = the standard lock-everything mode."** Postgres SSI is optimistic — it aborts instead of blocking. The practical contract of serializable in modern engines is "be ready to retry", not "wait in line".
- **"Isolation levels protect invariants across services."** Nothing here crosses a database boundary. Cross-service invariants live in [[Saga Pattern]] / [[The Outbox Pattern]] territory, and cross-replica read anomalies are a different topic again — [[Read Your Own Writes and Replica Lag Anomalies]].

## In your stack

- MongoDB's defaults sit roughly near read-committed behaviour (reads see committed data; snapshot read concern exists for stronger views) — the lost-update and write-skew reasoning transfers intact to any read-modify-write against Mongo documents. Single-document atomic operators (`$inc`, conditional `findAndModify`) are Mongo's version of the atomic-UPDATE fix.
- JPA/Hibernate: `@Version` optimistic locking for low-contention entities; `@Lock(PESSIMISTIC_WRITE)` → `SELECT ... FOR UPDATE` where money moves. Audit which of your money-mutating paths are "read entity, mutate in Java, flush" — each one is a lost-update candidate.
- Where MySQL exists: remember your level is repeatable read by default, your Postgres-habituated intuitions about read committed don't apply, and gap locks at RR are a real deadlock source under concurrent inserts.
- Balance checks before debits ("does the user have enough?") are check-then-act races at default isolation — the RBI-auditable answer is an atomic conditional update or `FOR UPDATE`, plus a database constraint as backstop, plus [[Idempotency]] on retries.
- Serializable-with-retries pairs badly with services pinned at 1 replica if retries are unbounded under load — cap retries and surface failures rather than melting the pool ([[Connection Pooling]]).

## Interview angle

The 60-second version: *"Four levels. Read uncommitted permits dirty reads — reading data that may be rolled back. Read committed, Postgres's default, stops those but re-reads can change mid-transaction. Repeatable read, MySQL's default, gives a transaction one consistent snapshot via MVCC — but lost updates still need explicit handling: Postgres aborts the second writer, InnoDB requires FOR UPDATE or atomic updates. Serializable stops everything including write skew — where two transactions write different rows but jointly break an invariant — with Postgres using optimistic SSI that aborts conflicting transactions, so you code retries. In practice: default level + SELECT FOR UPDATE or atomic conditional updates on contended rows covers most OLTP."*

Classic probes: the doctors-on-call write-skew story (can you tell it precisely?); "why doesn't repeatable read prevent lost updates in MySQL?"; "what's the difference between a snapshot and a lock?"; design follow-up: make a seat-booking or balance-deduction flow safe, and defend FOR UPDATE vs atomic update vs serializable.

## Check yourself

> [!question]- Q: Explain write skew and why repeatable read doesn't stop it, using the on-call doctors example.
> Invariant: at least one doctor on call. Both A and B, under repeatable read, each see a snapshot with two doctors on call. A sets her own row to off-call; B sets his. The writes touch different rows, so no write-write conflict exists for the engine to detect at RR; both commit; the invariant is broken. Snapshot isolation only detects conflicts on rows both transactions *wrote*. Preventing it needs serializable isolation — which tracks read-write dependencies and aborts one — or an explicit shared lock point, e.g. FOR UPDATE on a rota row both must read.

> [!question]- Q: What happens if two concurrent transactions each run read-balance-then-write-deduction, in Postgres at read committed vs repeatable read vs InnoDB at repeatable read?
> Postgres read committed: both read 100, both write their result, the second commit silently overwrites the first — lost update, balance wrong. Postgres repeatable read: the second committer aborts with a serialization error — the loss is prevented, but only if the application retries (and the retry re-reads the new balance). InnoDB repeatable read: plain reads take no locks, so the lost update happens just like Postgres read committed. In all three, `SELECT ... FOR UPDATE` or a single atomic conditional UPDATE makes it correct.

> [!question]- Q: Why can `SELECT ... FOR UPDATE` deadlock, and what's the standard prevention?
> Two transactions lock rows in opposite orders — A locks row 1 and wants row 2, B locks row 2 and wants row 1; each waits on the other and the engine kills one. Prevention: acquire locks in a globally consistent order (e.g. always lock accounts by ascending id — lock both rows of a transfer in sorted order), keep transactions short, and treat deadlock errors as retryable.

> [!question]- Q: Postgres and MySQL both say "repeatable read", but the same code behaves differently on each. Name the two differences that bite.
> First, lost updates: Postgres RR detects a write to a row concurrently updated since your snapshot and aborts your transaction (retry required); InnoDB RR lets the overwrite through silently — you must add `FOR UPDATE` or an atomic update yourself. Second, phantom/locking behaviour: InnoDB locking reads at RR take next-key (gap) locks that block concurrent inserts into the scanned range — a deadlock source Postgres doesn't have, while Postgres RR prevents phantoms for plain reads via its transaction-wide snapshot. Same level name, different contracts: porting code between them without re-auditing the read-modify-write paths is how the "same" service develops different money bugs per engine.

> [!question]- Q: Mini design: ticket booking — 50 seats, must never oversell, expect bursts of concurrent purchases. Sketch the isolation strategy.
> Make the seat count the serialization point: `UPDATE events SET seats_left = seats_left - 1 WHERE id = ? AND seats_left > 0` and treat zero affected rows as sold out — atomic, no race window, no lock held across app code. Per-seat allocation: `SELECT ... FOR UPDATE` on the chosen seat row (or unique constraint on seat+event as backstop). Avoid serializable-with-retries here: under a burst, aborts and retries amplify load exactly when you can least afford it ([[Timeouts Retries and Exponential Backoff]] reasoning applies inside the database too). Add [[Idempotency]] keys so client retries don't double-book.

## Sources

- *Designing Data-Intensive Applications*, Kleppmann — Chapter 7: the anomaly taxonomy (including write skew and the on-call example) this note follows.
- PostgreSQL documentation: Chapter "Concurrency Control" — MVCC, the three implemented levels, SSI behaviour and retry requirements.
- MySQL 8.0 Reference Manual: "Transaction Isolation Levels", "Consistent Nonlocking Reads", "Locking Reads" — the snapshot-vs-locking-read distinction and next-key locks.
- Berenson et al. (1995), "A Critique of ANSI SQL Isolation Levels" — the paper that named snapshot isolation and showed the standard's definitions were incomplete.
