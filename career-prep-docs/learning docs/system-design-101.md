# System Design 101 — HLD for SDE2

> **Who this is for**: Engineers preparing for SDE2 system design interviews (45–60 min). Every concept is built from first principles so you can reason about trade-offs, not just recite patterns.

---

## Table of Contents

1. [The Interview Framework](#1-the-interview-framework)
2. [Back-of-Envelope Estimation](#2-back-of-envelope-estimation)
3. [Core Properties — Scalability, Reliability, Availability](#3-core-properties)
4. [CAP Theorem and Consistency Models](#4-cap-theorem-and-consistency-models)
5. [Networking Fundamentals](#5-networking-fundamentals)
6. [DNS and CDN](#6-dns-and-cdn)
7. [Load Balancing](#7-load-balancing)
8. [Caching](#8-caching)
9. [Databases Deep Dive](#9-databases-deep-dive)
10. [Database Scaling — Replication and Sharding](#10-database-scaling)
11. [Message Queues and Event Streaming](#11-message-queues-and-event-streaming)
12. [API Design](#12-api-design)
13. [Microservices Patterns](#13-microservices-patterns)
14. [Distributed Systems Primitives](#14-distributed-systems-primitives)
15. [Storage Systems](#15-storage-systems)
16. [Search Systems](#16-search-systems)
17. [Rate Limiting](#17-rate-limiting)
18. [Concurrency and Locking](#18-concurrency-and-locking)
19. [Security Fundamentals](#19-security-fundamentals)
20. [Observability in System Design](#20-observability-in-system-design)
21. [Classic Design Problems — Patterns and Solutions](#21-classic-design-problems)
22. [Common Mistakes in Interviews](#22-common-mistakes-in-interviews)
23. [Glossary](#23-glossary)
24. [Further Reading](#24-further-reading)

---

## 1. The Interview Framework

System design interviews test your ability to **think through ambiguity, communicate trade-offs, and produce a workable architecture under constraints**. No one expects a perfect answer — they expect a structured thinker.

### The RADIO Framework (45-min interview)

| Phase | Time | What you do |
|---|---|---|
| **R**equirements | 5 min | Clarify scope — functional + non-functional |
| **A**PI Design | 5 min | Define contracts before the box diagram |
| **D**ata Model | 5 min | Schema, access patterns, write vs read ratio |
| **I**nitial Design | 10 min | Draw the happy path end-to-end |
| **O**ptimize | 20 min | Deep-dive one or two bottlenecks |

### Requirements Gathering — The Questions You Must Ask

**Functional (what the system does):**
- Who are the users? (consumers, businesses, internal engineers)
- What are the core operations? (create, read, search, stream?)
- Are there secondary features? (notifications, analytics, admin)
- What does success look like for a single user action?

**Non-functional (how the system behaves):**
```
Scale:        How many DAU? Reads/writes per second at peak?
Latency:      p99 < 200ms? Real-time or eventually consistent?
Availability: 99.9% (8.7 hr downtime/yr) or 99.99% (52 min/yr)?
Durability:   Can we lose data? (financial = no, analytics = maybe)
Geography:    Single region or global?
Consistency:  Strong (banking) or eventual (social feed)?
```

### How to Estimate at the Start

Turn DAU into QPS:
```
100M DAU
Assume 10 actions/day per user
→ 1B actions/day
→ 1B / 86400 ≈ 11,500 QPS average
→ peak = 3x → ~35,000 QPS
```

Storage estimate:
```
1M users × 100 bytes metadata = 100 MB
1M users × 1 profile photo (200 KB avg) = 200 GB
1M posts/day × 1 KB per post × 365 = 365 GB/yr
```

Bandwidth:
```
100 GB data × 8 bits = 800 Gbps
But you serve 1M users, so: 100 GB / 86400 sec = ~1.2 GB/s outbound
```

Memorize these numbers:
```
1 KB  = 10^3 bytes
1 MB  = 10^6 bytes
1 GB  = 10^9 bytes
1 TB  = 10^12 bytes

1 million = 10^6
1 billion = 10^9

1 day ≈ 86,400 seconds (~100K)
1 month ≈ 2.5M seconds
1 year ≈ 31.5M seconds

Ping same DC:  ~0.5 ms
Ping cross-DC: ~5 ms
Ping cross-country: ~50 ms
Disk seek: ~5 ms (SSD ~0.1 ms)
Memory read: ~100 ns
Network round trip (LAN): ~0.5 ms
```

---

## 2. Back-of-Envelope Estimation

### Powers of 2 Cheat Sheet

```
2^10 = 1,024       ≈ 1 thousand (1 KB)
2^20 = 1,048,576   ≈ 1 million (1 MB)
2^30 ≈ 1 billion   ≈ 1 GB
2^40 ≈ 1 trillion  ≈ 1 TB
```

### Real Estimation Example — Twitter Timeline

**Given**: 300M DAU, each user tweets 1x/day, reads 100 tweets/day

```
Writes:
300M tweets/day
= 300M / 86400 = ~3,500 writes/sec
Each tweet = 280 chars ≈ 300 bytes (+ metadata) ≈ 1 KB
Storage: 300M KB/day = 300 GB/day = ~100 TB/year

Reads:
300M × 100 = 30B reads/day = ~350,000 reads/sec
Read:Write ratio = 100:1 → read-heavy system → cache everything

Fanout (home timeline delivery):
Average followers = 200
Popular users = 10M followers
3,500 writes/sec × 200 = 700,000 timeline insertions/sec
→ separate path for celebrities (pull instead of push)
```

---

## 3. Core Properties

### Scalability

**Vertical scaling (scale up)**: Add more CPU/RAM/disk to one machine.
- Pros: Simple, no code change, no distributed systems complexity
- Cons: Hard limit (biggest machine AWS offers), single point of failure, expensive, no zero-downtime upgrade

**Horizontal scaling (scale out)**: Add more machines, distribute load.
- Pros: Near-infinite, fault tolerant, commodity hardware
- Cons: Stateless services easy; stateful (databases) much harder — need sharding, coordination

Rule of thumb: **scale vertically first, then horizontally**. Don't introduce distributed systems complexity before you need to.

### Reliability

A system is reliable if it continues to work correctly even when hardware fails, software crashes, or the operator makes a mistake.

Mechanisms:
- Redundancy: no single point of failure (N+1 replicas)
- Checksums: detect corruption on disk or network
- Retries with idempotency: retry without side effects
- Isolation: failures in one component don't cascade

### Availability

```
Availability (%) = Uptime / (Uptime + Downtime) × 100

The "nines":
99%      → 87.6 hr downtime/year   → barely acceptable
99.9%    →  8.7 hr downtime/year   → most web services
99.99%   → 52 min downtime/year    → payment systems
99.999%  →  5 min downtime/year    → telecom/banking
```

**Availability of systems in series** (both must work):
```
A = 0.99, B = 0.99
Combined = 0.99 × 0.99 = 0.9801 = 98.01%
Add more services → availability degrades fast
```

**Availability of systems in parallel** (either must work):
```
A = 0.99, B = 0.99
Combined = 1 - (1 - 0.99) × (1 - 0.99) = 1 - 0.0001 = 99.99%
Redundancy dramatically improves availability
```

### Durability

Data is not lost once written. Separate from availability.

A system can be:
- Available but not durable: in-memory cache (Redis without AOF) — responds fast but data lost on restart
- Durable but unavailable: database during maintenance — data is safe but you can't read it

Durability mechanisms: replication, WAL (write-ahead log), checksums, backups, RAID.

### Latency vs Throughput

**Latency**: time for one operation (p99 < 200ms)
**Throughput**: operations per unit time (10,000 QPS)

They are not the same and often conflict:
- Batching increases throughput but increases latency (you wait to batch)
- Caching reduces latency but requires eviction logic
- Async processing increases throughput but makes latency unpredictable

**Little's Law**: In a stable system:
```
L = λ × W
L = avg number of items in system
λ = avg arrival rate (throughput)
W = avg time in system (latency)

If QPS doubles and latency stays the same → queue depth doubles.
If queue grows unbounded → system is overloaded.
```

---

## 4. CAP Theorem and Consistency Models

### CAP Theorem

In a distributed system, you can have at most **two** of:

| Property | Meaning |
|---|---|
| **C**onsistency | Every read gets the most recent write |
| **A**vailability | Every request gets a (non-error) response |
| **P**artition Tolerance | System continues when network splits |

**Partition tolerance is non-negotiable** in any real distributed system — networks fail. So the real choice is **CP vs AP**:

| System type | During a network split... | Examples |
|---|---|---|
| **CP** (consistent) | Reject requests rather than serve stale data | HBase, Zookeeper, etcd, MongoDB (majority reads) |
| **AP** (available) | Serve possibly stale data rather than error | Cassandra, CouchDB, DynamoDB (eventual consistency) |

### PACELC — The Better Model

CAP only covers partitioned scenarios. PACELC covers the normal case too:

```
If Partition:   choose A (availability) or C (consistency)
Else (normal):  choose L (latency) or C (consistency)
```

| System | Partition choice | Normal choice |
|---|---|---|
| DynamoDB | A | L (eventual) |
| Cassandra | A | L (tunable) |
| MongoDB | C (by default) | C |
| PostgreSQL | C | C |

### Consistency Models (from weakest to strongest)

**Eventual consistency**: If no new updates, all replicas will eventually converge.
- Best for: likes, view counts, shopping cart (Amazon famously chose AP)
- Risk: users may see stale data temporarily

**Causal consistency**: Operations that are causally related appear in order.
- Example: you see replies to a post only after seeing the post itself
- Implemented via vector clocks

**Read-your-writes consistency**: After you write, you always see your own write.
- Example: after posting a tweet you see it immediately
- Implemented by routing your reads to the replica you wrote to (sticky sessions) or tagging writes with a timestamp

**Monotonic reads**: Once you read a value, you never read an older value.
- Without this: you refresh a page and see older content

**Linearizability (strong consistency)**: Every operation appears to be instantaneous. The system behaves like a single machine.
- Best for: financial transactions, inventory, distributed locks
- Cost: higher latency, lower availability

### How to Choose in an Interview

```
Ask: "What happens if a user reads stale data?"
Financial transfer → stale = disaster → strong consistency (CP)
Social media like count → stale = fine → eventual consistency (AP)
User profile → stale for a few seconds = fine → read-your-writes
Inventory → depends on cost of overselling → usually strong
```

---

## 5. Networking Fundamentals

### TCP vs UDP

| | TCP | UDP |
|---|---|---|
| Connection | 3-way handshake | Connectionless |
| Delivery | Guaranteed, ordered | Best-effort, no order |
| Error recovery | Retransmit lost packets | None (application handles) |
| Speed | Slower (ACKs, flow control) | Faster |
| Use cases | HTTP, databases, email | Video streaming, DNS, gaming, VoIP |

**Why does this matter in system design?**
- Kafka uses TCP — reliable delivery, ordering per partition
- UDP-based protocols (QUIC/HTTP3) reduce connection overhead for mobile

### HTTP Versions

**HTTP/1.1** (1997):
- One request per TCP connection (or pipelining — but broken in practice)
- Head-of-line blocking: request 2 blocked by request 1
- Text-based headers (verbose, no compression)

**HTTP/2** (2015):
- Multiplexing: multiple streams over one TCP connection
- Header compression (HPACK)
- Server push
- But: TCP head-of-line blocking remains (one lost packet blocks all streams)

**HTTP/3** (2022):
- Built on QUIC (UDP-based, custom reliability)
- Eliminates TCP head-of-line blocking
- Faster handshake (0-RTT reconnect)
- Ideal for mobile (connection survives IP change)

### Long Polling, WebSockets, SSE

| Mechanism | How | Best for |
|---|---|---|
| **Short polling** | Client asks every N seconds | Simple, but wasteful |
| **Long polling** | Client asks, server holds until event | Notifications (compatible with any HTTP) |
| **WebSockets** | Bidirectional persistent connection | Chat, multiplayer gaming, collaborative editing |
| **Server-Sent Events (SSE)** | Server pushes, one-way | Live scores, stock prices, dashboards |

**WebSocket handshake**:
```
Client → GET /chat HTTP/1.1
         Upgrade: websocket
         Connection: Upgrade
Server → 101 Switching Protocols
Now both can send at any time
```

SSE is simpler than WebSockets and works over plain HTTP. Use SSE when you only need server→client push. Use WebSockets when you need bidirectional real-time communication.

---

## 6. DNS and CDN

### DNS (Domain Name System)

DNS translates domain names to IP addresses. It is hierarchical and cached at multiple levels.

```
User types: api.niyo.com
1. Browser checks local DNS cache
2. OS checks /etc/hosts
3. OS asks local resolver (ISP or 8.8.8.8)
4. Resolver asks Root nameserver → returns .com TLD nameserver
5. Resolver asks .com TLD nameserver → returns niyo.com authoritative nameserver
6. Resolver asks niyo.com nameserver → returns IP 52.1.2.3
7. Browser connects to 52.1.2.3
```

**TTL (Time to Live)**: How long resolvers cache the response. Low TTL = faster propagation of changes but more DNS load. High TTL = fewer DNS queries but slow failover.

**DNS-based load balancing**:
- Round-robin DNS: return multiple A records, rotate
- Geo-DNS: return different IPs based on requester's location
- Health-aware DNS: Route53 health checks — stop returning unhealthy IP

**DNS record types**:
```
A     → domain to IPv4         niyo.com → 1.2.3.4
AAAA  → domain to IPv6
CNAME → domain to domain       www.niyo.com → niyo.com
MX    → mail server
TXT   → arbitrary text (SPF, DKIM, ownership proofs)
NS    → nameserver for domain
```

### CDN (Content Delivery Network)

A CDN is a globally distributed network of caches (PoPs — Points of Presence) that serve content from the edge closest to the user.

**Why**: A user in Mumbai hitting a server in us-east-1 has ~200ms latency. The same user hitting a Mumbai PoP has ~5ms.

**What to cache on CDN**:
- Static assets: JS, CSS, images, fonts
- Public API responses (product catalog, public profiles)
- Video chunks, podcast audio

**What NOT to cache**:
- Authenticated API responses with user-specific data (unless keyed on auth token)
- One-time tokens, OTPs

**CDN Push vs Pull**:

| | Push | Pull |
|---|---|---|
| How | You upload assets to CDN ahead of time | CDN fetches from origin on first miss |
| Best for | Large static files you know upfront | Dynamic content, frequently changing assets |
| Example | Game builds, app installers | Images, JS bundles |

**Cache invalidation on CDN**:
- Set short TTL and let it expire naturally
- Use versioned URLs: `main.a1b2c3.js` — deploy new hash, old URL stays cached
- CDN purge API (slow to propagate, not reliable for urgent fixes)

---

## 7. Load Balancing

A load balancer distributes incoming requests across multiple backend servers. It is the entry point into your system.

### Why Load Balancers?

1. Distribute traffic — prevent any one server from being overwhelmed
2. Health checks — route around failed servers automatically
3. SSL termination — decrypt TLS once at the LB, keep backend communication plain HTTP
4. Session persistence — route same user to same server if needed
5. Zero-downtime deploys — drain server, deploy, re-add

### L4 vs L7 Load Balancers

**L4 (Transport Layer)**:
- Operates on TCP/UDP — sees source/dest IP + port only
- Very fast, low overhead
- Cannot make routing decisions based on HTTP content
- Example: AWS NLB (Network Load Balancer)

**L7 (Application Layer)**:
- Operates on HTTP — sees headers, URL, cookies, body
- Can route: `/api/*` → API servers, `/static/*` → CDN or static servers
- Can rewrite headers, inject trace IDs, rate limit per user
- Slightly more overhead (parses HTTP)
- Example: AWS ALB, Nginx, HAProxy, Envoy

**In practice**: L7 everywhere for HTTP services. L4 for databases, game servers, anything non-HTTP.

### Load Balancing Algorithms

**Round Robin**: requests go to each server in turn (1→2→3→1→2→3...)
- Good when servers are identical
- Ignores server load — a slow server gets as many requests as a fast one

**Weighted Round Robin**: assign weights (server1=3, server2=1) — 3 of 4 requests go to server1
- Use when servers have different capacities

**Least Connections**: send to server with fewest active connections
- Better than round robin for long-lived connections (WebSockets, uploads)

**IP Hash**: hash client IP → deterministic server selection
- Same client always hits same server (session affinity)
- Problem: uneven distribution if many users share an IP (NAT)

**Least Response Time**: send to server with fastest recent response + fewest connections
- Most intelligent but requires monitoring

**Consistent Hashing**: hash the request key to a position on a ring, find nearest server
- Used in distributed caches (Redis Cluster), Cassandra, CDNs
- Adding/removing servers only moves a fraction of keys (no full reshuffling)

### Health Checks

LBs continuously probe backends:
- **Passive**: mark server unhealthy when requests fail
- **Active**: LB sends periodic health check requests (GET /health)

```
Active health check: every 10s send GET /health
  2xx → healthy
  timeout / 5xx → mark unhealthy, stop sending traffic
  After 3 consecutive successes → mark healthy again
```

### The Load Balancer is Also a SPOF

Solution: HA pair — two LBs with virtual IP (VIP). VRRP protocol elects primary. If primary dies, secondary takes over the VIP in seconds.

AWS handles this for you: ALB/NLB are managed and horizontally scaled.

---

## 8. Caching

Cache is a fast, temporary data store that reduces latency and backend load by serving frequently accessed data from memory.

**Rule of thumb**: 20% of data serves 80% of reads (Pareto principle). Cache that 20%.

### Where Caches Live

```
Browser Cache    → HTML, JS, CSS cached on user's device
CDN              → Static + public content at edge
API Gateway      → Rate limit counters, auth token caches
Application      → In-process cache (Caffeine, Guava) — per instance, no network hop
Distributed      → Redis, Memcached — shared across all instances
Database         → Query cache, buffer pool (MySQL InnoDB buffer pool)
```

### Cache Read Strategies

**Cache-aside (lazy loading)** — most common:
```
1. App checks cache for key
2. Cache hit → return cached value
3. Cache miss → fetch from DB, write to cache, return value

Pros: Only populate what's needed. Cache failure is non-fatal.
Cons: First request always slow (cold start). Cache can hold stale data.
```

**Read-through**:
```
App always reads from cache.
On miss, cache itself fetches from DB and populates.
App doesn't talk to DB directly.

Pros: App code simpler (single read path)
Cons: First request slow. Need cache library that supports this.
```

### Cache Write Strategies

**Write-through**:
```
App writes to cache AND DB synchronously.
Cache always up-to-date.

Pros: Cache is never stale.
Cons: Every write hits both cache and DB — write latency is DB latency.
      Wastes cache space (write data that may never be read).
```

**Write-behind (write-back)**:
```
App writes to cache only.
Cache asynchronously flushes to DB in background.

Pros: Write latency = cache write (fast). Batch DB writes.
Cons: Data loss risk if cache fails before flush.
      Complex to implement correctly.
```

**Write-around**:
```
App writes directly to DB, bypasses cache.
Cache only populated on read (cache-aside).

Use when: data is written once and rarely read (logs, audit trails).
```

### Cache Eviction Policies

When cache is full, which entry to evict?

| Policy | Description | Best for |
|---|---|---|
| **LRU** (Least Recently Used) | Evict entry not accessed the longest | General purpose — most common |
| **LFU** (Least Frequently Used) | Evict entry accessed fewest times | When access frequency matters more than recency |
| **FIFO** | Evict oldest inserted entry | Simple, but poor cache hit rate |
| **Random** | Evict randomly | Simple, surprisingly effective |
| **TTL** | Evict after fixed time, regardless of access | Time-sensitive data (OTPs, sessions) |

### Cache Invalidation — The Hardest Problem

"There are only two hard things in Computer Science: cache invalidation and naming things." — Phil Karlton

Strategies:
1. **TTL-based**: set expiry on write. Simplest. Stale window = TTL duration.
2. **Event-driven invalidation**: when DB record changes, publish event → invalidate cache key.
3. **Version tags**: store version number. On DB change, increment version. Cache key = `user:123:v5`.
4. **Write-through**: cache is always updated on write — no staleness.

### Cache Stampede (Thundering Herd)

Problem: A popular cache key expires. Simultaneously, 1000 requests miss the cache and all try to fetch from DB.

Solutions:
1. **Lock with early recompute**: one request gets a lock, recomputes, others wait or serve stale
2. **Probabilistic early expiration**: before TTL expires, randomly start refreshing early
3. **Background refresh**: a background job refreshes popular keys before they expire
4. **Jitter on TTL**: instead of TTL=3600, use TTL=3600±random(300) — stagger expirations

### Redis vs Memcached

| | Redis | Memcached |
|---|---|---|
| Data structures | Strings, Lists, Sets, Sorted Sets, Hashes, Streams, HLL, Geo | Strings only |
| Persistence | RDB snapshots + AOF log | None |
| Replication | Primary-replica + Sentinel | None built-in |
| Cluster | Redis Cluster (hash slots) | Client-side sharding |
| Pub/Sub | Yes | No |
| Lua scripting | Yes | No |
| Multi-threading | Single-threaded event loop (6.0+ I/O threads) | Multi-threaded |
| Use when | Most cases — richer feature set | Pure caching, maximum throughput, multi-threaded perf |

**Redis data structures for system design**:
```
String:      session tokens, counters (INCR), rate limit counters
Hash:        user profile (field-level reads)
List:        message queues (LPUSH/BRPOP), recent activity feeds
Set:         unique visitors, friend relationships
Sorted Set:  leaderboards (score → rank), delayed queues (score=timestamp)
HyperLogLog: approximate unique count (10TB → 12KB, ~0.81% error)
Geo:         store lat/lng, find nearby in radius
Streams:     append-only log, consumer groups (Kafka-lite)
```

---

## 9. Databases Deep Dive

### SQL vs NoSQL — When to Use What

**SQL (Relational)**:
- Structured data with relationships
- ACID transactions
- Complex queries with JOINs
- Schema enforced at write time
- Examples: PostgreSQL, MySQL, CockroachDB

**NoSQL (Non-relational)**:
- Flexible schema (schema enforced at read time)
- Horizontal scaling built-in
- Designed for specific access patterns
- Sacrifice some of ACID for scale/flexibility

**The real question isn't SQL vs NoSQL — it's: what are your access patterns?**

### ACID Properties

| Property | Meaning | Example |
|---|---|---|
| **Atomicity** | All or nothing — transaction either fully completes or fully rolls back | Transfer ₹1000: debit + credit happen together or neither |
| **Consistency** | DB moves from one valid state to another (constraints always hold) | Balance never goes negative (constraint) |
| **Isolation** | Concurrent transactions appear sequential | Two users can't both "claim" the last seat |
| **Durability** | Committed data survives crashes | After "payment confirmed", data is on disk |

### Transaction Isolation Levels

Problems that isolation levels prevent:

| Problem | Description |
|---|---|
| **Dirty read** | Read uncommitted data that may be rolled back |
| **Non-repeatable read** | Read same row twice in transaction, get different values |
| **Phantom read** | Read a range twice, second read has more/fewer rows |

| Isolation Level | Dirty Read | Non-repeatable | Phantom | Use case |
|---|---|---|---|---|
| Read Uncommitted | ✅ prevented? No | No | No | Almost never use |
| Read Committed | Yes | No | No | Default in most DBs (PostgreSQL) |
| Repeatable Read | Yes | Yes | No | Reporting, consistent snapshots |
| Serializable | Yes | Yes | Yes | Financial, inventory |

Higher isolation = fewer anomalies but lower concurrency (more locking, more contention).

### Database Indexing

An index is a separate data structure that maps column values to row locations, enabling fast lookups without full table scan.

**B-Tree index** (default for most DBs):
```
Balanced tree, O(log N) for point queries and range queries.
Used for: primary keys, WHERE clause columns, ORDER BY, JOIN keys.
Best for: columns with high cardinality and frequent range queries.
```

**Hash index**:
```
O(1) point lookups. No range queries.
Used for: exact match lookups.
```

**Composite index**:
```sql
CREATE INDEX idx_corporate_status_date ON transactions(corporate_id, status, created_at);
-- Useful for: WHERE corporate_id=X AND status=Y ORDER BY created_at
-- NOT useful for: WHERE status=Y (leftmost prefix rule)
```

**Covering index**: Index includes all columns the query needs — no need to fetch the actual row.

**Index trade-offs**:
- Reads faster, writes slower (index must be updated on every write)
- Disk space (index is a separate data structure)
- Too many indexes = slow writes, DB query planner confused

**When to NOT index**:
- Low-cardinality columns (e.g., `status` with 3 values — not selective enough)
- Small tables (full scan is faster)
- Columns never in WHERE/JOIN/ORDER BY

### Database Types

**Key-Value**: Redis, DynamoDB, Riak
- Access pattern: `get(key)`, `set(key, value)` only
- No queries on value fields
- Extreme speed, horizontal scale
- Use for: sessions, cache, leaderboards, shopping carts

**Document**: MongoDB, CouchDB, Firestore
- Documents (JSON/BSON), flexible schema
- Query on any field within the document
- No JOINs — embed related data instead
- Use for: user profiles, product catalogs, content management

**Wide-Column**: Cassandra, HBase, Google Bigtable
- Rows identified by partition key; columns are dynamic per row
- Optimized for writes at massive scale, time-series queries
- Schema design driven by query patterns
- Use for: IoT sensor data, time-series, audit logs, activity feeds

**Graph**: Neo4j, Amazon Neptune
- Nodes and edges, efficient for relationship traversal
- No joins needed — edges are first-class
- Use for: social networks (friends of friends), recommendation engines, fraud detection, knowledge graphs

**Time-Series**: InfluxDB, TimescaleDB, Prometheus
- Optimized for time-ordered data, rollup aggregations
- Automatic compression, downsampling
- Use for: metrics, monitoring, financial tick data, IoT telemetry

**Search Engine**: Elasticsearch, OpenSearch, Solr
- Inverted index for full-text search
- Relevance scoring, facets, autocomplete
- Not a primary store — replicate data from primary DB
- Use for: product search, log analytics, autocomplete

---

## 10. Database Scaling

### Read Replicas (Replication)

Copy data from primary to one or more replicas. Reads go to replicas, writes go to primary.

```
                         ┌─────────────┐
Write ──────────────────▶│   Primary   │───replication──▶ Replica 1
                         └─────────────┘                  Replica 2
Read (stale OK) ────────────────────────────────────────▶ Replica 3
```

**Types of replication**:
- **Synchronous**: primary waits for replica to acknowledge before confirming write. Durable, but higher latency.
- **Asynchronous**: primary confirms immediately, replica catches up later. Lower latency, but replica can lag.
- **Semi-synchronous**: wait for at least one replica (MySQL default).

**Replication lag problem**: If primary crashes before replica catches up, you lose those writes (in async mode). And reads from replica may be stale.

**When replication is enough**: read:write ratio > 5:1. Adds read capacity and geographic distribution.

### Sharding (Horizontal Partitioning)

Split the dataset across multiple independent databases (shards). Each shard holds a subset of data.

```
Shard 0: users where user_id % 4 = 0
Shard 1: users where user_id % 4 = 1
Shard 2: users where user_id % 4 = 2
Shard 3: users where user_id % 4 = 3
```

**Sharding strategies**:

**Hash-based**: `shard = hash(key) % N`
- Pro: Even distribution
- Con: Adding/removing shards requires rehashing all keys (use consistent hashing to mitigate)

**Range-based**: `shard = key range (A-M → shard1, N-Z → shard2)`
- Pro: Range queries are efficient (all data in one shard)
- Con: Uneven distribution (hotspots) — "A" names may be more common

**Directory-based**: A lookup service maps keys to shards
- Pro: Flexible, can move data without changing the application
- Con: Lookup service is a bottleneck and SPOF

**Sharding problems**:
1. **Cross-shard queries/JOINs**: You can't do `SELECT * FROM users JOIN orders` across shards. Either denormalize, keep related data on same shard, or accept client-side joins.
2. **Rebalancing**: When you add a shard, you must move data. Consistent hashing reduces movement.
3. **Hot shards**: If one shard gets most traffic (celebrity user, trending product), it becomes a bottleneck. Solution: further split, or shard by composite key.
4. **Auto-increment IDs**: Can't use DB auto-increment across shards (collisions). Use distributed ID generation (Snowflake IDs, UUIDs).

### Consistent Hashing

Problem: With `hash(key) % N`, adding one node reshuffles all keys (N-1)/N of them.

Solution: Place nodes on a ring of hash values 0–2^32. Each key is assigned to the next node clockwise.

```
Ring:  0 ─────── A ─────── B ─────── C ────── 0
         k1→A     k2→B       k3→C     k4→A
```

When you add node D between A and B: only keys that were going to B and are now closer to D move. Everything else is unchanged.

**Virtual nodes**: Each physical node has multiple positions on the ring (100–200). This prevents uneven load if nodes are at non-uniform positions.

Used by: Cassandra, DynamoDB, Redis Cluster, Akamai CDN.

### Vertical Partitioning

Split a table with many columns into multiple tables, each with a subset of columns.

```sql
-- Before: one wide table
CREATE TABLE users (
  id, name, email,           -- frequently read
  profile_photo_url,          -- frequently read
  preferences_json,           -- rarely read
  full_tax_documents_json     -- almost never read
);

-- After: split by access frequency
CREATE TABLE users_core (id, name, email, profile_photo_url);
CREATE TABLE users_preferences (id, preferences_json);
CREATE TABLE users_tax_documents (id, full_tax_documents_json);
```

Benefits: Smaller rows → more rows per page → better cache utilization. Rarely-read columns don't pollute the buffer pool.

### Connection Pooling

Each DB connection is expensive (~1–2 MB RAM, TCP handshake overhead). Without pooling, 10,000 concurrent requests → 10,000 DB connections → DB dies.

Connection pool: maintain N persistent connections, queue requests to reuse them.

```
Application ──── pool (20 connections) ──── Database
10,000 QPS flows through 20 connections (with fast queries)
Pool size = cores × 2 + disk spindles (PgBouncer rule of thumb)
```

Tools: HikariCP (Java), PgBouncer (PostgreSQL), ProxySQL (MySQL).

---

## 11. Message Queues and Event Streaming

### Why Queues?

1. **Decoupling**: Producer and consumer don't need to be available simultaneously
2. **Load leveling**: Absorb traffic spikes; consumers process at their own pace
3. **Async processing**: Don't block the user waiting for slow operations
4. **Fan-out**: One event → multiple consumers
5. **Durability**: Events persist even if consumers crash

**Without queue (synchronous)**:
```
User → API → Send Email → PDF → Push notification → Return 200
Total latency: 500ms (user waits for all of it)
If email service is down → 500 to user
```

**With queue (asynchronous)**:
```
User → API → Publish "order_placed" → Return 200 (fast, <20ms)
                        ↓ (async, decoupled)
           Email consumer → Notification consumer → Analytics consumer
```

### Message Delivery Guarantees

| Guarantee | Meaning | How |
|---|---|---|
| **At-most-once** | Message may be lost, never duplicated | Fire and forget. No ack. |
| **At-least-once** | No loss, but may deliver multiple times | Ack on successful process; retry on failure |
| **Exactly-once** | Delivered exactly once | Idempotent consumer + transactional producer |

In practice: **at-least-once + idempotent consumers** is the standard.

Idempotent consumer: processing the same message twice has the same effect as processing it once. Implement by tracking processed message IDs.

### Kafka vs RabbitMQ vs SQS

| | Kafka | RabbitMQ | AWS SQS |
|---|---|---|---|
| Model | Log/stream | Message broker | Cloud queue |
| Retention | Configurable (days/forever) | Until consumed | 4 days (max 14) |
| Consumer model | Pull, offset-based | Push to consumers | Pull |
| Replay | Yes (consumers control offset) | No | No |
| Ordering | Per partition | Per queue | Per MessageGroupId (FIFO) |
| Throughput | Millions/sec | ~50K/sec | ~3K/sec per queue |
| Use case | Event streaming, audit log, high throughput | Task queues, RPC, routing | Simple async tasks, AWS native |

### Kafka Deep Dive

**Core concepts**:
```
Topic:      Named stream of events (e.g., "transaction.created")
Partition:  Ordered, immutable log. Topics have N partitions.
Offset:     Position of a message within a partition.
Producer:   Writes messages to a topic/partition.
Consumer:   Reads messages from partitions, tracks its offset.
Consumer group: N consumers sharing partitions for parallel consumption.
Broker:     Kafka server node.
```

**Partitioning**:
```
Partition key = corporate_id
→ All events for a corporate go to same partition
→ Order is guaranteed within a corporate's events
→ Partition key = null → round-robin across partitions
```

**Consumer groups**:
```
Topic: transactions (4 partitions)
Consumer group A (2 consumers):
  Consumer A1 → Partition 0, 1
  Consumer A2 → Partition 2, 3

Consumer group B (1 consumer):
  Consumer B1 → All 4 partitions

Different groups are independent — each gets all messages.
```

Max parallelism per group = number of partitions. Extra consumers in the same group are idle.

**At-least-once in Kafka**:
```
Consumer reads message, processes it, commits offset.
If consumer crashes between process and commit → reads again → duplicate.
Solution: idempotent processing (check if event already processed).
```

**Exactly-once in Kafka** (Kafka transactions):
```
Producer writes atomically to multiple partitions.
Consumer reads, processes, writes output to Kafka + commits offset atomically.
Complex, higher latency. Only for critical paths.
```

**Dead Letter Queue (DLQ)**:
When a message fails processing after N retries, move it to a DLQ. Allows inspection, replay, or manual handling without blocking the main queue.

```java
// SME service pattern:
DefaultErrorHandler handler = new DefaultErrorHandler(
    new DeadLetterPublishingRecoverer(kafkaTemplate),
    new FixedBackOff(1000L, 1L)  // 1 retry after 1s
);
```

**Kafka consumer lag**: difference between latest offset and consumer's committed offset. High lag = consumer is behind.

### Message Queue Patterns

**Work queue**: Tasks distributed among N workers (competing consumers). Each task processed once.
```
Producer → Queue → Worker1, Worker2, Worker3 (one gets each message)
Use for: image resizing, email sending, payment processing
```

**Pub/Sub**: Messages broadcast to all subscribers.
```
Publisher → Topic → Subscriber1 (all get every message)
                 → Subscriber2
                 → Subscriber3
Use for: event notifications, cache invalidation, real-time dashboards
```

**Event sourcing**: All state changes are stored as an ordered log of events. Current state is derived by replaying events.
```
"account_created", "deposit:1000", "withdrawal:200", "deposit:500"
→ replay → balance = 1300

Benefits: full audit log, time travel (what was state at T?), replay
Cons: complex queries, eventual consistency, event schema evolution
```

---

## 12. API Design

### REST Principles

REST is an architectural style, not a protocol.

**Core constraints**:
1. **Stateless**: each request is self-contained. Server stores no client state.
2. **Uniform interface**: consistent resource identification (URLs), standard methods (GET/POST/PUT/DELETE/PATCH)
3. **Client-server**: UI and data storage are separated
4. **Layered system**: client doesn't know if it talks to LB, cache, or origin
5. **Cacheable**: responses should indicate if they're cacheable

**HTTP methods and idempotency**:

| Method | Idempotent | Safe | Use |
|---|---|---|---|
| GET | Yes | Yes | Read resource |
| HEAD | Yes | Yes | Check existence, metadata |
| PUT | Yes | No | Replace entire resource |
| DELETE | Yes | No | Delete resource |
| POST | No | No | Create resource, trigger action |
| PATCH | No | No | Partial update |

**Idempotent** = same request multiple times = same result. Important for retries.

**REST URL design**:
```
GET    /corporates/{id}                    → get corporate
POST   /corporates                         → create corporate
PUT    /corporates/{id}                    → replace corporate
PATCH  /corporates/{id}                    → update fields
DELETE /corporates/{id}                    → delete

GET    /corporates/{id}/employees          → employees of corporate
POST   /corporates/{id}/employees          → add employee
DELETE /corporates/{id}/employees/{empId}  → remove employee

GET    /transactions?status=PENDING&from=2024-01-01&page=1&size=20
```

**HTTP Status Codes**:
```
2xx Success:   200 OK, 201 Created, 202 Accepted, 204 No Content
3xx Redirect:  301 Moved Permanently, 302 Found, 304 Not Modified
4xx Client:    400 Bad Request, 401 Unauthorized, 403 Forbidden,
               404 Not Found, 409 Conflict, 429 Too Many Requests
5xx Server:    500 Internal Server Error, 502 Bad Gateway,
               503 Service Unavailable, 504 Gateway Timeout
```

**Pagination**:

Offset-based:
```
GET /transactions?page=5&size=20
SELECT * FROM transactions LIMIT 20 OFFSET 100;
Problem: if new data inserted, page 5 shifts → duplicate or missing items
```

Cursor-based (keyset pagination) — preferred for large datasets:
```
GET /transactions?after=cursor_xyz&size=20
SELECT * FROM transactions WHERE id > :cursor LIMIT 20;
Stable, no duplicates, works with real-time data
Downside: can't jump to page 50 directly
```

### API Versioning

Never break clients. Version your APIs.

| Strategy | Example | Notes |
|---|---|---|
| URL path | `/v1/users`, `/v2/users` | Most visible, easy to route |
| Query param | `/users?version=2` | Easy to test, not RESTful |
| Header | `Accept: application/vnd.niyo.v2+json` | Clean URLs, harder to test |
| Subdomain | `v2.api.niyo.com` | DNS routing, separate deployments |

**SME service pattern**: URL path versioning under context path `/sme/api/v1/`.

### GraphQL

Client specifies exactly what data it needs. Server returns exactly that — no over-fetching, no under-fetching.

```graphql
query {
  corporate(id: "123") {
    name
    employees(status: ACTIVE) {
      name
      salary
    }
  }
}
```

**When GraphQL over REST**:
- Mobile apps where bandwidth matters
- Client needs to compose data from many resources
- Multiple client types (web/mobile/partner) needing different data shapes
- Rapid UI iteration (change query without server change)

**GraphQL problems**:
- N+1 query problem (use DataLoader / batch loading)
- Complex authorization logic (field-level auth harder than endpoint-level)
- Caching harder (all POST to same endpoint)
- Higher backend complexity

### gRPC

RPC framework using Protocol Buffers (binary serialization). Strongly typed.

```protobuf
service CorporateService {
  rpc GetCorporate (GetCorporateRequest) returns (Corporate);
  rpc StreamTransactions (StreamRequest) returns (stream Transaction);
}
```

**gRPC vs REST**:
| | gRPC | REST/JSON |
|---|---|---|
| Protocol | HTTP/2 (binary) | HTTP/1.1 or 2 (text JSON) |
| Serialization | Protocol Buffers (~10x smaller) | JSON |
| Latency | Lower | Higher |
| Browser support | Limited (needs proxy) | Native |
| Code generation | Strong (stubs from proto) | Partial (OpenAPI) |
| Streaming | Bidirectional native | SSE/WebSocket workaround |
| Use case | Internal service-to-service | Public APIs |

### Idempotency Keys

For non-idempotent operations (payment, order creation), clients include an idempotency key:

```
POST /payments
Idempotency-Key: a1b2c3-uuid-per-client

Server:
1. Check if key exists in idempotency store (Redis)
2. If yes → return previous response (don't process again)
3. If no → process, store (key, response) with TTL, return response
```

This allows clients to safely retry on network timeouts without double-charging.

---

## 13. Microservices Patterns

### Monolith vs Microservices

**Monolith first.** Microservices are an optimization for scale and team independence, not a starting point.

| Dimension | Monolith | Microservices |
|---|---|---|
| Deployment | One deployable unit | Many independent deployables |
| Scaling | Scale everything together | Scale per service |
| Team independence | Hard with many teams | Each team owns their service |
| Latency | In-process calls (fast) | Network calls (slower) |
| Transactions | DB transactions easy | Distributed transactions are hard |
| Testing | Easier | Integration testing is harder |
| Operational complexity | Low | High (observability, deployment, discovery) |

**When to split**:
- Teams stepping on each other's code
- One module needs different scaling than another
- One part of the system has different reliability requirements
- Clear bounded contexts with minimal coupling

### Service Discovery

Services need to find each other. IPs change (auto-scaling, container restarts).

**Client-side discovery** (Ribbon/Eureka pattern):
```
Service A → asks Registry → gets list of Service B instances → picks one → calls it
```

**Server-side discovery** (Load Balancer pattern):
```
Service A → calls LB address → LB queries registry → LB routes to instance
```

In Kubernetes: DNS-based discovery built-in. Service name = DNS name. kube-proxy handles routing. No separate service registry needed.

### API Gateway

Single entry point for all client requests. Handles cross-cutting concerns:

```
Clients → API Gateway → Service A
                     → Service B
                     → Service C

Gateway responsibilities:
- SSL termination
- Authentication / JWT validation
- Rate limiting (per user, per IP, per endpoint)
- Request routing
- Request transformation (aggregate multiple service calls)
- Response transformation
- Logging, tracing, metrics injection
```

**BFF (Backend for Frontend)**: separate API gateways per client type (mobile BFF, web BFF, partner BFF). Each BFF can aggregate and shape data differently for its client without affecting others.

### Circuit Breaker

Prevent cascading failures when a downstream service is slow or failing.

**States**:
```
CLOSED (normal):   Requests pass through. Count failures.
OPEN (tripped):    After N failures, stop sending requests. Return error/fallback immediately.
HALF-OPEN (probe): After timeout, allow 1 test request. If success → CLOSED. If fail → OPEN.
```

**Without circuit breaker**:
```
Service A → Service B (slow, 30s timeout)
A has 100 threads. Each request waits 30s.
→ All 100 threads occupied waiting for B
→ Service A becomes unavailable even though B is the problem
```

**With circuit breaker (Resilience4j)**:
```java
@CircuitBreaker(name = "karza", fallbackMethod = "fallbackIFSC")
public IFSCDetails getIFSCDetails(String ifsc) { ... }

public IFSCDetails fallbackIFSC(String ifsc, Exception e) {
    return IFSCDetails.fromCache(ifsc); // stale data or default
}
```

### Saga Pattern — Distributed Transactions

Microservices span multiple databases. You can't do ACID across services. Saga is the solution.

**Saga**: a sequence of local transactions. Each step publishes an event/command. On failure, compensating transactions undo previous steps.

**Orchestration saga** (central coordinator):
```
Order Service (orchestrator):
1. Reserve inventory → success
2. Process payment  → success
3. Ship order       → FAIL
4. → Refund payment (compensate step 2)
5. → Release inventory (compensate step 1)
```

**Choreography saga** (event-driven, no coordinator):
```
Order Created → Inventory Service reserves stock → publishes "stock_reserved"
             → Payment Service charges card → publishes "payment_done"
             → Shipping Service creates shipment

On failure:  Payment Failed → Inventory Service listens → releases stock
```

Choreography is simpler to implement but harder to trace. Orchestration is easier to trace but the orchestrator is a bottleneck and coupling point.

### CQRS (Command Query Responsibility Segregation)

Separate the write model (commands) from the read model (queries).

```
Command side: REST API → validates, transforms → writes to MongoDB → publishes event
                                                                          ↓
Query side:   Read API ← materialized view (Elasticsearch, Redis) ← event consumer

Benefits:
- Read models can be optimized per query (no JOINs, pre-aggregated)
- Write and read scale independently
- Multiple read models from same write stream

Cost:
- Eventual consistency between write and read models
- More complexity, more moving parts
```

CQRS is powerful but adds significant complexity. Don't use it unless you have genuinely different read and write scaling needs.

### Bulkhead Pattern

Isolate resources for different services. If one overloads, it can't starve others.

```
HTTP client for external calls:
Pool A (20 threads) → Payment processor (slow 3rd party)
Pool B (50 threads) → YBL bank API
Pool C (10 threads) → KYC service

If payment processor is slow → only Pool A starves → B and C unaffected
```

---

## 14. Distributed Systems Primitives

### Distributed Locks

Two instances must not execute the same critical section simultaneously.

**Redis SETNX (SET if Not eXists)**:
```
SET lock:payment:txn123 "instance-id-456" NX PX 30000
# NX = only set if doesn't exist
# PX 30000 = expire in 30 seconds (TTL prevents deadlock)

On success: you hold the lock
On failure: another instance holds it
On complete: DEL lock:payment:txn123 (only if value matches your instance ID!)
```

**Redlock** (multi-node distributed lock):
```
Acquire lock on N/2+1 of N Redis nodes within a time window.
Safer against single Redis failure.
Controversial (Martin Kleppmann critique: still not safe with clock skew).
```

**Database-level lock** (simpler, use when you have a DB transaction):
```sql
SELECT * FROM transactions WHERE id=:id FOR UPDATE;
-- holds a row lock until transaction commits
```

### Leader Election

Only one instance should perform a task (scheduled jobs, partition ownership).

**Zookeeper/etcd ephemeral node**:
```
All instances try to create /election/leader (ephemeral node)
First one succeeds → becomes leader
Others watch for deletion → if leader dies, node disappears → race for new leader
```

**Kubernetes**: Use a Lease object. Leader continuously renews the lease. Others try to acquire when lease expires.

### Consensus — Raft and Paxos

How do distributed systems agree on a value (leader, log entry) when nodes can fail?

**Paxos**: Original algorithm. Correct but notoriously hard to understand and implement.

**Raft**: Designed to be understandable. Used by etcd, CockroachDB, TiKV.

Raft key ideas:
1. One leader at a time (elected by majority vote)
2. Leader receives all writes, replicates to followers
3. A write is committed when majority of nodes acknowledge it
4. If leader fails, followers elect a new one (whoever has the most up-to-date log)

**What this means for system design**: For strong consistency, you pay the cost of a quorum (majority of N nodes must agree). This is why etcd/Zookeeper are slow compared to Redis — they're giving you real distributed consensus.

### Gossip Protocol

Nodes in a cluster share state updates by randomly "gossiping" with a few neighbors. Information spreads exponentially (like a rumor).

Used by: Cassandra (cluster membership, token distribution), DynamoDB, Bitcoin.

```
Node A tells Node C, D about new node joining
Node C tells Node E, F
Node D tells Node G, B
→ All nodes know within O(log N) rounds
```

Properties: Eventual propagation (not instant), resilient to node failures, low overhead.

### Two-Phase Commit (2PC)

Protocol for atomic writes across multiple databases.

```
Phase 1 (Prepare):
  Coordinator → "Can you commit?"
  Participant 1 → "Yes" (locks resources)
  Participant 2 → "Yes" (locks resources)

Phase 2 (Commit):
  Coordinator → "Commit!"
  Participant 1 → commits
  Participant 2 → commits
```

**Problem**: Coordinator crashes between phases 2 → participants hold locks forever (blocking). 2PC is blocking and has a blocking failure mode.

**Better**: Saga (compensating transactions) for microservices. 2PC for databases that support it natively (distributed RDBMS).

### Vector Clocks

Logical timestamps to track causality in distributed systems.

```
A: [A:1, B:0, C:0]  "A did something"
A → B: B receives event → B: [A:1, B:1, C:0]
B → C: C receives event → C: [A:1, B:1, C:1]
A → C directly: C: [A:1, B:1, C:2]  "C saw A and B"

If A: [A:2, B:0] and B: [A:1, B:1] → concurrent (neither is ancestor)
```

Used by DynamoDB, Riak for conflict detection.

---

## 15. Storage Systems

### Object Storage (S3)

Flat namespace of key-value blobs. Not a filesystem.

```
Bucket: niyo-sme-documents
Key:    kyc/corporate/123/pan_card.pdf
Value:  binary blob

Get: GET /kyc/corporate/123/pan_card.pdf
```

**Strengths**: Infinite scale, 11 nines durability (3 replicas across AZs), cheap ($0.023/GB/month), accessible from internet.

**Weaknesses**: High latency (10–200ms), no atomic operations across objects, no partial updates (must replace whole object).

**Presigned URLs**: grant temporary access to private objects without making them public. Your server generates a signed URL, hands it to the client, client directly downloads from S3.

```java
// Generate presigned URL valid for 15 minutes
String url = s3Presigner.presignGetObject(r ->
    r.getObjectRequest(g -> g.bucket(bucket).key(key))
     .signatureDuration(Duration.ofMinutes(15)));
```

### Block Storage (EBS)

Like a hard drive attached to one server. Files, filesystem, databases can run on it.

```
EC2 instance → mounts EBS volume → sees it as /dev/xvda
Can store: PostgreSQL data files, MongoDB data directory, logs
```

**Strengths**: Low latency (<1ms), supports filesystems, atomic writes.
**Weaknesses**: Attached to one instance at a time, bounded size, not globally accessible.

### File Storage (EFS, NFS)

Shared filesystem mountable by multiple servers simultaneously.

```
EC2 1 → mounts /mnt/shared
EC2 2 → mounts /mnt/shared (same filesystem, same files)
```

Use for: shared config files, media processing (multiple workers on same file), legacy apps expecting a filesystem.

### Data Storage Decision Tree

```
Need to store large binary files (images, PDFs, videos)?
→ Object storage (S3)

Need a database?
→ Need ACID, complex queries, JOINs?
  → Relational: PostgreSQL, MySQL
→ Need flexible schema, document model?
  → MongoDB, Firestore
→ Need extreme write throughput, time-series?
  → Cassandra, InfluxDB
→ Need low-latency key-value cache?
  → Redis
→ Need full-text search?
  → Elasticsearch

Need a shared filesystem?
→ EFS/NFS

Need fast I/O for a database on one server?
→ Block storage (EBS/NVMe)
```

---

## 16. Search Systems

### How Search Engines Work

**Inverted index**: Instead of storing documents and searching through them, store a map from term → list of documents containing that term.

```
Documents:
  D1: "Niyo SME banking platform"
  D2: "SME corporate payroll"
  D3: "Niyo consumer banking"

Inverted index:
  "niyo"     → [D1, D3]
  "sme"      → [D1, D2]
  "banking"  → [D1, D3]
  "platform" → [D1]
  "corporate"→ [D2]
  "payroll"  → [D2]
  "consumer" → [D3]

Query "niyo sme" → intersect([D1,D3], [D1,D2]) → D1
```

**Tokenization**: Split text into tokens. "Niyo SME" → ["niyo", "sme"]. Includes lowercasing, punctuation removal.

**Stemming**: Reduce to root form. "running", "runs", "ran" → "run".

**Relevance scoring (TF-IDF)**:
- TF: how often does the term appear in the document?
- IDF: how rare is the term across all documents?
- Score = TF × IDF (rare terms in a specific document rank higher)

**BM25**: Modern scoring algorithm used by Elasticsearch. Better than TF-IDF for practical use.

### Elasticsearch

Distributed document search engine. Used for full-text search, log analytics.

```
Index = collection of documents (like a table)
Shard = Lucene index = unit of parallelism
Replica = copy of shard for availability

Write path: document → indexing → tokenization → inverted index update → searchable
(near-real-time: ~1 second delay before searchable)

Read path: query → scatter to all shards → each shard returns top K → merge → return top K globally
```

**Elasticsearch in system design**:
- NOT a primary store — replicate from MongoDB/PostgreSQL via Change Data Capture (CDC) or event-driven sync
- Use for: product search, employee search, transaction search with filters
- Trade-offs: eventual consistency (sync delay), complex cluster management, no ACID

**Autocomplete**: Use edge n-grams or completion suggester. Index "niyo" → "n", "ni", "nio", "niyo". User types "nio" → immediate results.

### Search in System Design Interviews

URL shortener doesn't need Elasticsearch. But a job search (millions of listings, complex filters, relevance ranking) does.

**When to recommend Elasticsearch**:
- Full-text search with relevance
- Faceted search (filter by category, price range, location simultaneously)
- Log analytics (ELK stack)
- Autocomplete / type-ahead

**Alternatives**: Meilisearch (simpler, fast, self-hosted), Typesense, Algolia (managed, expensive), pgvector + PostgreSQL full-text search (for smaller scale).

---

## 17. Rate Limiting

Rate limiting protects your service from abuse, controls costs, and enforces fair use.

### Algorithms

**Token Bucket**:
```
Bucket holds up to N tokens. Refilled at rate R tokens/sec.
Each request consumes 1 token.
No tokens → reject (429) or wait.

Allows burst up to N, then sustained rate of R.
Example: N=100 (burst), R=10/sec (sustained)
→ Can handle 100 requests instantly, then 10/sec ongoing
```

**Leaky Bucket**:
```
Requests enter queue. Processed at constant rate R.
If queue full → reject.
Smooths out bursts — output is always steady rate R.
Good for: protecting downstream services that can't handle bursts.
```

**Fixed Window Counter**:
```
Window: 1 minute
Counter per user per window.
Reset at window boundary.

Problem: user makes 100 requests at 11:59:59 and 100 requests at 12:00:01
→ 200 requests in 2 seconds (but allowed by fixed window rules).
```

**Sliding Window Log**:
```
Store timestamp of each request in Redis sorted set.
On request: remove entries older than window. Count remaining.
If count < limit → allow, add timestamp.
If count >= limit → reject.

Accurate but uses O(requests) memory per user.
```

**Sliding Window Counter** (hybrid):
```
current_rate = prev_window_count × (time_remaining_in_prev_window / window_size)
             + current_window_count

Accurate and memory-efficient.
```

### Where to Implement Rate Limiting

```
API Gateway:          Per IP, per API key — before reaching services
Application layer:    Per user, per endpoint — business logic aware
Distributed cache:    Redis stores counters — shared across all instances
```

### Rate Limiting in Redis

```
MULTI
INCR user:123:2024-01-01-14:30 (minute-based key)
EXPIRE user:123:2024-01-01-14:30 60 (TTL to auto-cleanup)
EXEC

If counter > limit → return 429
Response headers:
  X-RateLimit-Limit: 100
  X-RateLimit-Remaining: 42
  X-RateLimit-Reset: 1704106800
  Retry-After: 37
```

---

## 18. Concurrency and Locking

### Race Conditions

Two requests read the same value, both increment it, both write back.

```
Account balance = 1000
Request A reads 1000, computes 1000 + 500 = 1500
Request B reads 1000, computes 1000 + 500 = 1500
Request A writes 1500
Request B writes 1500
Final balance: 1500 (should be 2000!)
```

### Optimistic Locking

Read, compute, write — but check if data changed before committing.

```sql
-- Read with version
SELECT balance, version FROM accounts WHERE id=1;
-- Got balance=1000, version=5

-- Update only if version unchanged
UPDATE accounts
SET balance=1500, version=6
WHERE id=1 AND version=5;

-- If 0 rows affected → conflict → retry
```

**Use when**: Conflicts are rare. Low contention.

In MongoDB:
```java
// Store version field on document
// Use findOneAndUpdate with filter including current version
```

### Pessimistic Locking

Lock before reading. No one else can modify until you release.

```sql
BEGIN;
SELECT * FROM accounts WHERE id=1 FOR UPDATE; -- lock
-- compute new balance
UPDATE accounts SET balance=1500 WHERE id=1;
COMMIT; -- release lock
```

**Use when**: Conflicts are frequent. Data must be consistent.

**Problem**: Deadlocks. A locks row 1, B locks row 2. A waits for row 2, B waits for row 1. Solution: consistent lock ordering, or let DB detect and break deadlock.

### Idempotency

Critical for payment systems and any operation where retries happen.

```
Client sends "pay ₹1000 for order ABC"
Network times out — did it process?
Client retries

Without idempotency: ₹2000 charged
With idempotency key "order-ABC-payment-v1":
  → Server checks: already processed this key? Yes → return same response, don't charge again
```

Implementation:
```java
// 1. Client provides Idempotency-Key header
// 2. Server checks Redis: GET "idempotency:order-ABC-payment-v1"
// 3. If exists → return cached response
// 4. If not → process, store result in Redis with TTL 24h, return result
```

---

## 19. Security Fundamentals

### Authentication vs Authorization

**Authentication (AuthN)**: Who are you? Verify identity.
**Authorization (AuthZ)**: What are you allowed to do? Verify permissions.

```
AuthN: JWT token validated → user is "Kunal Ladhani, corporate admin"
AuthZ: Does "corporate admin" have permission to approve payroll? Yes.
```

### JWT (JSON Web Token)

```
Header.Payload.Signature

Header:  {"alg": "HS256", "typ": "JWT"}  (base64url encoded)
Payload: {"sub": "user123", "role": "ADMIN", "exp": 1704106800}  (base64url encoded)
Signature: HMAC-SHA256(header + "." + payload, secret)

Stateless: server verifies signature without DB lookup → fast, scalable
Downside: can't revoke individual tokens (until expiry)
```

**JWT best practices**:
- Short expiry for access tokens (15 min), longer for refresh tokens (7 days)
- Store refresh tokens in DB (allows revocation)
- Don't store sensitive data in payload (it's base64, not encrypted — visible to anyone)
- Use asymmetric keys (RS256) for multi-service scenarios — services verify with public key, only auth server has private key

### OAuth 2.0

Authorization framework for delegated access (third-party apps on behalf of user).

**Authorization Code Flow** (most secure, for web/mobile apps):
```
1. User clicks "Login with Google"
2. Redirect to Google /authorize
3. User consents
4. Google redirects back with authorization code
5. Your server exchanges code for access_token + refresh_token (server-to-server, code never touches browser)
6. Use access_token to call Google API
```

**Client Credentials Flow** (service-to-service, no user involved):
```
Service A → POST /token {client_id, client_secret, grant_type=client_credentials}
         → Gets access_token
         → Calls Service B with Bearer token
```

### TLS/HTTPS

TLS encrypts traffic between client and server. Prevents eavesdropping and tampering.

**TLS Handshake (simplified)**:
```
1. Client Hello: "I support TLS 1.3, here are my cipher suites"
2. Server Hello: "Let's use AES-256-GCM. Here's my certificate."
3. Client verifies certificate against trusted CA (Certificate Authority)
4. Key exchange (ECDHE): derive shared symmetric key
5. Both sides use shared key for AES encryption
```

**Certificate pinning**: Mobile apps hardcode expected certificate hash. Prevents MITM even with rogue CAs. Pain to rotate certificates, but stronger security.

### Common Vulnerabilities

**SQL Injection**: Never concatenate user input into SQL.
```java
// Vulnerable
String sql = "SELECT * FROM users WHERE email='" + email + "'";
// email = "'; DROP TABLE users; --"

// Safe: use prepared statements
PreparedStatement ps = conn.prepareStatement("SELECT * FROM users WHERE email=?");
ps.setString(1, email);
```

**CSRF (Cross-Site Request Forgery)**: Attacker tricks user's browser into making requests to your API.
- Mitigation: CSRF tokens, SameSite cookie attribute, check Origin/Referer headers

**XSS (Cross-Site Scripting)**: Attacker injects malicious JS into your page.
- Mitigation: sanitize user input, Content-Security-Policy headers, use framework escaping (React does this automatically)

**Rate limiting**: Prevent brute force, credential stuffing, scraping.

---

## 20. Observability in System Design

> Full detail in `docs/observability-101.md`. Summary for system design interviews below.

### The Three Pillars

**Metrics**: numerical time-series aggregated over instances.
- QPS, error rate, latency p99, DB connection pool utilization, Kafka lag

**Logs**: discrete events with context.
- Structured JSON logs with traceId, userId, amount for correlation

**Traces**: request journey across services.
- Span tree showing exactly which service/DB call was slow

### What Interviewers Want to Hear

When you design any system, mention:

1. **Metrics**: "I'd track p99 latency, error rate, throughput per endpoint, Kafka consumer lag, and DB connection pool saturation. I'd alert when p99 > SLO threshold or error rate > 1%."

2. **Health checks**: "Each service exposes GET /health. Load balancer uses it for routing. Kubernetes uses liveness and readiness probes."

3. **Distributed tracing**: "Each request gets a traceId propagated through all service calls. This lets us reconstruct the full call chain when debugging a slow request."

4. **Alerting strategy**: "Alert on symptoms (user-visible things: error rate, latency, availability) not causes (CPU, memory — these are debugging tools, not alert signals)."

5. **Dashboards**: "Four Golden Signals (latency, throughput, errors, saturation) dashboard is the starting point. Per-service, per-endpoint breakdown on drill-down."

---

## 21. Classic Design Problems

This section gives you the reasoning and key design decisions for the most common interview problems. Don't memorize solutions — internalize the approach.

---

### Design a URL Shortener (TinyURL)

**Functional**: Given long URL → generate short URL. Short URL → redirect to long URL.
**Non-functional**: 100M URLs shortened/day. 10B redirects/day. Low latency redirects (<10ms). 10-year retention.

**Estimation**:
```
Writes: 100M/day = 1,200/sec
Reads: 10B/day = 116,000/sec (100:1 read:write)
Storage: 100M × 365 × 10 = 365B URLs
Each URL: 500 bytes avg → 182 TB over 10 years
```

**Short URL generation**:
Option A: hash (MD5/SHA1) the long URL, take first 7 chars. Problem: collisions.
Option B: base62 encode an auto-increment ID. Problem: predictable sequential IDs (users can enumerate).
Option C: base62 encode a random number. Problem: random collision check needed.
Option D: **KGS (Key Generation Service)** — pre-generate 7-char keys, store in DB (used/unused). On request, take one key atomically.

Best for interviews: **base62 encoding of a distributed ID (Snowflake)**.
```
62^7 = 3.5 trillion unique URLs — enough for centuries
7 chars: a-z, A-Z, 0-9
```

**Architecture**:
```
Client → API Gateway → URL Shortener Service → Redis Cache → PostgreSQL
                              ↑
                         KGS or ID generator

Redirect path:
Client → GET /abc1234 → LB → URL Service → Redis lookup → 301/302 Redirect

Store: short_code → {long_url, created_at, expires_at, creator_id}
```

**Cache**: cache short→long mapping in Redis (read-heavy). 80% of redirects are for 20% of URLs (Pareto). Cache the hot ones.

**301 vs 302 redirect**:
- 301 Permanent: browser caches forever → reduces server load but you can't change destination later
- 302 Temporary: browser always asks server → you can track click analytics, update destination

---

### Design a Pastebin

Similar to URL shortener but stores text content (up to 10 MB).

**Key difference**: content can be large. Store text in S3 (object storage), not in DB.

```
DB schema: paste_id, title, language, visibility, expiry, s3_key, creator_id
S3: store text content by s3_key

Write: client uploads text → service stores in S3 → saves metadata in DB → returns paste URL
Read: fetch metadata from DB → fetch content from S3 (or CDN for public pastes)
```

**Access control**:
- Public pastes → serve via CDN
- Private pastes → presigned S3 URL, validate auth before generating

---

### Design Twitter/X (Social Feed)

**Functional**: post tweet, follow users, see home timeline (tweets from followed users, newest first).

**The hard part**: home timeline delivery at scale. 300M users. 500M tweets/day. Some users have 100M followers.

**Option 1: Fan-out on write (push model)**
```
User posts tweet → immediately write tweet to timeline of all followers
Timeline = pre-computed Redis sorted set per user
Read is O(1): just read your timeline

Problem: Lady Gaga (100M followers) tweets → 100M timeline writes!
Write amplification = followers count
Latency for celebrities: 100M insertions per tweet = minutes
```

**Option 2: Fan-out on read (pull model)**
```
User posts tweet → write to tweet store
On timeline read → fetch all followed users → get their tweets → merge/sort

Problem: user follows 1000 people → 1000 DB queries per timeline read
High read latency, high DB load
```

**Hybrid (Twitter's actual approach)**:
```
Regular users (< 1M followers): Fan-out on write → push to followers' timelines
Celebrities (> 1M followers): Fan-out on read → pull their tweets separately and merge

On timeline read:
1. Read pre-built timeline (push model)
2. Fetch any celebrity tweets separately (pull model)
3. Merge and sort
```

**Data model**:
```
Tweets: {tweet_id (Snowflake), user_id, content, created_at, media_urls}
User timeline: Redis sorted set — key="timeline:user_id", score=tweet_timestamp, value=tweet_id
Followers graph: Graph DB or denormalized list
```

---

### Design WhatsApp

**Functional**: 1:1 messaging, group messaging, message delivery receipts (sent/delivered/read), media sharing, online presence.

**Scale**: 2B users. 100B messages/day. ~1M messages/second.

**Core design challenge**: real-time message delivery when recipient may be offline.

**Architecture**:
```
Client → WebSocket → Chat Server → Message Queue → DB (Cassandra)
                                               → Push notification (if offline)

Chat servers are stateful: hold WebSocket connections.
User connects to a specific chat server.
Message Router (or Zookeeper) maps user → chat server.
```

**Message delivery**:
```
Sender (online, connected to Server A):
1. Client sends message to Server A via WebSocket
2. Server A stores message in Cassandra
3. Server A looks up which server holds recipient's WebSocket (Server B)
4. Server A sends message to Server B via internal protocol
5. Server B delivers to recipient via WebSocket
6. ACK: delivered → sent back to Server A → Server A notifies sender

If recipient offline:
  → Push notification (APNs/FCM)
  → Message queued in Cassandra until recipient comes online
```

**Database (Cassandra)**:
```
Table: messages_by_conversation
Partition key: conversation_id
Clustering key: message_id (time-based) DESC
→ "get last 20 messages for conversation X" = single partition read
```

**Media**: Upload to S3, share URL. CDN for fast delivery.

**End-to-end encryption**: Client generates key pair. Public key stored on server. Messages encrypted with recipient's public key on sender device. Server never sees plaintext.

**Group messaging**: Fan-out server-side or client-side. For large groups (1000+), fan-out on read to avoid massive write amplification.

---

### Design YouTube

**Functional**: upload video, watch video, search video, recommendations.

**Scale**: 500 hours of video uploaded per minute. 1B+ hours watched per day.

**Video upload pipeline**:
```
Client → Upload Service → Raw Video S3
                              ↓
                    Transcoding Pipeline (distributed, parallel)
                         ↓ (multiple resolutions: 360p, 720p, 1080p, 4K)
                         ↓ (multiple codecs: H.264, VP9, AV1)
                    Transcoded chunks → CDN S3

Metadata:  Video DB (PostgreSQL/Cassandra) — title, description, tags, channel_id
Thumbnails: auto-generated from video frames, stored in S3
```

**Video streaming**:
```
Adaptive bitrate streaming (HLS or DASH):
- Video split into 10-second chunks
- Multiple quality levels (360p, 720p, 1080p)
- Player selects quality based on available bandwidth
- Seamless switching without buffering

Client → CDN → fetches chunks
CDN edge serves chunks from nearby PoP
```

**Search**: Index title, description, tags in Elasticsearch. Recommendations: ML-based collaborative filtering (not in scope for HLD interview, just mention).

---

### Design a Ride-Sharing App (Uber/Ola)

**Functional**: rider requests ride, nearby drivers notified, driver accepts, tracking, payment.

**The hard part**: geospatial matching — find available drivers within N km of rider, in real-time.

**Driver location tracking**:
```
Driver app sends GPS coordinates every 5 seconds
Location Service → Redis Geo (GEOADD)

Redis Geo stores lat/lng internally as sorted set with geohash score.
GEORADIUS driver:locations 12.9716 77.5946 5 km → returns nearby driver IDs
```

**Matching**:
```
Rider requests → Matching Service
Matching Service: GEORADIUS → get nearby drivers → filter available → rank (proximity, rating, type)
→ Send push notification to top 3 drivers → first to accept wins
```

**Trip tracking**:
```
Driver → Location Service → Kafka → Trip Store
                                 → Rider's WebSocket (real-time location update)
Store trip waypoints: coordinates every 10 seconds (for ETA, routing, billing)
```

**Surge pricing**: aggregate driver supply and rider demand per geohash bucket. If demand >> supply → multiply base price.

---

### Design Google Drive / Dropbox

**Functional**: upload files, sync across devices, share files, version history.

**The hard part**: efficient sync — don't re-upload unchanged parts of large files.

**Block-based sync**:
```
File divided into blocks (4 MB each)
Each block hashed (SHA-256)

On change:
1. Client splits file into blocks
2. Compares block hashes with server
3. Only upload changed blocks

Delta sync: 1 GB file with 1 line changed → upload 4 MB block, not 1 GB
```

**Storage**:
```
S3: file blocks (immutable, keyed by content hash → deduplication!)
DB (PostgreSQL): file metadata, block references, version history
  file_id, name, owner_id, created_at
  file_version: version_id, file_id, block_ids[], timestamp
  block: block_hash, s3_key, size
```

**Sync protocol**:
```
Client registers a long-poll or WebSocket connection
Server pushes change notifications: "file X updated at version 7"
Client fetches latest version → pulls only changed blocks
```

**Conflict resolution**: last-write-wins or create conflict copies ("file.txt (Kunal's conflicted copy)").

---

### Design a Notification System

**Functional**: send email/SMS/push to users. Triggered by events. Priority levels.

**Scale**: 10M notifications/day across email, SMS, push.

**Architecture**:
```
Events (Kafka):
  "payment_received" → Notification Service subscribes
  "kyc_approved"     → Notification Service subscribes

Notification Service:
1. Receives event
2. Looks up user preferences (opt-out? preferred channel?)
3. Renders template with user data
4. Routes to channel handler:
   - Email → SendGrid/SES
   - SMS → Twilio/Gupshup
   - Push → APNs (iOS) / FCM (Android)
5. Stores delivery status

Priority queue: critical (OTP, fraud alert) in separate high-priority Kafka partition
```

**Template engine**: Mustache/Handlebars for email templates. Store templates in DB, render at send time.

**Delivery tracking**:
```
notification: id, user_id, channel, template, status (PENDING/SENT/DELIVERED/FAILED), sent_at
Webhook from provider (SendGrid) → mark DELIVERED or FAILED
Retry failed notifications with exponential backoff
```

**Rate limiting**: don't spam users. Max N notifications per user per hour per channel.

---

### Design a Rate Limiter (as a Service)

**Functional**: given user/IP + endpoint, allow or deny request based on configured limits.

**Architecture (distributed)**:
```
API Gateway → Rate Limiter Service → Redis (sliding window counters)
             (check limit)         
If denied → 429 Too Many Requests
If allowed → forward to backend
```

**Redis sliding window counter**:
```lua
-- Lua script (atomic in Redis)
local key = KEYS[1]          -- "rl:user123:POST:/payment"
local window = tonumber(ARGV[1])   -- 60 (seconds)
local limit = tonumber(ARGV[2])    -- 10 (max requests)
local now = tonumber(ARGV[3])      -- current timestamp ms

-- Add current request timestamp
redis.call('ZADD', key, now, now)

-- Remove entries outside window
redis.call('ZREMRANGEBYSCORE', key, 0, now - window * 1000)

-- Count remaining
local count = redis.call('ZCARD', key)
redis.call('EXPIRE', key, window)

if count > limit then
    return 0  -- deny
else
    return 1  -- allow
end
```

**Multiple limit dimensions**:
- Per IP: prevent DDoS
- Per user: fair use
- Per API key: partner quotas
- Per endpoint: protect expensive operations

---

### Design a Distributed Cache

**Functional**: get(key), set(key, value, ttl), delete(key). Horizontally scalable.

**Architecture**:
```
Clients → Cache Cluster (N shards)

Shard assignment: consistent hashing on key
Each shard: one primary + N replicas

Client library: knows cluster topology, routes requests to correct shard
```

**Cache cluster rebalancing**:
```
Add node: takes ownership of ~1/N keys from other nodes
Remove node: its keys redistributed to remaining nodes
Consistent hashing: only 1/N of keys move (not all)
```

**Persistence options** (Redis):
- RDB: periodic snapshot → fast restart, may lose recent writes
- AOF: log every write → slower but minimal data loss
- Both: snapshot + AOF replay for recovery

---

### Design a Search Autocomplete

**Functional**: user types "niy" → suggestions ["niyo", "niyo banking", "niyo sme"]

**Data structure**: Trie (prefix tree)
```
                root
               / | \
              n  ...
             |
             i
             |
             y
             |
             o  ← "niyo" (frequency: 50000)
            / \
           b   s
           |   |
           a   m
           |   |
           n   e  ← "niyo sme" (frequency: 5000)
```

**Scale problem**: Trie in memory won't fit for millions of queries. Update frequency data in real-time.

**Practical approach (Elasticsearch/Redis)**:

Using Redis Sorted Set:
```
Key: "autocomplete:prefix:niy"
Members: "niyo":50000, "niyo banking":40000, "niyo sme":5000

ZADD autocomplete:prefix:niy 50000 "niyo"
ZREVRANGE autocomplete:prefix:niy 0 4  → top 5 results

Problem: N keys per query string (one per prefix length) → huge storage
```

Better: store top-K suggestions per prefix. Offline job recomputes from query logs.

**Architecture**:
```
Query logs → Kafka → Analytics Service → Aggregate top queries
                                      → Update Trie/Search index daily
                                      
User types "niy" → GET /autocomplete?q=niy → Redis/Elasticsearch → top 5 results
```

---

## 22. Common Mistakes in Interviews

### Design Mistakes

**1. Starting with the solution**: Drawing boxes before clarifying requirements. Ask first.

**2. Ignoring the non-functional requirements**: "Design Instagram" — latency? consistency? scale? Without these you're designing in a vacuum.

**3. Over-engineering too early**: Adding Kafka, Elasticsearch, Redis, and sharding before establishing that you even need them. Start simple, add complexity when you articulate the bottleneck.

**4. Single points of failure**: Every component you draw should prompt "what if this fails?" Load balancers, services, databases — all should have HA answers.

**5. Not considering the data model early**: How you store data determines what queries are fast. Design schema in parallel with architecture.

**6. Assuming infinite resources**: You have real constraints. A 100 QPS system doesn't need sharding. Calibrate complexity to scale.

### Communication Mistakes

**7. Silence**: Thinking without talking. Interviewers want to hear your reasoning, not just your conclusion.

**8. Being too definitive too early**: "I'll use PostgreSQL." Why? What access patterns drove that? Show the trade-off analysis.

**9. Not using the whiteboard**: Visual architecture is much clearer than verbal. Draw as you talk.

**10. Getting stuck in details**: Spending 20 minutes on the ID generation scheme when the interviewer wants to discuss the overall architecture.

**11. Not asking about priorities**: "Should I focus on the write path or the read path?" Let the interviewer redirect you.

### Trade-off Mistakes

**12. Treating all requirements as equal**: Availability vs. consistency — which one loses? For payments, consistency. For social feed, availability.

**13. Not discussing failure scenarios**: What happens when Redis is down? When the DB is slow? When Kafka lag spikes?

**14. Ignoring operational concerns**: How is the system deployed? How do you do zero-downtime deployments? How do you debug production issues?

### The Checklist (Final 2 Minutes of Interview)

Before time is up, mention:
- [ ] Where are the bottlenecks?
- [ ] What's the failure mode of each component?
- [ ] How do you handle data consistency?
- [ ] Where does caching help?
- [ ] How do you monitor this system?
- [ ] How does the system scale from MVP to 10x?

---

## 23. Glossary

| Term | Definition |
|---|---|
| **ACID** | Atomicity, Consistency, Isolation, Durability — properties of DB transactions |
| **Availability** | % of time system responds to requests |
| **BASE** | Basically Available, Soft state, Eventually consistent — NoSQL alternative to ACID |
| **BFF** | Backend for Frontend — API gateway per client type |
| **CAP Theorem** | Can only guarantee 2 of: Consistency, Availability, Partition tolerance |
| **CDC** | Change Data Capture — stream DB changes as events |
| **CDN** | Content Delivery Network — edge cache distributed globally |
| **Circuit Breaker** | Stops calls to failing dependency; fails fast and recovers |
| **Consistent Hashing** | Hash ring algorithm minimizing key movement when nodes change |
| **CQRS** | Separate read and write models |
| **DLQ** | Dead Letter Queue — holds failed messages for inspection |
| **Durability** | Written data survives system failures |
| **Eventual Consistency** | All replicas will converge given no new writes |
| **Fan-out** | Deliver one message to multiple recipients |
| **Gossip Protocol** | Peer-to-peer state propagation via random neighbor sharing |
| **HLD** | High-Level Design — architecture, components, data flow |
| **Idempotent** | Same operation repeated = same result |
| **Inverted Index** | Map from term → documents containing it |
| **LB** | Load Balancer |
| **LFU** | Least Frequently Used cache eviction |
| **Little's Law** | L = λW — queue depth = arrival rate × latency |
| **LRU** | Least Recently Used cache eviction |
| **Linearizability** | Operations appear instantaneous and globally ordered |
| **MTTR** | Mean Time to Recovery |
| **MTTF** | Mean Time to Failure |
| **Partition** | Network split between nodes |
| **p99** | 99th percentile latency — worst 1% of requests |
| **Raft** | Understandable consensus algorithm (etcd, CockroachDB) |
| **Replication** | Copying data to multiple nodes for durability and availability |
| **Saga** | Distributed transaction using compensating transactions |
| **Sharding** | Horizontal partitioning of data across multiple DB instances |
| **SLA** | Service Level Agreement — contractual uptime commitment |
| **SLI** | Service Level Indicator — the actual measured metric |
| **SLO** | Service Level Objective — internal target (usually stricter than SLA) |
| **SPOF** | Single Point of Failure |
| **SSE** | Server-Sent Events — one-way server→client stream over HTTP |
| **Throughput** | Operations per unit time (QPS, TPS) |
| **TTL** | Time to Live — expiry duration for cache entries or DNS records |
| **Vector Clock** | Logical timestamp tracking causality across distributed nodes |
| **WAL** | Write-Ahead Log — changes logged before applying; enables recovery |
| **WebSocket** | Bidirectional persistent connection over TCP |
| **2PC** | Two-Phase Commit — atomic write across multiple databases |

---

> **Next steps**: After reading this, pick 5 design problems and do them timed (45 min each). For each one: write requirements, draw the architecture, identify 2–3 bottlenecks, and articulate the trade-offs out loud. The goal isn't memorizing this doc — it's building the mental model to reason from first principles when you see a problem you haven't seen before.

---

## 24. Further Reading

The best system designers read real engineering war stories, not just textbooks. These are the places where engineers write about what actually broke at scale and how they fixed it.

### Must-Read Blogs

**[High Scalability](https://highscalability.com/)**
The canonical resource. Deep dives into how real companies (Twitter, Netflix, Amazon, Dropbox) built their systems. The "How X scales" teardown format is exactly what you need to internalize architecture patterns. Start with the "All Time Favorites" section.

**[ByteByteGo Blog](https://blog.bytebytego.com/)**
Alex Xu (author of System Design Interview books). Well-illustrated explanations of common design problems — URL shorteners, notification systems, rate limiters. Great visual walkthroughs that map directly to interview problems.

**[Martin Fowler's Blog](https://martinfowler.com/)**
Foundational writing on microservices, CQRS, event sourcing, DDD, and distributed patterns. Slower reading but builds deep conceptual models. The articles on "Patterns of Distributed Systems" and "Microservices" are required reading.

**[The Morning Paper](https://blog.acolyer.org/)**
Adrian Colyer's readable summaries of academic distributed systems papers (Dynamo, Bigtable, Spanner, Kafka, Raft). If you want to understand why systems are built the way they are — this is the source.

**[Cloudflare Blog](https://blog.cloudflare.com/)**
Exceptional technical depth on networking, DDoS mitigation, DNS, TLS, CDN edge architecture, and Rust in production. Real incident reports that teach failure modes better than any textbook.

### Company Engineering Blogs

These companies operate at massive scale and write honestly about their architecture decisions. Pick a system you're designing for an interview, find how this company solved a similar problem, and use it as a reference point.

| Blog | What it covers best |
|---|---|
| [Netflix Tech Blog](https://netflixtechblog.com/) | Chaos engineering, CDN/streaming, microservices, Cassandra at scale |
| [Uber Engineering](https://www.uber.com/en-IN/blog/engineering/) | Geospatial systems, real-time dispatch, Kafka, databases, payments |
| [Discord Engineering](https://discord.com/blog/engineering) | WebSockets at scale, Cassandra → ScyllaDB migration, Rust, read replicas |
| [Stripe Engineering](https://stripe.com/blog/engineering) | Idempotency, distributed transactions, API design, Postgres at scale |
| [Dropbox Tech Blog](https://dropbox.tech/) | File sync protocols, block storage, metadata at scale, network optimization |
| [Airbnb Engineering](https://medium.com/airbnb-engineering) | Search, real-time pricing, data infrastructure, service mesh |
| [LinkedIn Engineering](https://engineering.linkedin.com/blog) | Kafka (invented here), feed ranking, graph systems, Espresso DB |
| [DoorDash Engineering](https://doordash.engineering/) | Geospatial routing, ETA prediction, order dispatch, Kafka pipelines |
| [Shopify Engineering](https://shopify.engineering/) | Flash sale traffic handling, database sharding, Rails at scale |
| [Figma Engineering](https://www.figma.com/blog/engineering/) | Real-time collaboration, CRDTs, multiplayer systems, WebAssembly |

### Cloud Provider Architecture Resources

**[AWS Architecture Blog](https://aws.amazon.com/blogs/architecture/)**
Reference architectures, Well-Architected Framework guidance, and real customer case studies. Good for understanding how to wire together cloud primitives.

**[Google Cloud Architecture Center](https://cloud.google.com/architecture)**
Google's own design patterns and reference architectures. Also read the original Google papers: Bigtable (2006), MapReduce (2004), Spanner (2012), Chubby (2006) — many of today's systems are direct descendants.

**[Azure Architecture Center](https://learn.microsoft.com/en-us/azure/architecture/)**
Microsoft's cloud design patterns (Circuit Breaker, Saga, CQRS, Bulkhead, Retry) documented cleanly with diagrams.

### Papers Worth Reading

These academic papers introduced ideas that became standard practice. Read the abstracts and conclusions even if you skip the proofs.

| Paper | Why it matters |
|---|---|
| **Amazon Dynamo (2007)** | Consistent hashing, vector clocks, eventual consistency — foundational for all modern NoSQL |
| **Google Bigtable (2006)** | Wide-column store model; basis for HBase, Cassandra |
| **Google MapReduce (2004)** | Distributed batch processing model |
| **Google Spanner (2012)** | Global distributed SQL with TrueTime; basis for CockroachDB |
| **Apache Kafka (2011, LinkedIn)** | Log-based messaging model that changed event streaming |
| **Raft Consensus (2014)** | Readable alternative to Paxos; used in etcd, CockroachDB, TiKV |
| **Facebook TAO (2013)** | Graph storage and caching at Facebook scale |
| **Twitter Snowflake** | Distributed ID generation — timestamps + machine ID + sequence |

Find them on [the morning paper](https://blog.acolyer.org/) or search the title + "PDF" on Google Scholar.

### Structured Interview Prep

**[System Design Primer (GitHub)](https://github.com/donnemartin/system-design-primer)**
The most-starred system design resource on GitHub. Condensed reference for interviews — concepts, flash cards, solved problems. Good as a checklist, not a replacement for depth.

**[Grokking System Design (Educative)](https://www.educative.io/courses/grokking-modern-system-design-interview-for-engineers-managers)**
Paid course but one of the best structured walkthroughs of classic design problems. Good if you prefer guided learning over free-form reading.

**[InfoQ](https://www.infoq.com/)**
Conference talks and articles from practitioners at major companies. Filter by "Architecture" and "Distributed Systems" for dense, practical content.

### How to Use These Resources

```
Interview in 2 weeks:
  → ByteByteGo + System Design Primer for breadth
  → High Scalability teardowns for the 5 problems you expect

Ongoing learning:
  → Pick one company blog per week, read one post
  → When you use a technology (Kafka, Redis), find the engineering blog post
    that describes how its inventor company uses it in production

Deep understanding:
  → Morning Paper summaries → then skim the original papers
  → Martin Fowler for patterns, Cloudflare for networking, Stripe for payments
```
