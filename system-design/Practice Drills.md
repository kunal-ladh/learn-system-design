---
title: Practice Drills
aliases: [Practice, Drills, Questions]
tags: [moc, system-design, interview]
type: moc
status: evergreen
created: 2026-10-01
updated: 2026-10-01
summary: Structured practice — retrieval questions per module, graded design problems, and an 11-week schedule.
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
3. How much slower is an HDD seek than a main-memory read — and an NVMe random read? A cross-region round trip than a same-DC one? → [[Latency Numbers Every Engineer Should Know]]
4. A request passes through five services, each 99.9% available, all in series on the request path. What's the combined availability — and why is the real number usually worse? → [[Availability Reliability Durability]]
5. S3 is designed for 11 nines of *durability* but 99.99% *availability*, with only a 99.9% SLA. Why do those three numbers differ so much? → [[Availability Reliability Durability]]
6. 10M daily active users each post 5 photos/day at ~2 MB, stored with 3× replication. Estimate one year of storage — show the arithmetic. → [[Back of the Envelope Estimation]]
7. Why does HTTP/3 use UDP when TCP already guarantees delivery? → [[HTTP 1.1 vs HTTP 2 vs HTTP 3]]
8. "DNS propagation is slow." Why is that phrasing wrong? → [[How DNS Works]]
9. Why is a *slow* dependency more dangerous than a *dead* one? → [[Network Partitions and Timeouts]]

### Traffic management
10. A load balancer at L4 vs L7 — what can each one see, and what can only an L7 balancer do? → [[Load Balancers]]
11. Token bucket vs leaky bucket: which one allows bursts, and what does each one's parameter actually mean? → [[Rate Limiting]]
12. A fixed-window rate limiter of 100/min can let through nearly 200 requests in 60 seconds. How, and which algorithm fixes it? → [[Rate Limiting]]
13. Liveness vs readiness vs startup probes — what does Kubernetes do when each one fails, and why must readiness never check a shared database at one replica? → [[Health Checks and Service Discovery]]
14. Your gateway timeout is 3 s and it calls a service whose own downstream timeout is 4 s. What happens on a slow dependency, and what's the rule? → [[Network Partitions and Timeouts]]

### Caching & Redis
15. Walk through cache-aside on a read miss and on a write. Where exactly can it serve stale data? → [[Caching Strategies]]
16. A popular key expires and 5,000 requests hit the database in the same second. Name this, and give two independent fixes. → [[Cache Stampede and Thundering Herd]]
17. Redis is single-threaded. Why is it still fast, and what does that imply about running `KEYS *` in production? → [[Redis Internals]]
18. What is a hot key, and why doesn't adding more Redis nodes fix it? → [[Hot Key Problem]]
19. Why can't you run `MGET` across arbitrary keys in Redis Cluster, and what's the workaround? → [[Redis Cluster and Sentinel]]
20. In Redis Sentinel, what does `quorum` decide versus the majority vote — and what can you still lose during a failover? → [[Redis Cluster and Sentinel]]
21. Why is a Redis-based distributed lock not safe on its own, and what does a fencing token add? → [[Distributed Locking]]
22. If indexes already made reads cheap, what does a cache remove that an index can't? → [[Why Caching Works]]
23. Redis is told to evict with `allkeys-lru`. What does "approximated" LRU mean in practice, and when would you pick LFU instead? → [[Cache Eviction Policies]]
24. You invalidate a cache key by deleting it on write. Name the race that still serves stale data, and one invalidation strategy that closes it. → [[Cache Invalidation]]
25. Redis pub/sub vs Redis Streams: what happens to a message when no consumer is connected, in each? → [[Redis Beyond Caching]]

