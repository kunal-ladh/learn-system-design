---
title: HLD MOC
aliases: [HLD Map, Curriculum, System Design Curriculum]
type: moc
tags: [moc, hld]
summary: "Master map of the HLD and system design curriculum — eleven categories, 166 concept notes, the problem bank, projects and resources."
created: 2026-10-29
updated: 2026-10-29
---

# HLD & System Design — Map of Content

> [!abstract] What this is
> The complete curriculum as a connected graph: **166 concept notes** in eleven categories, each linked to the design problems where it applies, the projects where you can build it, and the resources that explain it best. Nothing here is a checklist — every note is a place to write your own explanation and track your confidence.

> [!tip] New to system design?
> Read [[Start Here]] first. It tells you which 60 notes matter for SDE-2, in what order, and how to read a note without drowning in it.

**Navigation:** [[Engineering Dashboard]] (what to do next) · [[Knowledge Graph]] (the four chains) · [[Problem Bank MOC]] · [[Projects MOC]] · [[Resources MOC]] · [[Workspace Guide]]

## Learning tracks

- **SDE-2 interview track (12 weeks)** — the ordered plan lives in [[Start Here]]: Fundamentals → Networking essentials → Databases → Caching → Messaging → the resilience patterns → 8 design problems drilled twice. Distributed Systems, Storage & Search and Security are vocabulary at this level; Product Engineering is what makes you sound like you have shipped.
- **Work track (event-driven fintech backend)** — [[Messaging & Event-Driven Systems MOC]] → [[Databases MOC]] (transactions, locking, schema evolution) → [[Caching MOC]] → [[Security MOC]] → [[Observability & Reliability MOC]].
- **Product-engineering track (SDE-3 judgement)** — [[Product Engineering MOC]] end to end, then [[Architecture Patterns MOC]], then one project from [[Projects MOC]] taken through build → measure → improve → document.

## The eleven categories

### A. Fundamentals — [[Fundamentals MOC]]

*The vocabulary and the physics everything else is built on.*

- [[Scalability]] — Scalability is the ability to keep performance acceptable as a specific load parameter grows by adding resources
- [[Availability]] — Availability is the fraction of time, or of requests, for which a system gives a correct and timely answer
- [[Reliability]] — Reliability is the property of continuing to do the right thing when hardware, software or people go wrong
- [[Fault Tolerance]] — Fault tolerance is the set of techniques — redundancy, replication, isolation, failover and degradation — that let a system keep serving, possibly with reduced function…
- [[Latency]] — Latency is a distribution, not a number: the tail percentiles decide user experience, fan-out amplifies the tail multiplicatively
- [[Throughput]] — Throughput is completed work per unit time; it is tied to latency and concurrency by Little's Law, it is capped by the slowest stage of a pipeline
- [[CAP Theorem]] — CAP says that while a network partition is happening, a replicated system must choose between refusing some requests (consistency) and answering with possibly stale data…
- [[PACELC]] — PACELC extends CAP with the trade you pay every day: if there is a Partition choose Availability or Consistency, Else choose Latency or Consistency
- [[Consistency Models]] — A consistency model is the contract about which values a read may return given the writes that happened
- [[ACID]] — ACID names the four guarantees of a database transaction — all-or-nothing atomicity, application-defined consistency, isolation from concurrent transactions
- [[BASE]] — BASE — Basically Available, Soft state, Eventually consistent — is the design philosophy for partitioned stores that gives up cross-node atomicity and immediate…
- [[SLI SLO and SLA]] — An SLI is a measurement of one aspect of service quality as a ratio of good to total events, an SLO is the target that ratio must meet over a window
- [[Horizontal vs Vertical Scaling]] — Vertical scaling buys a bigger machine and keeps the architecture simple until the largest machine or its price runs out
- [[Back-of-the-Envelope Estimation]] — Back-of-the-envelope estimation turns a product requirement into orders of magnitude
- [[Bottleneck Analysis]] — Bottleneck analysis finds the one resource whose saturation limits a system's throughput
- [[Performance vs Cost Trade-offs]] — Cost is a non-functional requirement like latency: a cloud bill scales with compute hours, stored bytes, transferred bytes and request counts

### B. Networking — [[Networking MOC]]

*The layer under every request, and where most mysterious failures live.*

