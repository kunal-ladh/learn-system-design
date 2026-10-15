---
title: Numbers and Formulas
aliases: [Cheat Sheet, Quick Reference, Numbers]
tags: [system-design, foundations, interview]
type: note
status: evergreen
created: 2026-10-02
updated: 2026-10-15
summary: One-page revision sheet — latency tiers, availability math, estimation conventions, quorum and retry formulas. Everything you should be able to recall without looking it up.
related: ["[[System Design Index]]", "[[Practice Drills]]", "[[Latency Numbers Every Engineer Should Know]]", "[[Back of the Envelope Estimation]]"]
---

# Numbers and Formulas

The revision sheet. Everything here is derived in a full note — the links tell you where — but these are the values and formulas you should be able to produce **without looking anything up**, because in an interview or an incident you won't have time to.

Read this the morning of an interview. Everything else in the vault is understanding; this is recall.

---

## Latency tiers

Burn in the **tiers**, not the decimals. Each step is roughly 1–3 orders of magnitude. → [[Latency Numbers Every Engineer Should Know]]

| Operation | Order of magnitude |
|---|---|
| L1 cache reference | ~1 ns |
| Main memory reference | ~100 ns |
| NVMe SSD random 4 KB read | ~20–100 µs |
| Same-datacenter round trip | ~0.5 ms |
| Cross-AZ round trip, same region | ~0.5–2 ms |
| HDD seek | ~10 ms |
| Mumbai ↔ US East round trip | **~200 ms** |

The three that decide designs: **memory is ~5,000× faster than a same-DC network hop (100 ns vs 0.5 ms); a same-DC hop is ~400× faster than crossing the planet; the speed of light is not negotiable.**

Practical lookups, same DC:

| Operation | Rough value |
|---|---|
| Redis GET including network | ~0.2–1 ms |
| Indexed point read, warm MongoDB/MySQL | ~1–5 ms |
| TLS 1.3 handshake, same region | ~1 RTT ≈ 1–5 ms |

> An India-based user calling a US-hosted API pays ~200 ms per round trip **before any server work happens**. A cold HTTPS call needs ~3 round trips ≈ 600 ms — TCP, TLS 1.3, then the request itself (DNS is usually answered by a nearby resolver; TLS 1.2 adds a fourth). This single fact justifies CDNs, regional deployments, and geo-replication.

---

## Availability

### The nines → [[Availability Reliability Durability]]

| Availability | Downtime/year | Downtime/month |
|---|---|---|
| 99.9% (three nines) | 8.76 h | 43.2 min |
| 99.95% | 4.38 h | 21.6 min |
| 99.99% (four nines) | 52.56 min | 4.32 min |
| 99.999% (five nines) | 5.26 min | 25.9 s |

Rule of thumb: **three nines ≈ 9 hours/year** (a bad deploy plus two incidents — where most well-run single-region services live). **Four nines ≈ under an hour/year** — too fast for a human to detect and respond, so it requires automated failover. **Five nines ≈ 5 minutes/year** — no human in the loop at all.

### Composition math

**Series (every component must work) — multiply availabilities:**

```
A_total = A₁ × A₂ × ... × Aₙ
```
> Gateway 99.95% × Service 99.9% × DB 99.9% × Cache 99.95% ≈ **99.7%** ≈ 26 h/year.

Four respectable components chain into something worse than any of them. **This is the quantitative argument against long synchronous microservice chains.**

**Parallel (any one suffices) — multiply unavailabilities:**

```
A_total = 1 − (1 − A)ⁿ
```
> Two independent 99% nodes → 99.99%. Three → 99.9999%.

The word doing the work is **independent**. Same AZ, same deploy, same expiring certificate, same upstream dependency — correlated failures break this math entirely. → [[Failure Modes in Distributed Systems]]

### Error budget → [[SLA SLO SLI and Error Budgets]]

```
error budget = (1 − SLO) × total events in the window
```
> 99.9% SLO over 30 days with 100M requests → 100,000 requests may fail. Burn half in one incident and you have half a month's budget left — that is the number that decides whether you ship or freeze.

---

## Estimation → [[Back of the Envelope Estimation]]

### Rounding conventions

| Thing | Use |
|---|---|
| Seconds in a day | **10⁵** (86,400 rounded — ~16% off, which is noise) |
| Seconds in a month | ~2.5 × 10⁶ |
| 2¹⁰ | ~10³ (KB) |
| 2²⁰ | ~10⁶ (MB) |
| 2³⁰ | ~10⁹ (GB) |
| 2⁴⁰ | ~10¹² (TB) |

### The chain

```
DAU × actions/user/day            = actions/day
actions/day ÷ 10⁵                 = average RPS
average RPS × peak factor         = peak RPS        (2–3× smooth, 5–10× spiky)
writes/day × size × retention × 3 = storage         (3× for replication)
RPS × payload                     = bandwidth       (egress usually dominates)
```

Two anchors worth memorizing: **1M DAU doing 10 actions/day ≈ 100 RPS average**, and **1 KB × 1M/day ≈ 1 GB/day ≈ 365 GB/year**.

### Sanity anchors

| Claim | Reality check |
|---|---|
| One modest Postgres box | comfortably thousands of simple TPS |
| One Redis instance | ~10⁵ ops/sec with I/O threads off; Redis reports >1M ops/sec on 8.2 with I/O threads (2025) — command execution stays single-threaded |
| One Kafka broker | ~10⁵–10⁶ msgs/sec with batching |
| Working set fits in RAM? | the single most important question for read latency |