### Databases, replication & sharding
26. What does each letter of ACID actually guarantee — and what does ACID say about two services? → [[ACID Transactions]]
27. Per the SQL-standard table, what anomaly does repeatable read prevent that read committed doesn't — and what *extra* does Postgres's RR prevent that InnoDB's doesn't? → [[Transaction Isolation Levels]]
28. Why does adding an index make writes slower? → [[Database Indexes]]
29. Why is a bigger connection pool often *slower*? → [[Connection Pooling]]
30. Explain sharding to someone who has only ever used one database. → [[Sharding]]
31. Why does `hash(key) % N` break when you add a node, and how does the hash ring fix it? → [[Consistent Hashing]]
32. You *range*-shard transactions by `created_at`. What goes wrong, and what's the name for it? → [[Hot Partition Problem]]
33. A user posts, then refreshes and their post is gone. What happened and how do you fix it? → [[Read Your Own Writes and Replica Lag Anomalies]]
34. Facebook runs on MySQL. So what did "MySQL doesn't scale" ever mean? → [[Why MySQL Does Scale]]
35. When do you choose an LSM-tree over a B-tree, and what do you pay for it? → [[B-Tree vs LSM-Tree]]
36. A query stopped using its index after a seemingly harmless change. Name three ways a predicate becomes non-sargable, and what you'd run to confirm. → [[Query Planning and EXPLAIN]]
37. Why does a database write to a log *before* updating the page, when it has to do both anyway? → [[How Databases Store Data]]
38. You denormalise a user's name onto 50 million order rows for read speed. What did you just sign up to maintain, and how? → [[Normalization vs Denormalization]]
39. Walk the scaling ladder in order. Why is sharding last, and what does skipping a rung cost you? → [[Database Scaling Patterns]]
40. One partition key receives 5,000 writes/sec while the table as a whole has spare capacity. Why can't DynamoDB's adaptive capacity rescue it? → [[DynamoDB Design]]
41. MongoDB `w:1` vs `w:majority` — what exactly can you lose on a failover with each? → [[MongoDB Architecture]]
42. What does Spanner's commit-wait buy, and roughly how long is it? → [[Bigtable and Spanner]]
43. An async leader acknowledges a write and dies before any follower received it. What is lost, and which replication setting prevents it? → [[Failover]]
44. Why is two-node high availability a trap? → [[Failover]]
45. Users are sharded by `user_id`. How do you serve login-by-email, and what does each option cost? → [[Secondary Indexes on Sharded Data]]
46. Why does `hash mod N` make adding one shard catastrophic, and what are the alternatives? → [[Rebalancing Partitions]]

### Distributed systems theory
47. State CAP precisely. Why is "pick 2 of 3" a misleading summary? → [[CAP Theorem]]
48. What does PACELC explain that CAP doesn't? → [[PACELC]]
49. Give a two-client example that's eventually consistent but not causally consistent. → [[Consistency Models]]
50. Why can't you have exactly-once *delivery*, and what can you have instead? → [[Exactly-Once Delivery Is a Lie]]
51. With N=3, which (R, W) pairs satisfy R+W>N, and what does that overlap buy you — and not buy you? → [[Quorum Reads and Writes]]
52. Why does two-phase commit block, and what does the industry do instead? → [[Two-Phase Commit and Distributed Transactions]]
53. Design an idempotent "transfer money" endpoint. What's the key, where is it stored, for how long? → [[Idempotency]]
54. A leader is GC-paused for longer than its 30-second lease while a new leader is elected. What breaks, and how does a fencing token save you? → [[Split Brain and Fencing]]
55. A remote call times out. What are the possibilities on the other side — request lost, executed but response lost, or still running — and why does that decide whether you may retry? → [[The Fallacies of Distributed Computing]]
56. What can a vector clock tell you that a Lamport clock can't? → [[Logical Clocks and Ordering]]
57. Why do Raft clusters have odd sizes, and what happens to a 3-node cluster split 2/1? → [[Consensus and Raft]]

### Messaging & event-driven
58. What's the fundamental difference between a queue and a log, and why does it decide your broker? → [[Message Queues vs Event Logs]]
59. Your consumer processes a message, then crashes before acking. What happens next, and what must your consumer have been? → [[Delivery Semantics]]
60. Why does Kafka guarantee ordering only per partition and not per topic? → [[Message Ordering]]
61. Name three things that silently break ordering even when the broker preserves it. → [[Message Ordering]]
62. A service must write to MongoDB and publish to SQS for one action. Describe what fails under each of the two possible orderings, and the pattern that fixes both. → [[The Outbox Pattern]]
63. Your SQS visibility timeout is 30s and processing takes 45s. What happens? → [[SQS and SNS]]
64. What exactly does Kafka's "exactly-once semantics" cover, and what does it not? → [[Kafka Architecture]]
65. What must you verify, workload by workload, before moving SQS queues to Redis Streams? Name at least three of the checks. → [[Redis Streams]]
66. In a five-step transfer saga, the rail call at step 3 (the pivot) succeeds and the ledger post at step 4 fails. What does the saga do, and what is a compensation *not*? → [[Saga Pattern]]
67. A consumer replays a 6-month-old partition and crashes on deserialization. What went wrong months earlier? → [[Event Schema Evolution]]
68. Why does a synchronous chain of five services have worse availability than any one of them, and what does putting a queue between them change? → [[Why Async Messaging Exists]]
69. A message fails processing 20 times in a row. What should happen to it, and why is a DLQ nobody drains worse than having none? → [[Dead Letter Queues and Poison Messages]]
70. Consumer lag is growing. Name three ways to shed or absorb it, and the one metric you alert on. → [[Backpressure and Flow Control]]
71. A Kafka topic has 6 partitions and you run 8 consumers in one group. What happens to the extra two? → [[Consumer Groups and Rebalancing]]
72. Mongo change streams need a replica set, and a delete event carries only the `_id`. What does that force your CDC consumer to do? → [[Change Data Capture]]
73. When is event sourcing the wrong call even though you genuinely need an audit trail? → [[Event Sourcing]]
74. CQRS *without* event sourcing — what does that look like, and what consistency do you give up? → [[CQRS]]

