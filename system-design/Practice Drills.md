---
title: Practice Drills
aliases: [Practice, Drills, Questions]
tags: [moc, system-design, interview]
type: moc
status: evergreen
created: 2026-10-01
updated: 2026-10-01
summary: Structured practice — retrieval questions per module, graded design problems, and a 10-week schedule.
related: ["[[System Design Index]]", "[[The System Design Interview Framework]]"]
---

# Practice Drills

Reading creates the feeling of knowing; retrieval creates knowing. This page is the retrieval half of the vault. Every question here is answerable from a note — the pointer tells you which one, but **look only after you've attempted an answer out loud**. An answer you had to dig for teaches more than one you read.

## How to practice

The loop that works, per topic:

1. **Read** the note once, normally.
2. **Close it.** Redraw its main diagram from memory in Excalidraw. Gaps in your drawing are gaps in your understanding — they're the point of the exercise.
3. **Answer the note's own `Check yourself` questions** out loud, in full sentences, before expanding the collapsed answers.
4. **Come here** and answer this module's rapid-fire questions cold, a day or more later. Spacing is what moves it to long-term memory.
5. **Once per week**, do one full design problem from the tiers below on a blank page, timed.

For the numeric questions, [[Numbers and Formulas]] is the recall sheet those answers come from — use it to check yourself, not to study from first.

Say answers *out loud*. The gap between "I know this" and "I can say this" is exactly the gap that shows up in an interview and in a design review.

---

## Part 1 — Rapid-fire retrieval

Answer in 2–3 sentences each. If you can't, that module needs a re-read.

### Foundations & networking
1. What's the difference between latency, throughput, and bandwidth? → [[Latency vs Throughput vs Bandwidth]]
2. Your p50 is 20ms and your p99 is 900ms. What does that tell you that an average wouldn't? → [[Latency vs Throughput vs Bandwidth]]
3. Roughly how much slower is a disk seek than a main-memory read? A cross-region round trip than a same-datacenter one? → [[Latency Numbers Every Engineer Should Know]]
4. A service with 99.9% availability depends on four others, each also 99.9%. What's the realistic combined availability? → [[Availability Reliability Durability]]
5. Why is S3 advertised with 11 nines of *durability* but only four nines of *availability*? → [[Availability Reliability Durability]]
6. Estimate the storage for 10 million users posting 5 photos a day for a year. Show the arithmetic. → [[Back of the Envelope Estimation]]
7. Why does HTTP/3 use UDP when TCP already guarantees delivery? → [[HTTP 1.1 vs HTTP 2 vs HTTP 3]]
8. "DNS propagation is slow." Why is that phrasing wrong? → [[How DNS Works]]
9. Why is a *slow* dependency more dangerous than a *dead* one? → [[Network Partitions and Timeouts]]

### Caching & Redis
10. Walk through cache-aside on a read miss and on a write. Where exactly can it serve stale data? → [[Caching Strategies]]
11. A popular key expires and 5,000 requests hit the database in the same second. Name this, and give two independent fixes. → [[Cache Stampede and Thundering Herd]]
12. Redis is single-threaded. Why is it still fast, and what does that imply about running `KEYS *` in production? → [[Redis Internals]]
13. What is a hot key, and why doesn't adding more Redis nodes fix it? → [[Hot Key Problem]]
14. Why can't you run `MGET` across arbitrary keys in Redis Cluster, and what's the workaround? → [[Redis Cluster and Sentinel]]
15. What does Redis Sentinel actually decide, and what can you still lose during a failover? → [[Redis Cluster and Sentinel]]
16. Why is a Redis-based distributed lock not safe on its own, and what does a fencing token add? → [[Distributed Locking]]