If your estimate says 100 RPS, **say out loud that one database handles this** — proposing Kafka for 100 RPS scores worse than not estimating at all.

---

## Distributed systems formulas

### Quorum → [[Quorum Reads and Writes]]

```
R + W > N   → read and write sets overlap
W > N/2     → no two writes commit without overlapping
```
With **N=3**: `W=2, R=2` is the standard balanced choice. `W=3, R=1` gives fast reads but no write availability if any replica is down. `W=1, R=1` is fast and offers no overlap guarantee at all.

Overlap guarantees you *see* a recent write — it does **not** give you linearizability on its own.

### Consensus

A cluster of **2f + 1** nodes tolerates **f** failures. So 3 nodes tolerate 1; 5 tolerate 2. This is why consensus clusters (etcd, ZooKeeper, Kafka KRaft controllers) are **odd-numbered** — a 4th node adds cost and tolerates no more failures than 3. → [[Consensus and Raft]]

### Retry amplification → [[Timeouts Retries and Exponential Backoff]]

```
load multiplier = (attempts per layer) ^ (number of layers)
```
**Restate every policy in _attempts_ before multiplying** — this is the step teams get wrong. Libraries count differently: Resilience4j's `maxAttempts: 3` is one call plus two retries, while Spring Framework 7's built-in `@Retryable` counts *retries* — `maxRetries` defaults to 3, which is four attempts. Calling it "2 retries" and computing 2×2 reports 4×; the dependency actually sees 3×3 = 9.

> 3 layers at 3 attempts each → **27×** the original load. If each layer does 3 *retries* (4 attempts), it is 4³ = **64×** — arriving exactly when the system is already failing.
>
> At 500 RPS against a database provisioned for 2,000 qps: 500 × 27 = 13,500 qps offered, **6.75× over capacity**, entirely manufactured by its own clients. When the original fault clears, nothing improves — someone has to turn the retries off.

Backoff with full jitter:
```
sleep = random(0, min(cap, base × 2^attempt))
```
The **jitter is the point** — synchronized retries are a thundering herd regardless of how long the delay is.

### Little's Law → [[Capacity Planning]]

```
L = λ × W        concurrency = arrival rate × time in system
```
> 500 RPS × 200 ms = 100 in-flight requests. If your thread pool is 50, you are the bottleneck and the queue grows without bound. This is how you size pools and spot saturation.

---

## Caching → [[Why Caching Works]]

```
effective latency = hit% × cache_latency + miss% × (cache_latency + origin_latency)
```
> 90% hits at 1 ms, misses at 50 ms → 0.9(1) + 0.1(51) = **6 ms**, down from 50 ms.

The non-obvious half: going 90% → 99% hit rate cuts the miss traffic to your database **by 10×**. The latency gain looks modest; the load relief is the real prize.

---

## Decision one-liners

Recall-level answers. Each links to the note that earns it.

| Question | Short answer |
|---|---|
| Queue or log? | Need replay or multiple independent consumers → **log**. Need per-message routing and acks → **queue**. → [[Message Queues vs Event Logs]] |
| Shard key? | High cardinality, even access, matches your dominant query. → [[Sharding]] |
| One shard melting? | Hot partition — adding shards does **not** help. → [[Hot Partition Problem]] |
| SQL or NoSQL? | Access pattern and consistency needs decide, not scale alone. → [[Choosing a Database]] |
| Microservices? | An org answer before a technical one. → [[Monolith vs Microservices]] |
| Exactly-once? | Doesn't exist in delivery — at-least-once + idempotent consumer. → [[Exactly-Once Delivery Is a Lie]] |
| Retry safely? | Only if idempotent, only with backoff **and** jitter. → [[Idempotency]] |
| CAP choice? | During a partition only; the rest of the time it's latency vs consistency. → [[PACELC]] |
| Cache strategy? | Cache-aside unless you have a specific reason. → [[Caching Strategies]] |
| Dual-write to DB and broker? | You can't — use the outbox. → [[The Outbox Pattern]] |

---

## Check yourself

> [!question]- Without scrolling up: how many seconds in a day do you use for estimation, and why that number?
> **10⁵** (100,000). The true value is 86,400, so you are ~16% high — irrelevant when the goal is order of magnitude, and it makes division trivial in your head.

> [!question]- A request passes through 5 services each at 99.9%. What is the end-to-end availability, and what does that imply?
> 0.999⁵ ≈ **99.5%**, or about 43 hours of downtime a year. Each synchronous hop taxes availability multiplicatively, which is the quantitative case for shortening critical paths and moving work off them asynchronously.

> [!question]- Your service does 2,000 RPS with 150 ms average latency. How many requests are in flight, and what does that tell you about pool sizing?
> Little's Law: 2,000 × 0.15 = **300 concurrent requests**. Any pool (threads, connections) below 300 makes *you* the bottleneck rather than the downstream dependency — and queueing will show up as latency you may misattribute to the database.

> [!question]- Why is going from a 90% to a 99% cache hit rate worth far more than the latency number suggests?
> Latency improves modestly, but misses drop from 10% to 1% of traffic — a **10× reduction in load reaching the origin**. Cache improvements are best measured in database load removed, not milliseconds saved.

## Sources

Every number here is derived and sourced in its parent note: [[Latency Numbers Every Engineer Should Know]], [[Availability Reliability Durability]], [[SLA SLO SLI and Error Budgets]], [[Back of the Envelope Estimation]], [[Quorum Reads and Writes]], [[Timeouts Retries and Exponential Backoff]], [[Capacity Planning]], [[Why Caching Works]].