- [[HTTP and HTTPS]] — HTTP is the request-response protocol every API, browser, proxy and CDN speaks
- [[TCP and UDP]] — TCP gives you a reliable, ordered byte stream with flow and congestion control at the cost of handshakes and head-of-line blocking
- [[DNS]] — DNS is the globally distributed, heavily cached lookup service that turns names into records
- [[TLS]] — TLS authenticates the server (and optionally the client) with certificates and then encrypts the byte stream
- [[WebSockets]] — WebSockets upgrade an HTTP connection into a persistent full-duplex channel so the server can push
- [[gRPC]] — gRPC is contract-first RPC over HTTP/2 with Protocol Buffers
- [[REST]] — REST models an API as resources addressed by URLs and manipulated through HTTP's uniform, mostly idempotent methods and status codes
- [[Load Balancers]] — A load balancer spreads incoming connections or requests across a pool of healthy backends, at layer 4 by packet forwarding or at layer 7 by understanding the protocol
- [[Reverse Proxies]] — A reverse proxy is a server that accepts requests on behalf of one or more backends and forwards them
- [[CDNs]] — A CDN is a globally distributed fleet of caching reverse proxies that serves content from a point of presence near the user, cutting latency by removing round trips to…
- [[Connection Pooling]] — A connection pool keeps a bounded set of pre-established connections to a database, cache or HTTP service and lends them to callers, amortising the expensive setup cost…
- [[Timeouts and Retries]] — Every network call needs a deadline that bounds how long you will wait and a retry policy that says which failures to try again, how many times
- [[Keep-Alive Connections]] — Keep-alive reuses one TCP (and TLS) connection for many requests instead of opening a new one per request, removing handshake latency and churn
- [[Client State and Reconnection]] — A real-time client survives disconnects by tracking the last sequence number or cursor it has applied, reconnecting with jittered backoff, presenting a resume token…

### C. Databases — [[Databases MOC]]

*Where the state lives, what the database promises, and how it outgrows one machine.*

- [[SQL vs NoSQL]] — A decision framework for picking a datastore from data shape, query patterns, consistency needs, scale path and operational maturity rather than from fashion
- [[Relational Database Internals]] — Pages, the buffer pool, the write-ahead log, checkpoints and MVCC are the five mechanisms that explain almost every performance and durability behaviour a relational…
- [[Indexing]] — An index is a sorted, separately maintained structure that turns a scan into a handful of page reads, paid for with extra work on every write and extra space on disk
- [[Query Execution and Optimization]] — A query becomes a tree of physical operators chosen by a cost model fed with table statistics
- [[Transactions]] — A transaction groups several reads and writes into one unit that either fully commits or fully rolls back
- [[Isolation Levels]] — Isolation levels are a menu of which concurrency anomalies the database will prevent for you, from read committed through snapshot isolation to serializable, each buying…
- [[Replication]] — Replication keeps copies of the same data on several nodes for availability, durability and read scale
- [[Read Replicas]] — Read replicas scale reads by routing queries to asynchronously replicated followers
- [[Sharding]] — Sharding splits one logical dataset across independent database nodes by a shard key so writes and storage scale horizontally, at the cost of losing cross-shard joins…
- [[Partitioning]] — Partitioning divides a dataset into independently managed pieces by range, hash or list so that queries touch only the relevant piece and pieces can be moved, dropped…
- [[Consistent Hashing]] — Consistent hashing places nodes and keys on the same hash ring so that adding or removing a node remaps only the keys adjacent to it, roughly 1/N of the total, instead…
- [[Database Scaling]] — Database scaling is a ladder climbed in order, tune and index, cache, read replicas, scale up, split by function, shard, because each rung is cheaper and more reversible…
- [[Distributed Transactions]] — Two-phase commit makes a transaction atomic across several nodes by having a coordinator collect prepare votes before telling everyone to commit
- [[Optimistic and Pessimistic Locking]] — Pessimistic locking takes a row lock before reading so conflicting writers wait, optimistic locking reads freely and rejects a write whose version changed
- [[Data Modeling]] — Data modelling is deciding what facts you store, where each fact lives once
- [[Schema Evolution]] — Schema evolution changes a live database without downtime by expanding the schema so old and new code both work, migrating data in throttled batches with dual writes

### D. Caching — [[Caching MOC]]

*The highest-leverage performance tool and the fastest way to serve wrong data.*