### Architecture, auth, resilience & observability
75. When are microservices the *wrong* choice? Answer in terms of org, not technology. → [[Monolith vs Microservices]]
76. What's a distributed monolith, and which two measurements would you take first to prove you have one? → [[The Distributed Monolith Anti-Pattern]]
77. Walk a JWT from issuance to verification. What exactly does a service check, and in what order? → [[How JWT Works]]
78. Why RS256 over HS256 across microservices? → [[How JWT Works]]
79. How do you log a user out *now* when verification is stateless? → [[JWT Pitfalls]]
80. Your IAM service sits in the login, refresh and JWKS paths but *not* the per-request path. What breaks when it's down for 20 minutes, on what timeline, and how do you survive it? → [[Designing an IAM Service]]
81. Three layers each make 3 *attempts* per call (1 original + 2 retries). How many requests can the bottom service see for one user click, what's this called — and what changes if someone reads the policy as "3 retries"? → [[Timeouts Retries and Exponential Backoff]]
82. Walk the three circuit-breaker states and what moves between them. → [[Circuit Breakers]]
83. Why did Netflix build Chaos Monkey to kill production instances during business hours, on purpose? → [[Chaos Engineering]]
84. Define RPO and RTO using a concrete fintech example. → [[Disaster Recovery]]
85. You run one replica. What does a PodDisruptionBudget actually protect, honestly? → [[Graceful Shutdown and Deployment Safety]]
86. Your p99 latency alert fires but the average looks fine. Which of logs, metrics or traces answers *where* the time went, and why can't the other two? → [[Logs Metrics and Traces]], [[Distributed Tracing]]
87. An SLO burn-rate alert pages at 14.4× over a 1-hour window. What does that number mean, and why pair a short window with a long one? → [[Alerting and On-Call Design]]
88. A canary gets 5% of traffic and its error rate looks fine. What could still be wrong with that conclusion, and what would you compare instead? → [[Deployment Strategies]]

89. Why does offset pagination degrade as you page deeper, and what replaces it? → [[API Design and Versioning]]
90. Before native sidecar containers, why did a Kubernetes Job with a sidecar never complete? → [[Sidecar Pattern]]
91. Mesh vs gateway: which concern belongs where — and what happens when the app's timeout and the mesh's per-try timeout disagree? → [[Service Mesh]]
92. In a strangler-fig migration, name the three data stages — and at which one can you no longer undo cheaply? → [[Strangler Fig Migration]]
93. In an orchestrated saga, why is a timeout at the pivot step not a failure? → [[Orchestration vs Choreography]]
94. Two services own different tables. Give two ways to answer a query that needs a join across them, and the cost of each. → [[Database per Service]]
95. BFF vs API gateway vs GraphQL — which solves over-fetching, which solves cross-cutting policy, and can you run all three? → [[Backend for Frontend]]
96. Head vs tail sampling in tracing: which one can keep *every* slow trace, and what does it cost? → [[Distributed Tracing]]
97. Why is CPU the wrong HPA signal for an IO-bound Java service, and what would you scale on instead? → [[Autoscaling]]
98. Your service does 1,500 RPS at 200 ms. How many requests are in flight, and what does a thread pool of 100 do to you? → [[Capacity Planning]]

### AI infrastructure (bonus)
99. How much RAM for 10M 1536-dimension float32 vectors in HNSW, and why doesn't a B-tree help? → [[Vector Databases and Similarity Search]]
100. A source document is deleted but the delete event is lost. What does the RAG system now serve, and how do you detect it? → [[RAG Architecture]]
101. Why rate-limit an LLM endpoint by tokens rather than requests, and what would you autoscale on? → [[LLM Serving Infrastructure]]

