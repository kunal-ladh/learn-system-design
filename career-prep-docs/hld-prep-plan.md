# HLD / System Design Prep Plan — SDE2 Mentor Playbook

> **For**: 5 YOE backend engineer with real Kafka/MSK + microservices + AWS + K8s + MongoDB production experience but **crude HLD vocabulary** (no formal DBMS/OS/Networks in college).
> **Reality check**: You know more than you think. The gap is **structure + vocabulary**, not concepts. Your fear is overblown.
> **Goal**: Clear HLD rounds at SDE2 product companies in 8 weeks.

---

## Table of Contents

1. [The Most Important Thing](#1-the-most-important-thing-to-internalize)
2. [Your Unfair Advantage](#2-your-unfair-advantage-use-it)
3. [The HLD Answer Framework](#3-the-hld-answer-framework-9-steps)
4. [Building Blocks Vocabulary](#4-building-blocks-your-vocabulary-list)
5. [8-Week Plan](#5-the-8-week-plan)
6. [Daily Rhythm](#6-daily-rhythm)
7. [Use Your Grokking Folder](#7-use-your-grokking-folder)
8. [Production Stories Map](#8-where-your-production-stories-plug-in)
9. [Mocks Strategy](#9-mocks-strategy)
10. [Resources](#10-resources-pick-one-primary)
11. [Self-Assessment Checkpoints](#11-self-assessment-checkpoints)

---

## 1. The Most Important Thing To Internalize

HLD interviews are **not** "do you know how DNS works" tests.

They are: *"Can you take a vague product, decompose it into components, justify trade-offs, and communicate clearly within 45 minutes?"*

You will lose more interviews to **bad communication** than to bad knowledge. Even with imperfect vocabulary, if you structure your answer well, you clear bar.

**The gap year, the EEE background, the lack of formal CS — none of this is visible to an interviewer if you:**
1. Use the 9-step framework
2. Speak with clarity and structure
3. Drop 2-3 real production stories during the design

That's it. That's the test.

---

## 2. Your Unfair Advantage (Use It)

You have things 80% of SDE2 candidates don't:

| Asset | What it lets you do in an interview |
|---|---|
| **Production Kafka experience** | Discuss partitioning, consumer groups, rebalancing, offset management with authority |
| **AWS in anger** (MSK, S3, SQS, Lambda, EventBridge, K8s, ASM) | Reference real services, not textbook names |
| **Built 7+ microservices from scratch** | Talk about boundaries, decomposition, service-to-service patterns |
| **Production migrations** (Spring Boot 2→3, Facets, SDK v2→v3) | Discuss backwards compatibility, dual writes, blue-green |
| **Real failures debugged** (Kafka rebalancing, NPEs, async issues) | Reliability stories carry weight |
| **Banking domain** | Idempotency, audit trails, retries, eventual consistency naturally come up |

**The mantra**: *"When I built X at Niyo, we picked Y because Z"* — that single sentence puts you ahead of 80% of candidates. Most SDE2 candidates have zero production experience and bullshit through. You have receipts.

**Use 2-3 stories per HLD round, not more.** Don't make it about you — make it about the design.

---

## 3. The HLD Answer Framework (9 Steps)

Use for **every** problem. Drill until it's automatic. Write the 9 step labels on the whiteboard first, then fill them in.

### Step 1: Functional Requirements (3-4 min)
"What does the system need to do?"
- Ask 5 clarifying questions
- List 4-6 core features explicitly
- Confirm scope with interviewer

### Step 2: Non-Functional Requirements (2 min)
- Scale (DAU, requests/sec)
- Latency targets (p50, p99)
- Availability target (3 nines? 4 nines?)
- Consistency needs (strong? eventual?)
- Read-heavy or write-heavy?

### Step 3: Capacity Estimation (3 min)
Back-of-envelope:
- DAU × actions/day / 86400 = QPS
- Average payload × QPS × seconds in day = daily storage
- Project 5 years for storage planning

### Step 4: API Design (3 min)
- 3-5 REST endpoints with shapes
- Request/response JSON
- Auth + idempotency where it matters

### Step 5: Data Model (3 min)
- Entities + key fields + which DB type (SQL vs NoSQL — justify)
- Mention indexes you'd create
- Mention sharding key if relevant

### Step 6: High-Level Architecture Diagram (5 min)
- Client → CDN → LB → API GW → Services → DB / Cache / Queue
- Draw boxes + arrows
- Talk through the read path AND write path

### Step 7: Deep-Dive One Component (8-10 min)
- Pick the meatiest piece — usually the one most under load OR the one with the trickiest data flow
- Often the interviewer steers you here
- This is where you SHINE — go deep

### Step 8: Scaling + Bottlenecks (5 min)
- "Where does this break at 10x scale?"
- Caching strategy
- DB sharding strategy
- Async vs sync work
- Hot partition risk

### Step 9: Trade-offs + Extensions (3-4 min)
- "If we needed X, we'd add Y"
- Explicit trade-offs you made (CAP, consistency vs latency)
- What you'd monitor in prod (SLIs)

> Memorize the 9 labels. In every mock, write them on the whiteboard FIRST before saying anything.

---

## 4. Building Blocks (Your Vocabulary List)

Master these one at a time over Week 1-3. **You don't need depth — you need vocabulary + when to use each.**

### Tier 1 — Must Know Cold

| Block | What to know | Your existing knowledge |
|---|---|---|
| **HTTP / TCP** | Methods, status codes, keep-alive, TLS handshake | Solid from work |
| **DNS** | What it does, latency, caching, TTL | Refresh |
| **CDN** | Edge caching, static vs dynamic, when to use | Light |
| **Load Balancer** | L4 vs L7, round-robin / least-conns / consistent-hash, health checks | You've configured K8s services — solid |
| **API Gateway** | Auth, rate-limit, routing, request transformation | Solid from work |
| **SQL vs NoSQL** | When to pick each. ACID vs BASE. | You use both — solid |
| **Indexes** | B-tree, what indexes cost on writes, composite indexes | Refresh |
| **Sharding** | Hash-based, range-based, consistent hashing | Conceptual gap — fill |
| **Replication** | Leader-follower, multi-leader, leaderless; read replicas | Conceptual gap — fill |
| **Caching** | Redis vs Memcached, write-through / back / aside, eviction | Light — fill |
| **Message Queue / Stream** | Kafka, SQS, RabbitMQ — when each; partitioning, ordering, retention | **Your strongest area** |
| **Object Storage** | S3, Blob — for files, images, logs | Solid (you use S3 daily) |

### Tier 2 — Know the Name + 1-Line Summary

| Block | One-liner |
|---|---|
| **Zookeeper / etcd** | Distributed coordination, leader election, config |
| **Consistent Hashing** | Distribute keys to nodes with minimal reshuffling on add/remove |
| **Bloom Filter** | Space-efficient "is this maybe in the set?" check |
| **CAP Theorem** | In a partition, choose Consistency or Availability |
| **PACELC** | CAP + "even without partition, choose Latency or Consistency" |
| **Quorum** | R + W > N for consistency in leaderless replication |
| **WAL** | Write-ahead log — durability + replication foundation |
| **CDC** | Change Data Capture — stream DB changes to consumers |
| **Saga / Outbox** | Distributed transaction patterns (you've used these implicitly) |
| **Idempotency** | Same request twice = same effect once |
| **Backpressure** | Slow consumers signal upstream to slow down |
| **Circuit Breaker** | Fail fast when downstream is broken |
| **Geohash / QuadTree** | Indexing for location-based queries |

### Tier 3 — Recognize the Term (Don't Memorize)

Vector clocks, gossip protocols, MVCC, LSM trees vs B-trees, Paxos / Raft (high level only), HyperLogLog, inverted indexes, Lambda Architecture, Kappa Architecture.

---

## 5. The 8-Week Plan

### Phase 1: Foundations + Framework (Week 1-2)

> Goal: Build vocabulary. Start using the 9-step framework even on easy problems.

#### Week 1 — Networking + Infra Vocabulary
- **Day 1-2**: Read Grokking PDFs — `Introduction`, `HTTP-related`, `Long-Polling vs WebSockets vs SSE`. Watch ByteByteGo videos on each (15 min each).
- **Day 3-4**: Grokking PDFs — `LoadBalancing` (both files), `Proxies`.
- **Day 5-7**: Grokking PDFs — `Caching`, `Indexes`, `SQL vs NoSQL`. **DDIA Ch 1** (skim).

#### Week 2 — Data + Distributed Basics
- **Day 8-9**: Grokking PDFs — `Data Partitioning`, `Consistent Hashing`, `Redundancy & Replication`. **DDIA Ch 5** (skim).
- **Day 10-11**: Grokking PDFs — `CAP`, `Key Characteristics of Distributed Systems`.
- **Day 12-14**: **First HLD problem — URL Shortener**. Apply full 9-step framework. Time-box 60 min. Watch one solution video AFTER your attempt.

### Phase 2: Easy Problems With Framework (Week 3-4)

> Goal: Make the framework automatic. Easy problems give you space to focus on structure.

| Day | Problem | Why this one |
|---|---|---|
| 15-16 | **URL Shortener** (redo, cleaner) | Foundation — base62/hash, DB choice, caching, ID generation |
| 17-18 | **Pastebin** | Object storage + metadata DB + TTL + access control |
| 19-20 | **Rate Limiter** (HLD-style) | Connects to LLD work — distributed token bucket, Redis-based |
| 21-23 | **Web Crawler** | Queues, distributed workers, dedup with Bloom filter, politeness |
| 24-26 | **Notification System** | You've literally built one — articulate it via framework |
| 27-28 | **Mock #1** (self-recorded) | 45-min strict, pick from above |

### Phase 3: Medium Problems + DDIA Core (Week 5-6)

> Goal: handle multi-component systems. Read DDIA chapters that matter.

**DDIA reading (parallel)**:
- Week 5: Ch 5 (Replication), Ch 6 (Partitioning)
- Week 6: Ch 7 (Transactions, just key sections), Ch 11 (Stream Processing — your home turf)

| Day | Problem | Why |
|---|---|---|
| 29-30 | **Instagram / Photo Feed** | Newsfeed fanout (push/pull/hybrid), media storage, CDN |
| 31-32 | **Twitter (Read-heavy)** | Pull vs push timeline, fanout-on-write vs read |
| 33-34 | **WhatsApp Messenger** | WebSockets, message ordering, delivery receipts, presence |
| 35-36 | **News Feed** | Ranking algorithm, caching, fanout strategies |
| 37-38 | **Dropbox / Google Drive** | Chunking, dedup, sync, conflict resolution, metadata |
| 39-40 | **YouTube / Netflix** | Video upload pipeline, transcoding, CDN, HLS/DASH |
| 41-42 | **Mocks #2 + #3** | 2 mocks, 45 min each — start finding human peers |

### Phase 4: Hard Problems + Mocks (Week 7-8)

> Goal: confidence on novel problems. Mocks are where you fix communication.

| Day | Problem | Why |
|---|---|---|
| 43-44 | **Uber / Cab Booking** | Geo-indexing (Geohash/QuadTree), real-time matching, surge pricing |
| 45-46 | **Search Typeahead** | Trie at scale, ranking, caching popular prefixes |
| 47-48 | **Distributed Cache** | Consistent hashing, eviction, replication, hot key |
| 49-50 | **Stock Exchange / Order Matching** | Order book, matching engine, latency-critical design |
| 51-52 | **Distributed Job Scheduler** | Cron at scale, leader election, retries, idempotency |
| 53-56 | **Mocks #4, #5, #6** | 3 mocks in final week. Get human feedback. |

---

## 6. Daily Rhythm

Slot into your existing schedule (`tech-refined.md`):
- **Morning 7:00-8:15 AM** (75 min) — concept reading + design
- **Evening Mon/Wed/Fri 8-9 PM** — full HLD problem with framework

### Morning Block (75 min)
| Time | Activity |
|---|---|
| 7:00-7:10 | Re-read previous day's deep-dive notes |
| 7:10-8:10 | Active problem-solving OR concept reading |
| 8:10-8:15 | Write 3 bullets — "today's takeaways" |

### Evening HLD Slot (60 min, Mon/Wed/Fri)
| Time | Activity |
|---|---|
| 0-45 min | Strict 45-min solo design using framework |
| 45-60 min | Self-critique: watch a solution video, write what you missed |

### Weekends (3-4 hours total)
- Saturday: 1 problem with framework (60 min) + DDIA reading (60 min) + 1 hour skill mixing
- Sunday: 1 mock with peer (60-90 min) + buffer + apply to jobs

---

## 7. Use Your Grokking Folder

You **already have** `Grokking-System-Design/` in this repo. Use it ruthlessly.

### Reading Order

| Week | Read |
|---|---|
| 1 | `Introduction.pdf`, `Long-Polling vs WebSockets vs SSE.pdf`, `LoadBalancing.pdf`, `LoadBalancing - Link 1.pdf`, `Load Balancing - Link 2.pdf`, `Proxies.pdf` |
| 2 | `Caching.pdf`, `SQL vs No-SQL.pdf`, `Indexes.pdf`, `Data Partitioning.pdf`, `Consistent Hashing.pdf`, `Redundancy & Replication.pdf` |
| 3 | `CAP.pdf`, `Key Characteristics of Distributed Systems.pdf` |
| 3-8 | One `Problems/` folder per problem you're attacking |

### Problems Folder Mapping

Your `Grokking-System-Design/Problems/` has solutions for:
- `URL Shortener.pdf` → Week 3
- `Pastebin.pdf` → Week 3
- `Rate Limiting.pdf` → Week 3
- `Web Crawler.pdf` → Week 3
- `Instagram.pdf` → Week 5
- `Twitter.pdf` → Week 5
- `Twitter Search.pdf` → Week 5
- `FB Newsfeed.pdf` → Week 5
- `Messenger.pdf` → Week 5
- `Dropbox.pdf` → Week 5
- `Youtube_Netflix.pdf` → Week 5
- `Yelp.pdf` → Week 7 (similar to Uber, location-based)
- `QuadTree.pdf` → Week 7 (read before Uber)
- `TypeAhead Suggestion.pdf` → Week 7
- `TicketMaster.pdf` → Week 7 (similar to BookMyShow)
- `Steps.pdf` → Read first (general approach)

**Stop hoarding. Read and apply.**

---

## 8. Where Your Production Stories Plug In

These are your war stories. Practice deploying them naturally in HLD rounds.

| HLD Topic | Your Story to Deploy |
|---|---|
| **Kafka partitioning + consumer groups** | "At Niyo we migrated Bharat services from Confluent to AWS MSK with ZSTD compression…" |
| **Consumer rebalancing** | "We had a Kafka rebalancing issue in Bharat VAS where the consumer group kept rebalancing on each deployment. We fixed it by tuning session timeout and max poll interval…" |
| **Idempotency** | "For fund transfers in Utkarsh we had a duplicate detection window per beneficiary…" |
| **Async / event-driven** | "Our AOF PDF generation pipeline was async — request enqueued to Kafka, worker pulls and generates, uploads via SFTP, then triggers callback…" |
| **CDC / Outbox** | "For agent activity tracking we used Kafka events emitted by the service after DB commit…" |
| **Real-time push** | "We used SSE for fund transfer status updates so the user app gets live updates without polling…" |
| **Distributed locking** | "When discussing seat booking — we'd use either pessimistic row locks or Redis-based distributed locks with TTL…" |
| **API design + BFF** | "We built a BFF layer in Utkarsh because the mobile clients needed different aggregations than the web admin panel…" |
| **Migration / dual-write** | "When we did Spring Boot 2→3 we ran both versions in parallel for a window with feature flags…" |
| **Security at scale** | "For Bharat we moved secrets to AWS Secrets Manager + K8s secrets, added Trivy scanning in CI…" |

**Multi-region**: Be honest — say "I haven't directly architected multi-region, but conceptually you'd…"
**Geo-distributed**: Same — honest.

Honesty about gaps is a senior signal. Faking it is fatal.

---

## 9. Mocks Strategy

Mocks fix **communication**, not knowledge.

| When | What |
|---|---|
| End of Week 3 | Self-recorded mock — record on phone, watch back. Painful but essential. |
| End of Week 4 | Mock #1 with self-recording (URL Shortener or Notification) |
| End of Week 6 | Mocks #2 + #3 — find peers (Pramp, Interviewing.io free tier, Reddit r/cscareerquestions India HLD swap groups, Discord) |
| End of Week 8 | Mocks #4 + #5 + #6 — pay for a senior mentor for 1-2 sessions if you can afford ₹1500-3000 each. Worth it. |

### After Every Mock, Write Down
1. What I did well
2. What I missed / messed up
3. What new pattern I learned
4. What I'd do differently next time

This log becomes priceless by Week 6.

---

## 10. Resources (Pick ONE Primary)

| Type | Pick | Why |
|---|---|---|
| Book — primary | **DDIA** (only Ch 1, 5, 6, 7, 11 for now) | The single best HLD foundation |
| Problem source | **Your existing Grokking folder** | Already in this repo |
| Video — primary | **ByteByteGo YouTube** (free) | Best visual explanations |
| Video — backup | **Gaurav Sen YouTube** (HLD playlist) | Indian English, real prod context |
| Trade-off framing | **Software Architecture: The Hard Parts** (skim only) | Senior trade-off vocabulary |
| Mock partners | **Pramp** (free), **Interviewing.io** (paid for sr mentors) | Real feedback |
| Real-world reading | **High Scalability blog** (highscalability.com) — 1 article/week | How real companies actually scale |

**Stop hoarding. No second System Design book until you finish Grokking + DDIA core chapters.**

---

## 11. Self-Assessment Checkpoints

### End of Week 2
- [ ] Can recite the 9-step framework from memory
- [ ] Can explain Tier 1 vocabulary (12 items) in 1 sentence each
- [ ] Done first HLD problem end-to-end (URL Shortener)
- [ ] Read Grokking PDFs from Week 1 + 2 list

### End of Week 4
- [ ] Done 5+ easy/medium HLD problems with framework
- [ ] DDIA Ch 5 + 6 read
- [ ] Mock #1 completed (self-recorded — watched the recording!)
- [ ] Can drop 2 "at Niyo we did X" stories naturally without forcing them
- [ ] Applied to 10+ companies (parallel to DSA prep)

### End of Week 6
- [ ] Done 10+ HLD problems
- [ ] DDIA Ch 7 + 11 read
- [ ] 2 mocks with humans
- [ ] Can sketch architecture for Twitter / Instagram / Messenger from memory

### End of Week 8
- [ ] 15+ HLD problems done
- [ ] 3+ human mocks
- [ ] Confident with framework + Tier 1 + Tier 2 vocabulary
- [ ] Real SDE2 interview rounds scheduled
- [ ] Cleared at least 1 HLD round in a real interview

---

## One-Page Cheat Sheet

```
THE 9-STEP FRAMEWORK:
  1. Functional reqs  (3-4 min) — ask 5 questions
  2. Non-functional   (2 min)   — scale, latency, availability
  3. Capacity est     (3 min)   — QPS, storage
  4. APIs             (3 min)   — 3-5 REST endpoints
  5. Data model       (3 min)   — entities + DB choice
  6. Architecture     (5 min)   — boxes + arrows
  7. Deep dive        (8-10 min) — meatiest component
  8. Scaling          (5 min)   — bottlenecks at 10x
  9. Trade-offs       (3-4 min) — what we'd add for X

TIER-1 VOCABULARY (must know cold):
  HTTP/TCP, DNS, CDN, LB, API Gateway, SQL vs NoSQL,
  Indexes, Sharding, Replication, Caching, MQ/Stream,
  Object Storage

TIER-2 VOCABULARY (name + 1-liner):
  Zookeeper, Consistent Hashing, Bloom Filter, CAP,
  PACELC, Quorum, WAL, CDC, Saga/Outbox, Idempotency,
  Backpressure, Circuit Breaker, Geohash

YOUR ACES TO PLAY:
  Kafka MSK migration, consumer rebalancing,
  Fund transfer idempotency, AOF async pipeline,
  Agent activity tracking (Outbox-style), SSE push,
  Spring Boot 2→3 dual-write migration

GOLDEN PHRASES:
  "When I built X at Niyo we picked Y because Z"
  "Single-region works, for multi-region we'd need..."
  "We could go strong consistency here, but the
   latency cost would be ~X — eventual is fine for
   this product"
  "I haven't done this in prod, but conceptually..."

THE GOLDEN RULE:
  Bad knowledge < Bad structure.
  Memorize the framework. Vocabulary follows.
```

---

*Companion docs: `tech-refined.md` (overall plan), `dsa-prep-plan.md` (DSA), `lld-prep-plan.md` (LLD), `advice-refined.md` (job search).*