### Databases, replication & sharding
17. What does each letter of ACID actually guarantee — and what does ACID say about two services? → [[ACID Transactions]]
18. What anomaly does repeatable read prevent that read committed doesn't? → [[Transaction Isolation Levels]]
19. Why does adding an index make writes slower? → [[Database Indexes]]
20. Why is a bigger connection pool often *slower*? → [[Connection Pooling]]
21. Explain sharding to someone who has only ever used one database. → [[Sharding]]
22. Why does `hash(key) % N` break when you add a node, and how does the hash ring fix it? → [[Consistent Hashing]]
23. You shard by `created_at`. What goes wrong, and what's the name for it? → [[Hot Partition Problem]]
24. A user posts, then refreshes and their post is gone. What happened and how do you fix it? → [[Read Your Own Writes and Replica Lag Anomalies]]
25. Facebook runs on MySQL. So what did "MySQL doesn't scale" ever mean? → [[Why MySQL Does Scale]]
26. When do you choose an LSM-tree over a B-tree, and what do you pay for it? → [[B-Tree vs LSM-Tree]]

### Distributed systems theory
27. State CAP precisely. Why is "pick 2 of 3" a misleading summary? → [[CAP Theorem]]
28. What does PACELC explain that CAP doesn't? → [[PACELC]]
29. Give a two-client example that's eventually consistent but not causally consistent. → [[Consistency Models]]
30. Why can't you have exactly-once *delivery*, and what can you have instead? → [[Exactly-Once Delivery Is a Lie]]
31. With N=3, which (R, W) pairs give you overlap, and what does overlap buy you? → [[Quorum Reads and Writes]]
32. Why does two-phase commit block, and what does the industry do instead? → [[Two-Phase Commit and Distributed Transactions]]
33. Design an idempotent "transfer money" endpoint. What's the key, where is it stored, for how long? → [[Idempotency]]
34. A leader is GC-paused for 30 seconds while a new leader is elected. What breaks, and how does fencing save you? → [[Split Brain and Fencing]]

### Messaging & event-driven
35. What's the fundamental difference between a queue and a log, and why does it decide your broker? → [[Message Queues vs Event Logs]]
36. Your consumer processes a message, then crashes before acking. What happens next, and what must your consumer have been? → [[Delivery Semantics]]
37. Why does Kafka guarantee ordering only per partition and not per topic? → [[Message Ordering]]
38. Name three things that silently break ordering even when the broker preserves it. → [[Message Ordering]]
39. You write to MongoDB and then publish to SQS. Describe both failure orderings and the pattern that fixes them. → [[The Outbox Pattern]]
40. Your SQS visibility timeout is 30s and processing takes 45s. What happens? → [[SQS and SNS]]
41. What exactly does Kafka's "exactly-once semantics" cover, and what does it not? → [[Kafka Architecture]]
42. You're moving from SQS to Redis Streams. What are the three things you must verify first? → [[Redis Streams]]
43. A payment fails at step 3 of 4 after the irreversible rail call succeeded. What pattern handles this, and what is a compensation *not*? → [[Saga Pattern]]
44. A consumer replays a 6-month-old partition and crashes on deserialization. What went wrong months earlier? → [[Event Schema Evolution]]

### Architecture, auth & resilience
45. When are microservices the *wrong* choice? Answer in terms of org, not technology. → [[Monolith vs Microservices]]
46. What's a distributed monolith, and what's the giveaway symptom? → [[The Distributed Monolith Anti-Pattern]]
47. Walk a JWT from issuance to verification. What exactly does a service check, and in what order? → [[How JWT Works]]
48. Why RS256 over HS256 across microservices? → [[How JWT Works]]
49. How do you log a user out *now* when verification is stateless? → [[JWT Pitfalls]]
50. Your IAM service is in every request path. What happens when it's down, and how do you survive it? → [[Designing an IAM Service]]
51. Three layers each retry 3 times. How many requests does the bottom service see, and what's this called? → [[Timeouts Retries and Exponential Backoff]]
52. Walk the three circuit-breaker states and what moves between them. → [[Circuit Breakers]]
53. Why does Netflix kill instances in production, during business hours, on purpose? → [[Chaos Engineering]]
54. Define RPO and RTO using a concrete fintech example. → [[Disaster Recovery]]
55. You run one replica. What does a PodDisruptionBudget actually protect, honestly? → [[Graceful Shutdown and Deployment Safety]]

---

## Part 2 — Design problems, by tier

Do these on a blank page or whiteboard, timed, using the 4-phase method in [[The System Design Interview Framework]]. **Write before you read the note** — then compare, and note what you missed. What you missed is your real syllabus.