### Shipping code & AWS
102. Walk one commit from `git push` to serving traffic on EKS. Name every system it passes through and what each one guarantees. → [[The Deployment Pipeline]]
103. A container is not a lightweight VM. Which two kernel features make it a process with walls, and what does "shares one kernel" imply for isolation? → [[Docker and Container Images]]
104. Why does Jib split a Java app into separate dependency, resource and class layers — and what does a code-only change actually push? → [[Jib]]
105. Prod deploys `myservice:latest`. Describe the rollback that silently deploys a *different* artefact, and the registry setting that prevents it. → [[Container Registries and ECR]]
106. You run `kubectl apply` on a Deployment. Walk what the API server, etcd, scheduler, controller-manager and kubelet each do, in order. → [[Kubernetes Architecture]]
107. Deployment → ReplicaSet → Pod: what does a rollout actually create, and what is `kubectl rollout undo` really doing? → [[Kubernetes Workloads]]
108. A Service has a ClusterIP but zero endpoints. Name two causes — and explain why long-lived gRPC connections pin to one pod even with a Service in front. → [[Kubernetes Networking]]
109. Memory limit vs CPU limit: one throttles, one kills. Which is which, and why does that make JVM heap sizing inside a container a trap? → [[Kubernetes Configuration and Resources]]
110. ECR, ECS, EKS — which one is the registry, and when would a small team be right to pick ECS over EKS? → [[EKS vs ECS]]
111. A `t3` instance ran fine for weeks and now crawls at 20% CPU. What happened, and what are the two fixes? → [[EC2]]
112. A pod in a private subnet needs to read from S3. Trace how it gets permission with no static key anywhere, and name the two mechanisms. → [[IAM]]
113. Private-subnet services call S3 heavily and the bill spikes. Which component is charging per GB, and what removes it? → [[VPC and Networking]]
114. An ALB in front of Kong starts returning 502s on long-running requests. Name the timeout mismatch and the direction the chain must satisfy. → [[ALB and NLB]]
115. Name the four cost shapes that dominate an AWS bill — and the one that is free in one direction and billed in the other. → [[AWS Cost Shape]]
116. An SQS-triggered Lambda has a 60 s timeout and the queue's visibility timeout is also 60 s. Some orders get processed twice. Why, and what should the visibility timeout be? → [[Lambda]]

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

## Part 3 — An 11-week schedule

Assumes about an hour on weekdays and one longer weekend session. Compress or stretch freely; the sequence matters more than the pace.

| Week | Read | Drill |
|---|---|---|
| 1 | Modules 1–2 (Foundations, Networking) | Q1–9, Q14; estimation practice until the arithmetic is automatic |
| 2 | Modules 3–4 (Traffic, Caching & Redis) | Q10–13, Q15–25; [[Design a URL Shortener]], [[Design a Rate Limiter]] |
| 3 | Module 5 (Relational DBs) | Q26–29, Q36–39; run `EXPLAIN` on three real queries at work |
| 4 | Module 6 (Replication & Sharding) | Q30–34, Q43–46; [[Design a Distributed ID Generator]] |
| 5 | Module 7 (NoSQL & Storage Engines) | Q35, Q40–42; [[Design Typeahead Autocomplete]] |
| 6 | Module 8 (Distributed Theory) — slowest module, don't rush | Q47–57|
| 7 | Module 9 (Messaging) — your deepest module | Q58–74; [[Design a Notification Service]] |
| 8 | Modules 10–11 (Architecture, Auth & IAM) | Q75–80, Q89–95; [[Design a Chat System]] |
| 9 | Modules 12–13 (Resilience, Observability) | Q81–88, Q96–98; [[Design a News Feed]] |
| 10 | Module 14 (Case Studies) + Module 15 review | [[Design a Payment System]], then [[Design a Key-Value Store]] as the final exam |
| 11 | Modules 17–18 (Shipping Code, AWS) | Q102–116; draw your own deploy pipeline from memory, then compare with [[The Deployment Pipeline]] |

## Self-assessment rubric

After each design problem, score yourself honestly:

- **Did I clarify requirements before designing?** Jumping straight to boxes is the single most common failure.
- **Did I quantify anything?** A design with no numbers in it is an opinion.
- **Did I name trade-offs, or just make choices?** "I chose X" is weak; "I chose X over Y because we're read-heavy and can tolerate staleness" is the answer.
- **Did I identify the one hard part?** Every problem has a dominant difficulty. Finding it is most of the signal.
- **Did I say how it fails?** Failure modes separate people who've run systems from people who've only drawn them.

If you miss the same criterion across three problems, that's not a knowledge gap — it's a habit gap. Fix it deliberately in the next one.
