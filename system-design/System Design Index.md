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

> **Start here.** This is the home MOC for the entire curriculum. Every topic below is a written note — all links resolve, so you can click straight through from here.

## How to use this vault

1. **Go in order the first time.** Modules build on each other: you can't reason about Kafka partitioning (Module 9) without sharding (Module 6), and you can't reason about sharding without replication and latency basics (Modules 1, 6).
2. **Every component note follows the same skeleton** — *the problem that created it → what it solves → how it works → trade-offs → failure modes → when NOT to use it → in your stack → interview angle → check yourself → sources*. If you remember nothing else from a note, remember the problem it was invented to solve; everything else re-derives from that.
3. **Draw it yourself.** Mermaid diagrams are embedded where precision matters; for intuition, redraw the architecture in Excalidraw from memory after reading. If you can't redraw it, you haven't learned it.
4. **Link as you learn.** When a note reminds you of something at work (a Niyo incident, a config you've seen), add your own line under *In your stack*. Notes you've annotated are notes you'll remember.

## Study tracks

- **SDE2 interview track (current goal)** — depth where it is graded, breadth elsewhere:
  1. **Non-negotiable core**: Modules 1 (foundations + estimation), 3 (load balancing, gateway, rate limiting), 4 (caching), 5 (databases + indexing), 6 (replication & sharding), 9 (messaging basics), 12 (timeouts, retries, circuit breakers).
  2. **Know the vocabulary, don't go deep**: Module 8 — CAP/PACELC, consistency models, idempotency. Be able to define them and say why they matter; you are not expected to derive consensus protocols.
  3. **Drill these problems**: all of Tier 1 and Tier 2 in [[Practice Drills]]. SDE2 rounds are almost always one of these, or a close variant.
  4. **Know your own platform cold**: Modules 17 (how your code ships, Kubernetes basics) and 18 (the AWS services you use). Product-company interviewers ask "walk me through a deploy" and "which AWS services have you used and why" — these are the easiest points to score and the easiest to fumble.
  5. **Bonus, not blockers**: Modules 7 (storage engines), 14 (case studies), 16 (AI infra). Read for interest and for the one or two details that make an answer sound lived-in.
  - What is actually graded at SDE2: driving the conversation, clarifying requirements before designing, putting numbers on things, and naming trade-offs out loud. See [[The System Design Interview Framework]].
- **Guided trails** — two curated paths through existing notes:
  - [[Database Scaling Path]] — indexing → caching → replicas → eventual consistency → CAP, with the write fork into sharding → sagas
  - [[Microservices Patterns Path]] — proxies, gateways, BFF, discovery, DB-per-service, sagas, circuit breakers, bulkheads, sidecars, mesh
- **Work track** (event-driven systems, Redis, auth): Modules 1 → 4 → 8 → 9 → 11 → 12
- **Interview track** (8–10 weeks): all modules in order, then drill Module 15 problems twice
- **Night before an interview**: [[Numbers and Formulas]] — the one-page recall sheet
- **"I have one weekend"**: [[Back of the Envelope Estimation]], [[Latency Numbers Every Engineer Should Know]], [[Replication]], [[Sharding]], [[Caching Strategies]], [[Message Queues vs Event Logs]], [[The System Design Interview Framework]]

---

## Module 1 — Foundations

What "scale" actually means, the physics you can't negotiate with, and the vocabulary everything else uses.

- [[What Is System Design]] — what the discipline is, and why there's no "correct" answer
- [[Anatomy of a Web Request]] — the full journey: DNS → TCP/TLS → CDN → LB → gateway → app → cache → DB, and where latency accumulates
- [[Scalability]] — load parameters, why scaling is about *bottlenecks*, not servers
- [[Vertical vs Horizontal Scaling]] — scale up vs scale out — cost curves, Amdahl's ceiling, why stateless tiers go out and databases go up first
- [[Latency vs Throughput vs Bandwidth]] — the three words people conflate, with percentiles (p50/p99)
- [[Latency Numbers Every Engineer Should Know]] — the Jeff Dean table, memorized ratios, 2020s updates, realistic cross-region RTTs
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
- [[WebSockets Server-Sent Events and Long Polling]] — long polling vs SSE vs WebSockets, connection-count sizing, why push tiers are stateful
- [[Network Partitions and Timeouts]] — the network is not reliable; what partial failure looks like

## Module 3 — Traffic Management

Getting requests to the right healthy server, and refusing the ones you can't serve.

- [[Load Balancers]] — L4 vs L7, algorithms (round robin, least-conn, consistent hash), health checks
- [[API Gateway]] — authn, rate limiting, routing at the edge; gateway vs load balancer (Kong at Niyo)
- [[Reverse Proxy vs Forward Proxy]] — who each proxy represents; CDNs, gateways and L7 LBs as specialised reverse proxies; X-Forwarded-For trust
- [[CDN]] — edge caching, push vs pull, invalidation, why static ≠ the only use case
- [[Rate Limiting]] — the five algorithms (token/leaky bucket, fixed window, sliding log/counter); local vs distributed; 429 and Retry-After
- [[Health Checks and Service Discovery]] — liveness vs readiness vs startup, shallow vs deep checks, registries (Eureka/Consul), Kubernetes Services as discovery

## Module 4 — Caching & Redis

The highest-leverage performance tool, and the fastest way to serve stale or wrong data.

- [[Why Caching Works]] — locality, Zipf skew and what it really buys, hit-ratio math, why cache when indexes exist
- [[Caching Strategies]] — cache-aside, read-through, write-through, write-behind, write-around
- [[Cache Eviction Policies]] — LRU, LFU, TTL, and why Redis uses approximated LRU
- [[Cache Invalidation]] — TTL, explicit DEL, CDC, versioned keys and Redis client-side tracking — and the races each leaves open
- [[Cache Stampede and Thundering Herd]] — dogpile effect, request coalescing, jittered TTLs, stale-while-revalidate, XFetch
- [[Redis Internals]] — single-threaded event loop, data structures, persistence (RDB vs AOF)
- [[Redis Cluster and Sentinel]] — how Redis itself scales and fails over; hash slots
- [[Redis Beyond Caching]] — counters, leaderboards, locks, pub/sub vs streams, sessions, dedup, delay queues — and what losing each key means
- [[Distributed Locking]] — fencing tokens, why "just use a Redis lock" has sharp edges
- [[Hot Key Problem]] — when one key melts a shard: detection, local caching, key splitting

## Module 5 — Relational Databases

The default datastore, and what it actually guarantees.

- [[How Databases Store Data]] — pages, heap files, WAL (write-ahead log), buffer pool
- [[Database Indexes]] — from pages on disk up: why not a binary tree, B+-tree fanout, clustered vs heap, composite/covering, MongoDB ESR and the RAM wall
- [[Query Planning and EXPLAIN]] — how the optimizer chooses, statistics and cardinality, reading a plan in Postgres/MySQL/Mongo, why plans go wrong
- [[ACID Transactions]] — what each letter really guarantees (and doesn't)
- [[Transaction Isolation Levels]] — dirty reads → serializable; what MySQL/Postgres default to
- [[Connection Pooling]] — why small pools beat big ones, HikariCP defaults, PgBouncer, pool exhaustion as an outage shape
- [[N+1 Queries and ORM Pitfalls]] — lazy loading's hidden query multiplication, the three fixes (join fetch, batch fetch, DTO projection), and the sibling ORM traps
- [[Normalization vs Denormalization]] — when to break the rules, and the cost of doing so
- [[Database Scaling Patterns]] — the ladder in order: tune and index → cache → read replicas → scale up → split by function → shard last; why skipping rungs hurts

> **Guided trail:** [[Database Scaling Path]] walks Modules 5, 6 and 8 as one chain — indexing → caching → replicas → eventual consistency → CAP, with the write-side fork into sharding → distributed transactions → sagas.

## Module 6 — Replication, Partitioning & Sharding

How data survives machine death and outgrows one machine. **This module answers your sharding and hot-partition questions.**

- [[Replication]] — leader-follower, multi-leader, leaderless; sync vs async vs semi-sync and what each mode can lose; replication lag
- [[Read Your Own Writes and Replica Lag Anomalies]] — read-your-writes, monotonic reads, consistent prefix; what leader-pinning costs, causality tokens, measuring lag honestly
- [[Failover]] — leader election, split brain, why failover is the dangerous part
- [[Sharding]] — what it is, range vs hash partitioning, choosing a shard key
- [[Consistent Hashing]] — the ring, virtual nodes, why naive mod-N hashing fails on resize
- [[Hot Partition Problem]] — celebrity keys, time-ordered keys, salting, splitting (answers "hot key partition")
- [[Rebalancing Partitions]] — fixed partitions vs dynamic splitting vs consistent hashing; the backfill → change-replay → verify → cutover playbook; throttling guardrails
- [[Secondary Indexes on Sharded Data]] — local vs global indexes, scatter-gather reads
- [[Why MySQL Does Scale]] — Facebook/YouTube-style sharded MySQL, Vitess, Shopify pods; which single-box wall breaks first — kills the "MySQL doesn't scale" myth

## Module 7 — NoSQL & Storage Engines

Different storage for different shapes of data — and the engines underneath.

- [[B-Tree vs LSM-Tree]] — the two storage engines behind almost every database
- [[NoSQL Taxonomy]] — key-value, document, wide-column, graph: what each models well
- [[The Dynamo Paper]] — Amazon's 2007 paper: quorums, hinted handoff, vector clocks, gossip
- [[DynamoDB Design]] — partitions, adaptive capacity, single-table design, hot partition handling
- [[How S3 Works]] — object storage at planet scale: durability math (11 nines), erasure coding, strong consistency since Dec 2020
- [[Cassandra and Wide-Column Stores]] — tunable consistency, write-optimized paths
- [[MongoDB Architecture]] — replica sets, sharded clusters, when documents fit (your Niyo default)
- [[Bigtable and Spanner]] — Google's path: GFS → Bigtable → Spanner, TrueTime, globally consistent SQL
- [[Time Series and Analytics Stores]] — columnar storage and why OLAP ≠ OLTP; time-series stores — Gorilla compression, downsampling, retention, cardinality
- [[Choosing a Database]] — a decision framework, not a popularity contest

## Module 8 — Distributed Systems Theory

The rules of the game. Short module, highest concept density — everything in Modules 6–9 is an application of this.

- [[The Fallacies of Distributed Computing]] — the 8 assumptions that kill systems, plus the ninth: grey failure
- [[CAP Theorem]] — what it actually says (less than people claim), CP vs AP in practice
- [[PACELC]] — the useful extension: latency vs consistency even without partitions
- [[Consistency Models]] — linearizability → eventual, a spectrum not a binary
- [[Logical Clocks and Ordering]] — happens-before, Lamport and vector clocks, TrueTime/HLC, CRDTs, and choosing the right clock
- [[Consensus and Raft]] — why agreement is hard, leader election, log replication, quorums
- [[Quorum Reads and Writes]] — R + W > N, sloppy quorums
- [[Two-Phase Commit and Distributed Transactions]] — why 2PC blocks, why everyone avoids it
- [[Idempotency]] — the most practical idea in this module: idempotency keys, retries without double-charges
- [[Exactly-Once Delivery Is a Lie]] — at-most-once, at-least-once, what "exactly-once" actually means, and what Kafka EOS really covers
- [[Split Brain and Fencing]] — two leaders at once from a partition or GC pause; quorum election, fencing tokens, STONITH

## Module 9 — Messaging & Event-Driven Systems 🔥

**Your deepest module.** Covers every broker you've used (RabbitMQ, Kafka, SQS) and the one you're moving to (Redis Streams), plus the patterns that make async architectures safe.

### Concepts
- [[Why Async Messaging Exists]] — temporal decoupling, load leveling, the problem with synchronous chains
- [[Message Queues vs Event Logs]] — the single most important distinction: RabbitMQ/SQS vs Kafka
- [[Delivery Semantics]] — at-least-once in practice, consumer idempotency, acks
- [[Message Ordering]] — why global order doesn't scale, per-key ordering, what breaks it
- [[Dead Letter Queues and Poison Messages]] — poison messages, max-receive caps, retry vs quarantine, triage and redrive; why a write-only DLQ is worse than none
- [[Backpressure and Flow Control]] — what happens when consumers can't keep up
- [[Consumer Groups and Rebalancing]] — competing consumers vs partition ownership; eager vs cooperative rebalancing, static membership; partitions cap group parallelism
- [[The Outbox Pattern]] — atomically updating a DB *and* publishing an event (dual-write problem)
- [[Change Data Capture]] — Debezium-style: the database log as an event source; Mongo change streams, DynamoDB Streams
- [[Event Sourcing]] — events as the source of truth; when it's brilliant and when it's a trap
- [[CQRS]] — separating read and write models
- [[Saga Pattern]] — distributed transactions without 2PC; choreography vs orchestration
- [[Event Schema Evolution]] — schema registry, compatibility modes, versioning events

### Technologies (compared on one rubric)
- [[RabbitMQ]] — smart broker, exchanges/bindings, per-message acks, quorum queues (mirroring removed in 4.0), where it shines and strains
- [[Kafka Architecture]] — partitions, offsets, ISR, consumer groups, log compaction, retention
- [[Kafka Operations]] — self-managed vs AWS MSK vs Confluent: what you're actually paying for
- [[SQS and SNS]] — serverless queueing, visibility timeout, FIFO queues, fan-out
- [[Redis Streams]] — consumer groups, PEL, XAUTOCLAIM; honest limits vs Kafka
- [[Choosing a Message Broker]] — decision framework + migration notes (your RabbitMQ → Kafka → SQS → Redis Streams journey)

## Module 10 — Architecture Styles

The monolith vs microservices question, answered without religion.

> **Guided trail:** [[Microservices Patterns Path]] walks every microservices pattern in the vault grouped by the problem it solves — edge, discovery, data ownership, failure, and the infrastructure layer.

- [[Monolith vs Microservices]] — what each actually costs; the answer is almost always "it depends on your org"
- [[Modular Monolith]] — the underrated middle; boundaries without network calls
- [[Service Boundaries and Domain-Driven Design]] — bounded contexts, why entity-per-service fails
- [[Communication Patterns Between Services]] — sync vs async, call-chain availability and tail math, API composition vs materialized views
- [[API Design and Versioning]] — compatibility, pagination, error contracts
- [[Orchestration vs Choreography]] — who is in charge of a multi-step process: central coordinator vs event reactions; designing a durable orchestrator; Temporal and Step Functions
- [[Database per Service]] — the pattern that makes a split real; private data, and what you give up to get it
- [[Backend for Frontend]] — one backend per client experience; BFF vs gateway vs GraphQL
- [[Sidecar Pattern]] — a second container sharing the pod's network namespace; the mechanism under Envoy and Istio
- [[Service Mesh]] — control plane + Envoy data plane; mTLS, retry budgets, when mesh is overkill (usually)
- [[The Distributed Monolith Anti-Pattern]] — microservices' failure mode: all the cost, none of the benefit
- [[Strangler Fig Migration]] — how to actually decompose a monolith

## Module 11 — Authentication, Authorization & IAM

**How your JWT + IAM setup works under the hood**, and how to design one.

- [[Sessions vs Tokens]] — cookie sessions, the stateful/stateless trade, CSRF
- [[How JWT Works]] — structure, signing (HS256 vs RS256/ES256), claims, validation order, clock leeway, JWKS and kid rotation
- [[JWT Pitfalls]] — alg=none, long expiry, revocation, where NOT to put data in a JWT
- [[Refresh Tokens and Token Rotation]] — access/refresh split, rotation with reuse detection, absolute vs sliding sessions, refresh traffic cost
- [[OAuth 2 and OpenID Connect]] — the flows, which one for which client, PKCE
- [[Designing an IAM Service]] — authn vs authz split, token issuance, key rotation (JWKS), your IAM service's shape
- [[RBAC vs ABAC]] — roles vs attributes vs policies, ReBAC, token vs request-time checks, maker-checker
- [[CORS]] — the same-origin policy and how a server opts out of it; why CORS is browser-enforced and not an API defence
- [[API Security Essentials]] — mTLS, HMAC request signing, replay protection
- [[Single Sign-On]] — IdP vs SP, SAML vs OIDC, the two-session model, why single logout is best-effort, workforce vs customer identity

## Module 12 — Resilience Engineering

Designing for the failures that *will* happen. **Covers Chaos Monkey.**

- [[Timeouts Retries and Exponential Backoff]] — the trio every network call needs; retry storms
- [[Circuit Breakers]] — failing fast, half-open probing
- [[Bulkheads and Isolation]] — containing blast radius
- [[Load Shedding and Graceful Degradation]] — admission control, priority tiers, bounded queues, brownouts — the 1,200-vs-800 RPS collapse arithmetic
- [[Failure Modes in Distributed Systems]] — cascading, retry amplification, thundering herd, metastable, gray and correlated failure — signatures and defenses
- [[Chaos Engineering]] — Netflix's Chaos Monkey (2010) and Simian Army (2011, now retired), and the experiment discipline they grew into
- [[Disaster Recovery]] — RPO/RTO, backup vs replication, multi-region active-active vs active-passive
- [[Graceful Shutdown and Deployment Safety]] — SIGTERM, the preStop endpoint race, draining, consumer rebalance, and 1-replica rollout/PDB honesty (your Facets/K8s world)

## Module 13 — Observability & Operations

You can't fix what you can't see.

- [[Logs Metrics and Traces]] — the three pillars and what each answers
- [[Distributed Tracing]] — correlation IDs (your x-correlation-id convention), W3C traceparent, OpenTelemetry, head vs tail sampling
- [[Alerting and On-Call Design]] — symptom-based alerts, multi-window SLO burn rates, page vs ticket, alert fatigue
- [[Deployment Strategies]] — rolling, blue-green, canary, shadow, feature flags, expand–migrate–contract migrations
- [[Autoscaling]] — HPA, metrics to scale on, why CPU is often the wrong signal
- [[Capacity Planning]] — load testing, USE method, N+1 and safety-factor headroom, Little's Law, finding your real ceiling

## Module 14 — Planet-Scale Case Studies

How the giants actually do it. Read these *after* the theory — they're applications of Modules 6–9.

- [[Anatomy of 1M RPS]] — what actually changes at each order of magnitude: 100 → 10K → 1M RPS
- [[How Facebook Scales MySQL]] — sharded MySQL + TAO cache + async cross-region replication, MyRocks and MySQL Raft (the myth-killer, in depth)
- [[Google Spanner Deep Dive]] — TrueTime, external consistency, global SQL
- [[DynamoDB in Production]] — adaptive capacity, on-demand scaling, real failure stories; Global Tables' two consistency modes
- [[S3 Deep Dive]] — scale numbers, durability engineering, strong consistency (2020 change)
- [[Netflix Architecture]] — microservices at extreme scale, Open Connect CDN, resilience culture
- [[WhatsApp and the Small-Team Scale Story]] — 50 engineers, 900M users (2015): Erlang, simplicity
- [[Discord Message Storage Evolution]] — MongoDB → Cassandra → ScyllaDB: a migration story in three acts
- [[Uber Architecture]] — geo-indexing (H3), dispatch, the DOMA model
- [[Twitter Timeline Design]] — fan-out on write vs read: the canonical hot-key case study
- [[Stripe and Payment System Design]] — idempotency at money-scale, double-entry ledgers

## Module 15 — The Interview Playbook

The framework, then the classics. Each problem note = requirements → estimation → high-level → deep dives → wrap.

- [[The System Design Interview Framework]] — the 4-step method, time budgeting, signals interviewers grade
- [[Design a URL Shortener]] — the "hello world": ID generation, redirects, cache
- [[Design a Rate Limiter]] — algorithm choice, atomic Redis Lua counters, fail-open vs fail-closed, multi-region limits
- [[Design a Distributed ID Generator]] — Snowflake IDs: the 64-bit layout, clock-backwards handling, UUIDv7
- [[Design a Notification Service]] — fan-out, dedup, retries (very close to your day job)
- [[Design a Chat System]] — WhatsApp/Slack: WebSockets, message sync, online presence
- [[Design a News Feed]] — fan-out strategies, ranking, celebrity problem
- [[Design Typeahead Autocomplete]] — precomputed top-k per prefix (trie vs flat prefix table), offline build pipeline, hot prefixes, debounce
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
- [[LLM Serving Infrastructure]] — continuous batching, KV-cache memory, SSE streaming, token-based rate limits, why GPU utilization misleads autoscaling

## Module 17 — Shipping Code

How a commit becomes a running pod — the pipeline you actually use, and the Kubernetes underneath it.

> **Your flow:** Bitbucket Pipelines → Jib → ECR → EKS rollout. Start with [[The Deployment Pipeline]] — it traces one commit end to end and links every component note. [[Kubernetes Glossary]] is the reference for every K8s term.

- [[The Deployment Pipeline]] — Why code reaches production through a chain of gates rather than a person with SSH, and a full trace of one commit — pull request, Bitbucket…
- [[Docker and Container Images]] — What a container really is (a Linux process under namespaces and cgroups, sharing the host kernel), what an image is (immutable content-addressed…
- [[Jib]] — Why a Java service can become a container image without a Dockerfile or a Docker daemon — how Jib splits the app into layers by change frequency,…
- [[Container Registries and ECR]] — Why a built image needs a durable, authenticated, content-addressed home — how a registry stores blobs and manifests behind the OCI distribution API,…
- [[Kubernetes Architecture]] — Why Kubernetes exists and the one idea that explains all of it — reconciliation loops over desired state held in the API — then the control plane,…
- [[Kubernetes Workloads]] — Why a bare container cannot run a service, and how Kubernetes stacks Pod, ReplicaSet and Deployment to keep N copies alive and update them without…
- [[Kubernetes Networking]] — How a request finds a pod whose address changes every deploy — the flat pod network and the VPC CNI, Services as virtual IPs implemented by…
- [[Kubernetes Configuration and Resources]] — How one image runs unchanged in dev, uat and prod while many services share a node — ConfigMaps and Secrets and the env-var-versus-file trade-off,…
- [[EKS vs ECS]] — Straightens out three near-identical AWS names — ECR stores the image, ECS and EKS are two different orchestrators that run it — then compares what…
- [[Kubernetes Glossary]] — Alphabetical reference of the Kubernetes and EKS vocabulary this module uses, each term in one honest line with a pointer to the note that teaches it…

## Module 18 — AWS Services

The managed-AWS side of concepts the vault already teaches — vocabulary, knobs, limits, failure modes, and the **cost shape** of each (what the bill scales with).

> Start with [[AWS Services Map]] — service → generic concept → AWS note. Every note here has a *Cost shape* subsection.

- [[AWS Services Map]] — One-page lookup from each AWS service to the vault note that teaches the underlying concept and the note that teaches the AWS specifics, plus a…
- [[AWS Foundations]] — The ground every AWS note stands on — Regions, Availability Zones and edge locations as failure domains, accounts as the isolation and billing wall,…
- [[VPC and Networking]] — The private network under every AWS workload — CIDR and subnets, what makes a subnet public, NAT Gateway and why it is the classic cost trap,…
- [[IAM]] — AWS's authorization system for machines and engineers — principals, JSON policies, the deny-wins evaluation logic with SCPs and permission…
- [[EC2]] — AWS's raw virtual machine and the layer under EKS nodes — reading instance types, burstable credits, EBS volume behaviour, ENI-capped pod density,…
- [[ALB and NLB]] — AWS's managed load balancers and the L4-versus-L7 split behind them — ALB rules and target groups, NLB static IPs and flow hashing, IP targets on…
- [[Route 53]] — AWS's authoritative DNS, health checker and registrar — public and private hosted zones, alias records, the routing policies and the problem each…
- [[CloudFront]] — AWS's CDN as a configuration object — distributions, origins and path-matched behaviours, cache policies versus origin request policies (what is in…
- [[WAF and ACM]] — The two edge-security services that sit on the same door — ACM for certificates (DNS validation, renewal and the three ways it silently fails,…
- [[ElastiCache for Redis]] — AWS's managed Redis and Valkey — node-based clusters vs Serverless, cluster mode disabled vs enabled endpoints, what Multi-AZ failover guarantees and…
- [[CloudWatch]] — AWS's built-in telemetry store and alarm engine — metrics and the dimension-cardinality trap, log groups and the never-expire default, alarms and why…
- [[Secrets Manager Parameter Store and KMS]] — Where credentials and config live on AWS and how a pod gets them — KMS and envelope encryption underneath, Parameter Store for config, Secrets…
- [[Lambda]] — AWS's function-as-a-service — the execution environment lifecycle and cold starts, concurrency controls, SQS and other triggers, Java and SnapStart…
- [[AWS Cost Shape]] — The skill of naming what an AWS bill scales with — compute-hours, per-request, per-GB stored, processed and transferred, per-resource fees and…

**AWS-native services taught where their concept lives** (listed here so this section is complete):
- [[SQS and SNS]] and [[Kafka Operations]] (MSK) — Module 9
- [[How S3 Works]] and [[DynamoDB Design]] — Module 7; [[S3 Deep Dive]] and [[DynamoDB in Production]] — Module 14
- [[EKS vs ECS]] and [[Container Registries and ECR]] — Module 17

---

## Progress tracker

All 18 modules are written. Use the **Read** column to track your own progress — tick a module when you've read it *and* redrawn its core diagram from memory.

| Module | Notes | Written | Read |
|---|---|---|---|
| 1. Foundations | 10 | ✅ | ⬜ |
| 2. Networking | 7 | ✅ | ⬜ |
| 3. Traffic Management | 6 | ✅ | ⬜ |
| 4. Caching & Redis | 10 | ✅ | ⬜ |
| 5. Relational Databases | 9 | ✅ | ⬜ |
| 6. Replication & Sharding | 9 | ✅ | ⬜ |
| 7. NoSQL & Storage Engines | 10 | ✅ | ⬜ |
| 8. Distributed Systems Theory | 11 | ✅ | ⬜ |
| 9. Messaging & Event-Driven | 19 | ✅ | ⬜ |
| 10. Architecture Styles | 12 | ✅ | ⬜ |
| 11. Auth & IAM | 10 | ✅ | ⬜ |
| 12. Resilience | 8 | ✅ | ⬜ |
| 13. Observability | 6 | ✅ | ⬜ |
| 14. Case Studies | 11 | ✅ | ⬜ |
| 15. Interview Playbook | 16 | ✅ | ⬜ |
| 16. AI Infrastructure | 3 | ✅ | ⬜ |
| 17. Shipping Code | 10 | ✅ | ⬜ |
| 18. AWS Services | 14 | ✅ | ⬜ |

**181 notes.** Practice drills live in [[Practice Drills]]; the one-page recall sheet is [[Numbers and Formulas]]. `Vault Dashboard.base` gives filterable tables over every note.

## Primary sources this curriculum draws from

- *Designing Data-Intensive Applications* (Kleppmann) — the bible for Modules 5–9
- *System Design Interview Vol 1 & 2* (Alex Xu) — Module 15's problem structure
- Grokking the System Design Interview (Design Gurus/Educative) — component coverage
- Papers: Dynamo (2007), Bigtable (2006), Spanner (2012), Raft (2014), TAO (2013)
- Engineering blogs: Netflix TechBlog, Uber Engineering, Discord, Cloudflare, Stripe, Meta Engineering, AWS Builders' Library