### Tier 1 — build the reflex (45 min each)
These have clean, well-understood answers. The goal is fluency in the *method*: requirements → estimation → high level → deep dive.

| Problem | What it really tests |
|---|---|
| [[Design a URL Shortener]] | ID generation, read-heavy caching, the redirect path |
| [[Design a Rate Limiter]] | Algorithm trade-offs, distributed state, atomicity |
| [[Design a Distributed ID Generator]] | Why auto-increment fails, sortability, clock handling |
| [[Design Typeahead Autocomplete]] | Precomputation vs freshness, prefix sharding |

### Tier 2 — the common rounds (45–60 min each)
The problems most often asked at senior level. Each has a dominant hard decision — find it before designing.

| Problem | The dominant decision |
|---|---|
| [[Design a News Feed]] | Fan-out on write vs read, and the celebrity exception |
| [[Design a Chat System]] | Connection state, per-conversation ordering, offline sync |
| [[Design a Notification Service]] | Dedup, retries, provider failover — closest to your day job |
| [[Design a Web Crawler]] | Politeness, dedup at billions scale, frontier partitioning |
| [[Design Distributed File Storage]] | Chunking, dedup, metadata as the consistency-critical core |
| [[Design a Video Streaming Service]] | Async transcoding pipeline, CDN economics |

### Tier 3 — the hard ones (60+ min each)
Deeper constraints: money correctness, microsecond latency, or building infrastructure itself.

| Problem | Why it's hard |
|---|---|
| [[Design a Ride Sharing Service]] | Geo-sharding, matching without double-dispatch |
| [[Design a Payment System]] | Ledgers, idempotency, reconciliation — your home turf, so go deepest here |
| [[Design a Stock Exchange]] | Determinism and fairness at microsecond latency |
| [[Design a Distributed Cache]] | Build Redis Cluster from first principles |
| [[Design a Key-Value Store]] | **The capstone** — uses nearly every note in the vault |

---

## Part 3 — A 10-week schedule

Assumes about an hour on weekdays and one longer weekend session. Compress or stretch freely; the sequence matters more than the pace.

| Week | Read | Drill |
|---|---|---|
| 1 | Modules 1–2 (Foundations, Networking) | Q1–9; estimation practice until the arithmetic is automatic |
| 2 | Modules 3–4 (Traffic, Caching & Redis) | Q10–16; [[Design a URL Shortener]], [[Design a Rate Limiter]] |
| 3 | Module 5 (Relational DBs) | Q17–20; run `EXPLAIN` on three real queries at work |
| 4 | Module 6 (Replication & Sharding) | Q21–25; [[Design a Distributed ID Generator]] |
| 5 | Module 7 (NoSQL & Storage Engines) | Q26; [[Design Typeahead Autocomplete]] |
| 6 | Module 8 (Distributed Theory) — slowest module, don't rush | Q27–34 |
| 7 | Module 9 (Messaging) — your deepest module | Q35–44; [[Design a Notification Service]] |
| 8 | Modules 10–11 (Architecture, Auth & IAM) | Q45–50; [[Design a Chat System]] |
| 9 | Modules 12–13 (Resilience, Observability) | Q51–55; [[Design a News Feed]] |
| 10 | Module 14 (Case Studies) + Module 15 review | [[Design a Payment System]], then [[Design a Key-Value Store]] as the final exam |

## Self-assessment rubric

After each design problem, score yourself honestly:

- **Did I clarify requirements before designing?** Jumping straight to boxes is the single most common failure.
- **Did I quantify anything?** A design with no numbers in it is an opinion.
- **Did I name trade-offs, or just make choices?** "I chose X" is weak; "I chose X over Y because we're read-heavy and can tolerate staleness" is the answer.
- **Did I identify the one hard part?** Every problem has a dominant difficulty. Finding it is most of the signal.
- **Did I say how it fails?** Failure modes separate people who've run systems from people who've only drawn them.

If you miss the same criterion across three problems, that's not a knowledge gap — it's a habit gap. Fix it deliberately in the next one.