- [[Cache-Aside]] — Cache-aside has the application check the cache first, load from the database on a miss and populate the cache itself
- [[Write-Through Caching]] — Write-through caching sends every write through the cache to the store synchronously
- [[Write-Back Caching]] — Write-back caching acknowledges a write once the cache has it and flushes it to the store later, in batches and with repeated updates to the same key coalesced
- [[TTL]] — A TTL bounds how stale a cached value can be by discarding it after a fixed time
- [[Eviction Policies]] — An eviction policy decides which entry to drop when a cache is full
- [[Cache Invalidation]] — Cache invalidation is deciding when a cached copy stops being served after the source changes
- [[Redis]] — Redis is an in-memory data-structure server that executes commands one at a time on a single thread
- [[Distributed Caching]] — A distributed cache spreads keys across many nodes so that memory and throughput scale out
- [[Cache Stampedes]] — A cache stampede is many concurrent requests missing the same key at once and each recomputing it from the origin
- [[Hot Keys]] — A hot key is one key, partition or shard that receives a disproportionate share of traffic
- [[Cache Consistency]] — A cache is a replica with a weak consistency model: it can offer bounded staleness
- [[Multi-Level Caching]] — Multi-level caching stacks caches from the browser and CDN through the gateway, the in-process cache and the distributed cache down to the database buffer pool, each…

### E. Messaging & Event-Driven Systems — [[Messaging & Event-Driven Systems MOC]]

*Your deepest area at work: queues, logs, delivery guarantees and the patterns that make async safe.*

- [[Kafka]] — Kafka is a partitioned, replicated, append-only log that retains records by time rather than by consumption
- [[Message Queues]] — A message queue is a point-to-point buffer where competing consumers take messages, acknowledge them after processing
- [[Pub-Sub]] — Publish-subscribe decouples a producer from an unknown number of independent subscribers by routing one published event through a topic to every subscription, each…
- [[Consumer Groups]] — A consumer group is a set of processes that share the partitions of a topic
- [[Topic Partitions]] — A partition is the unit of ordering, parallelism and placement in a log
- [[Ordering Guarantees]] — Messaging systems promise order only per key within one partition or message group, never globally
- [[Delivery Semantics]] — Delivery semantics describe whether a message can be lost, duplicated or neither on each hop
- [[At-Most-Once Delivery]] — At-most-once delivery acknowledges or forgets a message before its effect is secured
- [[At-Least-Once Delivery]] — At-least-once delivery retries until acknowledged and acknowledges only after the work is durable
- [[Exactly-Once Semantics]] — Exactly-once delivery over an unreliable network is impossible, but exactly-once effects are achievable
- [[Idempotency]] — An idempotent operation produces the same final state whether it runs once or many times
- [[Retry Strategies]] — A retry strategy decides which failures to retry, how many times, how far apart, where the message waits meanwhile and what happens when attempts run out
- [[Dead-Letter Queues]] — A dead-letter queue quarantines messages that keep failing so they stop blocking healthy work and are preserved with their failure context for triage and redrive
- [[Backpressure]] — Backpressure is how a slow stage tells a fast one to slow down
- [[Event Schemas]] — An event schema is the published contract of an event — its meaning as a fact or command, the envelope of ids, timestamps and versions every consumer relies on, the…
- [[Event Schema Evolution]] — Event schema evolution is changing an event's contract while old events are still retained and old consumers still run
- [[Outbox Pattern]] — Write the event into an outbox table inside the same database transaction as the state change, then relay it to the broker
- [[Change Data Capture]] — Change data capture turns a database's own replication log into an ordered stream of row-level change events
- [[Synchronous vs Asynchronous Communication]] — Synchronous calls couple the caller to the callee's availability and latency in exchange for an immediate answer
- [[Fan-Out]] — Fan-out is delivering one event or item to many recipients — precomputed at write time, assembled at read time, or copied by a broker — and the choice sets the cost shape
- *Also in this section:* [[Schema Evolution]] (canonical note lives elsewhere)

### F. Distributed Systems — [[Distributed Systems MOC]]

*The rules of the game once there is more than one machine.*

