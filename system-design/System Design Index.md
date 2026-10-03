---
title: System Design Index
aliases: [System Design MOC, SD Index, Home]
tags: [moc, system-design]
type: moc
status: evergreen
created: 2026-10-01
updated: 2026-10-01
summary: Master map of content for the complete system design curriculum — start here.
related: ["[[Practice Drills]]"]
---

# System Design — Master Index

> **Start here.** This is the home MOC for the entire curriculum. Every topic below is a note (or will become one). Unresolved links are topics not yet written — they are the roadmap, not errors.

## How to use this vault

1. **Go in order the first time.** Modules build on each other: you can't reason about Kafka partitioning (Module 9) without sharding (Module 6), and you can't reason about sharding without replication and latency basics (Modules 1, 6).
2. **Every component note follows the same skeleton** — *the problem that created it → what it solves → how it works → trade-offs → failure modes → when NOT to use it → in your stack → interview angle*. If you remember nothing else from a note, remember the problem it was invented to solve; everything else re-derives from that.
3. **Draw it yourself.** Mermaid diagrams are embedded where precision matters; for intuition, redraw the architecture in Excalidraw from memory after reading. If you can't redraw it, you haven't learned it.
4. **Link as you learn.** When a note reminds you of something at work (a Niyo incident, a config you've seen), add your own line under *In your stack*. Notes you've annotated are notes you'll remember.

## Study tracks

- **Work track** (event-driven systems, Redis, auth): Modules 1 → 4 → 8 → 9 → 11 → 12
- **Interview track** (8–10 weeks): all modules in order, then drill Module 15 problems twice
- **"I have one weekend"**: [[Back of the Envelope Estimation]], [[Latency Numbers Every Engineer Should Know]], [[Replication]], [[Sharding]], [[Caching Strategies]], [[Message Queues vs Event Logs]], [[The System Design Interview Framework]]

---

## Module 1 — Foundations

What "scale" actually means, the physics you can't negotiate with, and the vocabulary everything else uses.

- [[What Is System Design]] — what the discipline is, and why there's no "correct" answer
- [[Anatomy of a Web Request]] — the full journey: DNS → LB → app → DB → response
- [[Scalability]] — load parameters, why scaling is about *bottlenecks*, not servers
- [[Vertical vs Horizontal Scaling]] — scale up vs scale out, and why everyone ends up horizontal
- [[Latency vs Throughput vs Bandwidth]] — the three words people conflate, with percentiles (p50/p99)
- [[Latency Numbers Every Engineer Should Know]] — the Jeff Dean table, memorized ratios, 2020s updates
- [[Availability Reliability Durability]] — the nines table, series vs parallel availability math
- [[SLA SLO SLI and Error Budgets]] — how Google-style reliability targets actually work
- [[Back of the Envelope Estimation]] — the interview superpower: RPS, storage, bandwidth math
- [[Stateless vs Stateful Services]] — why statelessness is the price of horizontal scaling

## Module 2 — Networking Fundamentals

The layer under everything. Most "mysterious" distributed-systems failures are networking failures.

- [[How DNS Works]] — recursive resolution, TTLs, DNS as a load balancing and failover tool
- [[TCP vs UDP]] — handshakes, ordering, head-of-line blocking, when UDP wins
- [[TLS and HTTPS]] — handshake cost, session resumption, mTLS, where TLS terminates
- [[HTTP 1.1 vs HTTP 2 vs HTTP 3]] — multiplexing, QUIC, why HTTP/2 to the browser but often 1.1 inside the cluster
- [[REST vs gRPC vs GraphQL]] — contract styles, payload cost, when each wins
- [[WebSockets Server-Sent Events and Long Polling]] — pushing data to clients
- [[Network Partitions and Timeouts]] — the network is not reliable; what partial failure looks like

## Module 3 — Traffic Management

Getting requests to the right healthy server, and refusing the ones you can't serve.

- [[Load Balancers]] — L4 vs L7, algorithms (round robin, least-conn, consistent hash), health checks
- [[API Gateway]] — authn, rate limiting, routing at the edge; gateway vs load balancer (Kong at Niyo)
- [[Reverse Proxy vs Forward Proxy]]
- [[CDN]] — edge caching, push vs pull, invalidation, why static ≠ the only use case
- [[Rate Limiting]] — token bucket, leaky bucket, sliding window; local vs distributed limiters
- [[Health Checks and Service Discovery]] — liveness vs readiness, registries, DNS-based vs client-side discovery

## Module 4 — Caching & Redis

The highest-leverage performance tool, and the fastest way to serve stale or wrong data.

- [[Why Caching Works]] — locality, the 80/20 access skew, cache hit math
- [[Caching Strategies]] — cache-aside, read-through, write-through, write-behind, write-around
- [[Cache Eviction Policies]] — LRU, LFU, TTL, and why Redis uses approximated LRU
- [[Cache Invalidation]] — "one of the two hard problems": TTL vs event-driven invalidation
- [[Cache Stampede and Thundering Herd]] — dogpile effect, request coalescing, jittered TTLs
- [[Redis Internals]] — single-threaded event loop, data structures, persistence (RDB vs AOF)
- [[Redis Cluster and Sentinel]] — how Redis itself scales and fails over; hash slots
- [[Redis Beyond Caching]] — locks (and why Redlock is contested), counters, leaderboards, pub/sub, streams
- [[Distributed Locking]] — fencing tokens, why "just use a Redis lock" has sharp edges
- [[Hot Key Problem]] — when one key melts a shard: detection, local caching, key splitting

## Module 5 — Relational Databases

The default datastore, and what it actually guarantees.

- [[How Databases Store Data]] — pages, heap files, WAL (write-ahead log), buffer pool
- [[Database Indexes]] — B-trees, composite indexes, covering indexes, why writes pay for reads
- [[ACID Transactions]] — what each letter really guarantees (and doesn't)
- [[Transaction Isolation Levels]] — dirty reads → serializable; what MySQL/Postgres default to
- [[Connection Pooling]] — why DBs die at a few thousand connections
- [[N+1 Queries and ORM Pitfalls]]
- [[Normalization vs Denormalization]] — when to break the rules, and the cost of doing so
- [[Database Scaling Patterns]] — read replicas → caching → partitioning, in that order

## Module 6 — Replication, Partitioning & Sharding

How data survives machine death and outgrows one machine. **This module answers your sharding and hot-partition questions.**

- [[Replication]] — leader-follower, multi-leader, leaderless; sync vs async; replication lag
- [[Read Your Own Writes and Replica Lag Anomalies]] — monotonic reads, consistent prefix
- [[Failover]] — leader election, split brain, why failover is the dangerous part
- [[Sharding]] — what it is, range vs hash partitioning, choosing a shard key
- [[Consistent Hashing]] — the ring, virtual nodes, why naive mod-N hashing fails on resize
- [[Hot Partition Problem]] — celebrity keys, time-ordered keys, salting, splitting (answers "hot key partition")
- [[Rebalancing Partitions]] — moving data without downtime
- [[Secondary Indexes on Sharded Data]] — local vs global indexes, scatter-gather reads
- [[Why MySQL Does Scale]] — Facebook/YouTube-style sharded MySQL, Vitess — kills the "MySQL doesn't scale" myth

## Module 7 — NoSQL & Storage Engines

Different storage for different shapes of data — and the engines underneath.

- [[B-Tree vs LSM-Tree]] — the two storage engines behind almost every database
- [[NoSQL Taxonomy]] — key-value, document, wide-column, graph: what each models well
- [[The Dynamo Paper]] — Amazon's 2007 paper: quorums, hinted handoff, vector clocks, gossip
- [[DynamoDB Design]] — partitions, adaptive capacity, single-table design, hot partition handling
- [[How S3 Works]] — object storage at planet scale: durability math (11 nines), erasure coding
- [[Cassandra and Wide-Column Stores]] — tunable consistency, write-optimized paths
- [[MongoDB Architecture]] — replica sets, sharded clusters, when documents fit (your Niyo default)
- [[Bigtable and Spanner]] — Google's path: GFS → Bigtable → Spanner, TrueTime, globally consistent SQL
- [[Time Series and Analytics Stores]] — columnar storage, why OLAP ≠ OLTP
- [[Choosing a Database]] — a decision framework, not a popularity contest

## Module 8 — Distributed Systems Theory

The rules of the game. Short module, highest concept density — everything in Modules 6–9 is an application of this.

- [[The Fallacies of Distributed Computing]] — the 8 assumptions that kill systems
- [[CAP Theorem]] — what it actually says (less than people claim), CP vs AP in practice
- [[PACELC]] — the useful extension: latency vs consistency even without partitions
- [[Consistency Models]] — linearizability → eventual, a spectrum not a binary
- [[Logical Clocks and Ordering]] — happens-before, Lamport clocks, vector clocks
- [[Consensus and Raft]] — why agreement is hard, leader election, log replication, quorums
- [[Quorum Reads and Writes]] — R + W > N, sloppy quorums
- [[Two-Phase Commit and Distributed Transactions]] — why 2PC blocks, why everyone avoids it
- [[Idempotency]] — the most practical idea in this module: idempotency keys, retries without double-charges
- [[Exactly-Once Delivery Is a Lie]] — at-most-once, at-least-once, and what "exactly-once" actually means
- [[Split Brain and Fencing]]

## Module 9 — Messaging & Event-Driven Systems 🔥

**Your deepest module.** Covers every broker you've used (RabbitMQ, Kafka, SQS) and the one you're moving to (Redis Streams), plus the patterns that make async architectures safe.

### Concepts
- [[Why Async Messaging Exists]] — temporal decoupling, load leveling, the problem with synchronous chains
- [[Message Queues vs Event Logs]] — the single most important distinction: RabbitMQ/SQS vs Kafka
- [[Delivery Semantics]] — at-least-once in practice, consumer idempotency, acks
- [[Message Ordering]] — why global order doesn't scale, per-key ordering, what breaks it
- [[Dead Letter Queues and Poison Messages]]
- [[Backpressure and Flow Control]] — what happens when consumers can't keep up
- [[Consumer Groups and Rebalancing]]
- [[The Outbox Pattern]] — atomically updating a DB *and* publishing an event (dual-write problem)
- [[Change Data Capture]] — Debezium-style: the database as an event source
- [[Event Sourcing]] — events as the source of truth; when it's brilliant and when it's a trap
- [[CQRS]] — separating read and write models
- [[Saga Pattern]] — distributed transactions without 2PC; choreography vs orchestration
- [[Event Schema Evolution]] — schema registry, compatibility modes, versioning events

### Technologies (compared on one rubric)
- [[RabbitMQ]] — smart broker, exchanges/bindings, per-message acks, where it shines and strains
- [[Kafka Architecture]] — partitions, offsets, ISR, consumer groups, log compaction, retention
- [[Kafka Operations]] — self-managed vs AWS MSK vs Confluent: what you're actually paying for
- [[SQS and SNS]] — serverless queueing, visibility timeout, FIFO queues, fan-out
- [[Redis Streams]] — consumer groups, PEL, XAUTOCLAIM; honest limits vs Kafka
- [[Choosing a Message Broker]] — decision framework + migration notes (your RabbitMQ → Kafka → SQS → Redis Streams journey)

## Module 10 — Architecture Styles

The monolith vs microservices question, answered without religion.

- [[Monolith vs Microservices]] — what each actually costs; the answer is almost always "it depends on your org"
- [[Modular Monolith]] — the underrated middle; boundaries without network calls
- [[Service Boundaries and Domain-Driven Design]] — bounded contexts, why entity-per-service fails
- [[Communication Patterns Between Services]] — sync vs async, orchestration vs choreography
- [[API Design and Versioning]] — compatibility, pagination, error contracts
- [[Service Mesh]] — sidecars, when mesh is overkill (usually)
- [[The Distributed Monolith Anti-Pattern]] — microservices' failure mode: all the cost, none of the benefit
- [[Strangler Fig Migration]] — how to actually decompose a monolith

## Module 11 — Authentication, Authorization & IAM

**How your JWT + IAM setup works under the hood**, and how to design one.

- [[Sessions vs Tokens]] — cookie sessions, the stateful/stateless trade, CSRF
- [[How JWT Works]] — structure, signing (HS256 vs RS256), claims, validation, the logout problem
- [[JWT Pitfalls]] — alg=none, long expiry, revocation, where NOT to put data in a JWT
- [[Refresh Tokens and Token Rotation]]
- [[OAuth 2 and OpenID Connect]] — the flows, which one for which client, PKCE
- [[Designing an IAM Service]] — authn vs authz split, token issuance, key rotation (JWKS), your IAM service's shape
- [[RBAC vs ABAC]] — roles vs attributes vs policies
- [[API Security Essentials]] — mTLS, HMAC request signing, replay protection
- [[Single Sign-On]]

## Module 12 — Resilience Engineering

Designing for the failures that *will* happen. **Covers Chaos Monkey.**

- [[Timeouts Retries and Exponential Backoff]] — the trio every network call needs; retry storms
- [[Circuit Breakers]] — failing fast, half-open probing
- [[Bulkheads and Isolation]] — containing blast radius
- [[Load Shedding and Graceful Degradation]] — serving *something* at 1M RPS when you can only afford 800K
- [[Failure Modes in Distributed Systems]] — cascading failure, metastable failure, retry amplification
- [[Chaos Engineering]] — Netflix's Chaos Monkey and Simian Army: breaking prod on purpose
- [[Disaster Recovery]] — RPO/RTO, backup vs replication, multi-region active-active vs active-passive
- [[Graceful Shutdown and Deployment Safety]] — connection draining, probes (your Facets/K8s world)

## Module 13 — Observability & Operations

You can't fix what you can't see.

- [[Logs Metrics and Traces]] — the three pillars and what each answers
- [[Distributed Tracing]] — correlation IDs (your x-correlation-id convention), spans, sampling
- [[Alerting and On-Call Design]] — symptom-based alerts, alert fatigue
- [[Deployment Strategies]] — blue-green, canary, rolling, feature flags
- [[Autoscaling]] — HPA, metrics to scale on, why CPU is often the wrong signal
- [[Capacity Planning]] — headroom, load testing, finding your real ceiling

## Module 14 — Planet-Scale Case Studies

How the giants actually do it. Read these *after* the theory — they're applications of Modules 6–9.

- [[Anatomy of 1M RPS]] — what actually changes at each order of magnitude: 100 → 10K → 1M RPS
- [[How Facebook Scales MySQL]] — sharded MySQL + TAO cache + async replication (the myth-killer, in depth)
- [[Google Spanner Deep Dive]] — TrueTime, external consistency, global SQL
- [[DynamoDB in Production]] — adaptive capacity, on-demand scaling, real failure stories
- [[S3 Deep Dive]] — scale numbers, durability engineering, strong consistency (2020 change)
- [[Netflix Architecture]] — microservices at extreme scale, Open Connect CDN, resilience culture
- [[WhatsApp and the Small-Team Scale Story]] — 50 engineers, 900M users: Erlang, simplicity
- [[Discord Message Storage Evolution]] — MongoDB → Cassandra → ScyllaDB: a migration story in three acts
- [[Uber Architecture]] — geo-indexing (H3), dispatch, the DOMA model
- [[Twitter Timeline Design]] — fan-out on write vs read: the canonical hot-key case study
- [[Stripe and Payment System Design]] — idempotency at money-scale, double-entry ledgers

## Module 15 — The Interview Playbook

The framework, then the classics. Each problem note = requirements → estimation → high-level → deep dives → wrap.

- [[The System Design Interview Framework]] — the 4-step method, time budgeting, signals interviewers grade
- [[Design a URL Shortener]] — the "hello world": ID generation, redirects, cache
- [[Design a Rate Limiter]]
- [[Design a Distributed ID Generator]] — Snowflake IDs
- [[Design a Notification Service]] — fan-out, dedup, retries (very close to your day job)
- [[Design a Chat System]] — WhatsApp/Slack: WebSockets, message sync, online presence
- [[Design a News Feed]] — fan-out strategies, ranking, celebrity problem
- [[Design Typeahead Autocomplete]] — tries, top-k, precomputation
- [[Design a Web Crawler]] — politeness, frontier, dedup at scale
- [[Design Distributed File Storage]] — Dropbox/Drive: chunking, sync, metadata
- [[Design a Video Streaming Service]] — YouTube/Netflix: upload pipeline, ABR, CDN
- [[Design a Ride Sharing Service]] — Uber: location updates, matching, surge
- [[Design a Payment System]] — ledgers, idempotency, reconciliation (your fintech home turf)
- [[Design a Stock Exchange]] — matching engines, fairness, extreme low latency
- [[Design a Distributed Cache]] — build Redis-cluster-like from first principles
- [[Design a Key-Value Store]] — build mini-Dynamo: the capstone that uses everything

## Module 16 — Modern AI Infrastructure *(bonus)*

Increasingly appears in 2026 interviews; skim after everything else.

- [[Vector Databases and Similarity Search]] — embeddings, ANN indexes (HNSW), when pgvector is enough
- [[RAG Architecture]] — retrieval pipelines as a system design problem
- [[LLM Serving Infrastructure]] — batching, KV caches, GPU utilization, token streaming

---

## Progress tracker

All 16 modules are written. Use the **Read** column to track your own progress — tick a module when you've read it *and* redrawn its core diagram from memory.

| Module | Notes | Written | Read |
|---|---|---|---|
| 1. Foundations | 10 | ✅ | ⬜ |
| 2. Networking | 7 | ✅ | ⬜ |
| 3. Traffic Management | 6 | ✅ | ⬜ |
| 4. Caching & Redis | 10 | ✅ | ⬜ |
| 5. Relational Databases | 8 | ✅ | ⬜ |
| 6. Replication & Sharding | 9 | ✅ | ⬜ |
| 7. NoSQL & Storage Engines | 10 | ✅ | ⬜ |
| 8. Distributed Systems Theory | 11 | ✅ | ⬜ |
| 9. Messaging & Event-Driven | 19 | ✅ | ⬜ |
| 10. Architecture Styles | 8 | ✅ | ⬜ |
| 11. Auth & IAM | 9 | ✅ | ⬜ |
| 12. Resilience | 8 | ✅ | ⬜ |
| 13. Observability | 6 | ✅ | ⬜ |
| 14. Case Studies | 11 | ✅ | ⬜ |
| 15. Interview Playbook | 16 | ✅ | ⬜ |
| 16. AI Infrastructure | 3 | ✅ | ⬜ |

**151 notes.** Practice drills live in [[Practice Drills]].

## Primary sources this curriculum draws from

- *Designing Data-Intensive Applications* (Kleppmann) — the bible for Modules 5–9
- *System Design Interview Vol 1 & 2* (Alex Xu) — Module 15's problem structure
- Grokking the System Design Interview (Design Gurus/Educative) — component coverage
- Papers: Dynamo (2007), Bigtable (2006), Spanner (2012), Raft (2014), TAO (2013)
- Engineering blogs: Netflix TechBlog, Uber Engineering, Discord, Cloudflare, Stripe, Meta Engineering, AWS Builders' Library