- [[Leader Election]] — Pick exactly one node to coordinate for a bounded lease, via a quorum or a lock service
- [[Consensus]] — Consensus lets a group of unreliable nodes agree on one sequence of decisions despite crashes and partitions
- [[Distributed Locks]] — A distributed lock is a lease on a name held in a shared store
- [[Failure Detection]] — A failure detector turns silence into a decision by timing out heartbeats
- [[Eventual Consistency]] — Eventual consistency promises only that replicas converge once writes stop
- [[Distributed ID Generation]] — Generate unique identifiers on many nodes without a coordination round trip by combining time, a node id and a sequence (Snowflake, UUIDv7, ULID) or by using enough…
- [[Clocks and Time]] — Wall clocks on different machines disagree and jump, so ordering across machines must come from logical clocks, versioning or a single arbiter, while wall time is kept…
- [[Gossip Protocols]] — Gossip spreads membership and state by having each node periodically exchange what it knows with a few random peers, reaching every node in logarithmic rounds with no…
- [[Quorums]] — A quorum is any set of replicas large enough that two such sets must overlap
- [[Split-Brain]] — Split-brain is two nodes both acting as the single authority at once, caused by a partition or a pause outliving a lease
- [[Network Partitions]] — A partition is a network fault that leaves nodes alive but unable to reach each other
- [[Retry Storms]] — A retry storm is the feedback loop in which retries from clients and intermediate layers multiply load on an already struggling dependency, converting a brief slowdown…
- [[Duplicate Message Processing]] — Duplicates are unavoidable wherever a sender cannot tell whether its message arrived
- [[Coordination vs Availability]] — Every point where nodes must agree before acting — a lock, a leader, a transaction, a quorum — costs latency on every operation and availability whenever the agreeing…
- *Also in this section:* [[Replication]] (canonical note lives elsewhere)

### G. Architecture Patterns — [[Architecture Patterns MOC]]

*How services are shaped, split and protected.*

- [[Monoliths]] — One deployable, one process, one database: the cheapest architecture to build, test and operate
- [[Modular Monoliths]] — A single deployable whose internal modules have explicit public interfaces, enforced dependency rules and private data, giving you service-like boundaries without…
- [[Microservices]] — Independently deployable services that each own their data and talk over the network, trading in-process simplicity for team autonomy, independent scaling and a…
- [[Event-Driven Architecture]] — Services publish immutable facts about what happened and other services react to them, removing call-time coupling between producer and consumer at the price of eventual…
- [[CQRS]] — Separate the model that handles writes from the models that serve reads
- [[Event Sourcing]] — Store every state change as an immutable event in an append-only log and derive current state by replay
- [[API Gateways]] — A single entry point that routes client requests to backend services and handles cross-cutting edge concerns once
- [[Service Discovery]] — A registry of healthy service instances and their current addresses, consulted by clients or by a proxy
- [[Circuit Breakers]] — A per-dependency switch that stops calling a dependency once it is failing beyond a threshold, fails fast for a cooling period, then probes before resuming
- [[Rate Limiting]] — Bound how many requests a caller or a system accepts per unit of time, using token bucket, leaky bucket or window counters, to protect capacity, enforce fairness…
- [[Bulkheads]] — Partition capacity — threads, connections, queues, instances or whole cells — so that a failure or overload in one partition cannot consume the resources of the others…
- [[Saga Pattern]] — A long-running business transaction spanning services is executed as a sequence of local transactions, each with a compensating action that semantically undoes it
- [[Strangler Pattern]] — Replace a legacy system incrementally by putting a facade in front of it and routing one slice of functionality at a time to a new implementation, until the old system…
- [[Hexagonal Architecture]] — Keep domain logic in the centre with no knowledge of HTTP, databases or brokers
- [[Domain-Driven Design]] — Model software around the business domain using a shared language, split it into bounded contexts with their own models
- [[Separation of Concerns]] — Give each part of a system one responsibility and one reason to change, keep related things together and unrelated things apart
- [[Designing for Extensibility]] — Make the changes you can reasonably predict cheap — through stable contracts, explicit extension points, configuration and versioned interfaces — while refusing to build…
- [[Real-Time Systems]] — Systems that push updates to connected clients are a chain — persistent connections, an event stream, fan-out, backpressure, client state, ordering, reconnection…

### H. Storage & Search — [[Storage & Search MOC]]

*Where bytes live when they are not rows: objects, files, blocks, warehouses, and search.*

- [[Object Storage]] — Object storage keeps immutable blobs in a flat key namespace behind an HTTP API, engineered for eleven-nines durability through replication or erasure coding across…
- [[Blob Storage]] — A blob is a large opaque byte value — a PDF, image or archive — and the design question is where its bytes live
- [[File Systems]] — A file system gives a hierarchical, mutable, POSIX view over storage
- [[Block Storage]] — Block storage presents a raw, addressable array of fixed-size blocks to a single host as a disk
- [[Data Warehouses]] — A data warehouse is a separate analytical store that lays data out by column, compresses it heavily and scans billions of rows in seconds for aggregate queries, fed from…
- [[Data Lakes]] — A data lake lands raw data in open file formats on object storage and applies schema on read
- [[Time-Series Databases]] — A time-series database stores append-mostly, timestamp-ordered samples per labelled series, compresses them with delta and XOR encoding to about a byte or two per…
- [[Search Engines]] — A search engine such as Elasticsearch or OpenSearch is a separate, Lucene-based system that keeps a denormalised, analysed copy of your data in sharded and replicated…
- [[Full-Text Search]] — Full-text search turns documents and queries into normalised tokens through an analysis chain, matches them via an inverted index
- [[Inverted Indexes]] — An inverted index maps each term to a sorted posting list of the documents containing it
- [[Search Indexing Pipelines]] — A search indexing pipeline moves changes from the source of truth into the search index
- [[Data Retention and Archival]] — Data retention and archival is the explicit policy and machinery deciding how long each class of data lives in which tier

### I. Observability & Reliability — [[Observability & Reliability MOC]]

*You cannot fix what you cannot see — and you cannot promise what you do not measure.*

- [[Logging]] — Logs are timestamped, structured records of discrete events, correlated by request and trace ids and shipped to a central store
- [[Metrics]] — Metrics are cheap, pre-aggregated numeric time series — counters, gauges and histograms with a bounded set of labels — that answer how the system is behaving right now…
- [[Tracing]] — Tracing records one request as a tree of timed spans — each operation with its start, duration, parent and attributes — so you can see where a request's time went and why
- [[Monitoring]] — Monitoring is the practice of continuously answering whether the system is healthy from the user's point of view
- [[Alerting]] — Alerting interrupts a human only when users are being hurt fast enough to need one, using SLO burn-rate rules over multiple windows for pages and tickets for everything…
- [[Distributed Tracing]] — Distributed tracing carries a trace id and parent span id across every process boundary
- [[Error Budgets]] — An error budget is the unreliability an SLO permits over its window, expressed as a quantity you can spend
- [[Health Checks]] — Health checks are the endpoints and probes by which load balancers and orchestrators decide whether to send traffic to an instance (readiness) or restart it (liveness)
- [[Graceful Degradation]] — Graceful degradation keeps a system serving something useful under overload or dependency failure by shedding excess load early, prioritising important work
- [[Disaster Recovery]] — Disaster recovery is the planned, rehearsed ability to restore service after losing a zone, a region or a dataset, choosing among backup-restore, pilot light, warm…
- [[Backups and Restoration]] — A backup is a copy of data isolated from the failures that threaten the original
- [[Recovery Time Objective]] — The recovery time objective is the maximum tolerable time from the start of a disruption to restored service, set per system from business impact
- [[Recovery Point Objective]] — The recovery point objective is the maximum acceptable amount of data loss, measured as time, after a disaster
- [[Capacity Planning]] — Capacity planning is knowing each component's ceiling before traffic finds it
- [[Production Incident Analysis]] — Production incident analysis is the structured, blameless reconstruction of what happened, why the system allowed it

### J. Security — [[Security MOC]]

*Who you are, what you may do, and how secrets and data stay protected.*

- [[Authentication]] — Authentication establishes who is calling, for people with hashed passwords, MFA and sessions or tokens
- [[Authorization]] — Authorization decides whether an authenticated principal may perform an action on a resource, using roles, attributes or relationships evaluated by a policy decision…
- [[OAuth]] — OAuth 2.0 lets a client obtain a scoped access token from an authorization server on behalf of a resource owner without ever seeing their password, OpenID Connect adds a…
- [[JWT]] — A JWT is a signed, self-contained set of claims that any service can verify locally with the issuer's public key
- [[API Keys]] — An API key is a high-entropy shared secret that identifies a machine client, stored only as a hash, scoped and rate-limited per key, rotated with overlap
- [[Encryption at Rest and in Transit]] — Encryption in transit (TLS) protects data between endpoints and encryption at rest (volumes, databases, objects, fields) protects stored data from whoever gets the…
- [[Secrets Management]] — Secrets management keeps credentials out of code, images and tickets by storing them in an encrypted, access-controlled store, injecting them into workloads at run time…
- [[Least Privilege]] — Least privilege gives every identity, human or workload, exactly the permissions its job needs and nothing more, across cloud IAM, Kubernetes, the network and the…
- [[Input Validation]] — Input validation rejects everything that crosses a trust boundary unless it matches an explicit schema of type, size, format and allowed values
- [[Audit Logs]] — An audit log is an append-only, tamper-evident record of who did what to which resource, when, from where and with what outcome, written reliably as part of the action…
- [[Threat Modeling]] — Threat modelling is the structured habit of drawing the system with its trust boundaries, asking what can go wrong at each crossing using a checklist such as STRIDE…
- *Also in this section:* [[Rate Limiting]] (canonical note lives elsewhere)

### K. Product Engineering — [[Product Engineering MOC]]

*The point of all of this: shipping products that work, change safely and stay cheap to run.*

- [[Translating Product Requirements into Technical Requirements]] — Turn a product ask into a buildable, testable spec by asking the clarifying questions, walking the user journey into data flows, surfacing the implicit scale, latency…
- [[Functional vs Non-Functional Requirements]] — Functional requirements say what the system does; non-functional requirements say how well
- [[Prioritizing Reliability Performance Cost and Time to Market]] — Every engineering decision sits in a four-way tension between reliability, performance, cost and time to market
- [[Designing APIs and Contracts]] — An API is a contract and a product surface: model resources around the caller's job, define errors, pagination, idempotency and consistency promises explicitly, document…
- [[Data Ownership and Lifecycle]] — Every piece of data has exactly one owning service that is its source of truth and controls its creation, change and deletion
- [[Backward Compatibility]] — A change is backward compatible when every existing caller, consumer and stored record keeps working unchanged
- [[Versioning and Migrations]] — When a change cannot be made compatible you version the contract and migrate the data in phases
- [[Feature Flags]] — A feature flag is a runtime switch that decouples deploying code from releasing behaviour
- [[Incremental Delivery]] — Deliver a system as a sequence of thin, end-to-end slices — a walking skeleton first, then one user-visible increment at a time, each merged to trunk in small PRs…
- [[Designing for Maintainability]] — Maintainability is designing for the next engineer: small modules with clear boundaries, code that reads like the domain, documentation that lives beside the code…
- [[Testing Strategies]] — A testing strategy decides what to test at which level — unit for logic, integration for real dependencies, contract for service boundaries, a few end-to-end for the…
- [[Deployment and Rollback Strategies]] — Rolling, blue-green, canary and shadow deployments are different ways to expose a new version to traffic while keeping a fast path back
- [[Monitoring User-Facing Performance]] — Measure performance where the user feels it — real-user monitoring from the client, synthetic checks from outside, p99 at the edge rather than the service — set budgets…
- [[Operational Simplicity]] — Operational simplicity treats on-call load and the number of moving parts as design inputs
- [[Technical Debt]] — Technical debt is the future cost of a shortcut taken today, deliberate or accidental, prudent or reckless
- [[Build vs Buy Decisions]] — Build what differentiates your product and buy or adopt everything else
- [[Evaluating Architectural Complexity]] — Separate the complexity the problem demands from the complexity the solution added, count the moving parts and the failure modes each one brings
- [[Designing Systems for Changing Requirements]] — Assume requirements will change and design so that change is cheap
- *Also in this section:* [[Graceful Degradation]] (canonical note lives elsewhere)

## Progress across the curriculum

Every concept note carries `status` (not-started → reading → understood → applied, or needs-revision) and `confidence` (1 unfamiliar … 5 can defend design decisions). The views below are native Bases; the [[Engineering Dashboard]] has the full set.

![[Concepts.base#By category]]

## Where the concepts go next

- **Apply** them in [[Problem Bank MOC]] — 28 design problems, each a decision sheet rather than a model answer.
- **Build** them in [[Projects MOC]] — 16 projects across backend, distributed, real-time, fintech, AI, dev tools, productivity and experimental.
- **Go deeper** with [[Resources MOC]] — primary docs, papers, engineering write-ups and books, each linked to the concepts it explains.
- **Review** with the weekly template in `Templates/` and the revision queue on the dashboard.
