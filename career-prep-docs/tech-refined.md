# SDE2 Tech Prep Playbook (Refined)

> **START HERE EVERY DAY → `daily-action-plan.md`** — one consolidated daily checklist.
> All other files are reference material you open only when `daily-action-plan.md` tells you to.
>
> ---
>
> **Core prep**
> - `daily-action-plan.md` — **THE daily checklist** (open this every morning)
> - `tech-refined.md` (this file) — overall plan, schedule, books, summary tables
> - `dsa-prep-plan.md` — DSA 8-week deep-dive (anti-dropout protocol, Java templates, spaced repetition)
> - `lld-prep-plan.md` — LLD 8-week deep-dive (patterns + problems sequence + answer framework)
> - `hld-prep-plan.md` — HLD 8-week deep-dive (9-step framework + vocabulary + canonical problems)
>
> **Job search**
> - `advice-refined.md` — job search execution (HR/HM rounds, applications strategy)
> - `02_linkedin_content-refined.md` — LinkedIn playbook (headline, About, experience, posts, recommendations)
> - `03_resume_content-refined.md` — Resume playbook (1-page strategy, ATS optimization, bullet ranking)
> - `linkedin-posts.md` — 4 publication-ready LinkedIn post drafts
> - `linkedin-cheatsheet.md` — 5-minute LinkedIn refresh one-pager

## Quick Index

- [Profile Snapshot](#profile-snapshot)
- [What To Learn First](#what-to-learn-first-priority-order)
- [Book Stack](#book-stack-high-roi-ordered)
- [14-16 Week Roadmap](#14-16-week-prep-roadmap)
- [Daily Schedule](#daily-operating-schedule-weekdays)
- [DSA Weak-Area Sprint](#dsa-weak-areas---correct-priority)
- [Java Speed Strategy](#java-interview-speed-strategy)
- [System Design Answer Template](#system-design-answer-template-use-every-time)
- [Application Pipeline](#application-pipeline-parallel-to-prep)
- [Company Target Tiers](#company-target-bands-comp--probability-balanced)
- [LLD Design Patterns](#lld-design-patterns-sde2--india-focus)
- [Progress Metrics](#progress-metrics-track-weekly)

---

## Profile Snapshot

- **Target**: SDE2 backend roles (3-5 YOE band), high-comp product companies
- **Current stack**: Java, Spring Boot, Kafka/RabbitMQ basics, Mongo/MySQL, AWS, K8s, Docker
- **Current prep**: DSA + LLD in progress, HLD ramp-up
- **Time budget**:
  - Weekdays: morning `5:00-9:30`, evening `8:00-10:00`
  - Weekends: mostly free
- **Goal split**:
  - **0-4 months**: clear interviews with strong signal
  - **6-12 months**: become reliability/architecture go-to engineer

---

## What To Learn First (Priority Order)

### Must-have for SDE2 interviews

1. **Distributed systems fundamentals**
2. **System design answer structure**
3. **Reliability thinking (production behavior)**
4. **Core DSA patterns (timed execution in Java)**
5. **Messaging/event-driven depth (Kafka patterns)**

### Avoid over-investing right now

- Segment Tree
- Fenwick Tree
- Very niche advanced DS topics

These are low ROI for typical SDE2 interviews unless a specific team demands them.

---

## Book Stack (High ROI, ordered)

## Tier 1: Foundations (Non-negotiable)

1. **Designing Data-Intensive Applications (DDIA)** - Martin Kleppmann  
   Best single source for consistency, replication, partitioning, streams, and storage tradeoffs.

2. **Computer Networking: A Top-Down Approach** - Kurose & Ross  
   Sharpens intuition for latency, HTTP/TCP, DNS, congestion, and real request paths.

3. **Operating Systems: Three Easy Pieces (OSTEP)** - free  
   Essential for concurrency, scheduling, memory, and performance reasoning.

## Tier 2: Interview Execution

4. **System Design Interview (Vol 1 & 2)** - Alex Xu  
   Converts concepts into interview-ready system design communication.

5. **Software Architecture: The Hard Parts** - Richards & Ford  
   Strong for senior tradeoff articulation (sync vs async, consistency vs availability, decomposition).

## Tier 3: Reliability + Production Depth

6. **Release It!** - Michael Nygard  
   Failure patterns and practical resilience controls.

7. **Site Reliability Engineering** - Google SRE book  
   SLIs/SLOs, error budgets, incident response, reliability governance.

## Event-driven specialization (pick based on role demand)

8A. **Kafka: The Definitive Guide** - if Kafka-heavy roles are target  
8B. **Microservices Patterns** - if workflow consistency/outbox/saga questions dominate

---

## Minimal Book Set (If You Want Speed)

If you only commit to 5 paid books:

1. DDIA
2. Kurose & Ross
3. Alex Xu Vol 1
4. Alex Xu Vol 2
5. Release It!

Plus free: OSTEP.

---

## 14-16 Week Prep Roadmap

This keeps OA + LLD + HLD moving in parallel.

### Weeks 1-2: Stabilize basics + start applying

- DSA: arrays, hashing, sliding window, two pointers, binary search
- LLD: Parking Lot, Rate Limiter, Elevator (clean interfaces + extensibility)
- HLD: 2 prompts/week (URL shortener, rate limiter)
- Reading: Networking fundamentals + DDIA intro chapters

### Weeks 3-6: Become interview-usable

- DSA: trees, BFS/DFS, heaps, intervals, backtracking
- LLD: Splitwise, logger, cache
- HLD: 3 prompts/week (chat, feed, notification)
- Reading: DDIA core + Release It! (timeouts/retries/circuit breakers)

### Weeks 7-10: Close major gaps

- DSA: graphs + DP basics + mixed timed sets
- LLD: one full mock/week (with critique)
- HLD: 2 full mocks/week (45 min strict format)
- Reading: SRE fundamentals + selected Kafka chapters

### Weeks 11-16: Interview mode

- DSA: mock-heavy, weak-area drilling
- LLD: 1 mock/week + template refinement
- HLD: 3 mocks/week + one deep dive topic/week
- Applications/referrals become a fixed pipeline

---

## Daily Operating Schedule (Weekdays)

## Morning block (`5:00-9:30`)

- `5:00-6:45` -> DSA timed solving
- `6:45-7:00` -> mistake log (patterns + edge cases)
- `7:00-8:15` -> system design reading (DDIA/Networking/OS rotation)
- `8:15-9:15` -> LLD or HLD practice (alternate days)
- `9:15-9:30` -> recall notes (5 bullets max)

## Evening block (`8:00-10:00`)

- `8:00-9:00`:
  - Mon/Wed/Fri -> HLD prompt
  - Tue/Thu -> LLD prompt
- `9:00-10:00` -> application pipeline + weak-topic revision

---

## Weekend Accelerator

### Saturday

- 1 OA-style mock (70-90 min)
- 1 HLD mock + critique
- Light reading block (DDIA/Kafka/SRE)

### Sunday

- 1 LLD mock + review
- 2 HLD prompts
- Application/referral operations

---

## DSA Weak Areas -> Correct Priority

From current gaps, this order gives best ROI:

1. **Heap / PriorityQueue** (must)
2. **Graph BFS/DFS + Topo** (must)
3. **DP basics** (must)
4. **DSU / Union-Find** (medium-high)
5. **Trie** (medium)

Can skip for now: Fenwick/Segment Tree.

### 4-week algorithm gap-closure sprint

## Week 1
- Heap patterns + binary search on answer

## Week 2
- Graphs I: BFS/DFS/components/grid

## Week 3
- Graphs II: topo + shortest path basics + DSU template

## Week 4
- DP essentials: 1D first, then 2D basics

---

## Java Interview Speed Strategy

You can crack SDE2 loops with Java if execution is standardized.

### Must internalize templates

- BFS/DFS (graph + grid)
- PriorityQueue (min/max + custom comparator)
- Binary search on answer
- Union-Find
- Sliding window
- Top K / heap frequency

### Code-speed rules

- Prefer `ArrayDeque` over `Stack`
- Keep reusable comparator snippets ready
- Maintain one personal "Java templates + mistakes" doc
- Rewrite one solved problem cleanly after each timed attempt

---

## System Design Answer Template (Use Every Time)

1. Clarify requirements + scale
2. Define APIs
3. Data model
4. High-level components
5. Data flow (read/write)
6. Scaling strategy
7. Reliability (retry, timeout, idempotency, DLQ)
8. Tradeoffs + bottlenecks

Consistency in structure is a major interview signal.

---

## Application Pipeline (Parallel to Prep)

## Weekly targets

- `20-30` high-quality applications
- `10-15` referral pings
- Track every role in one sheet:
  - Company
  - Role link
  - Date
  - Referral status
  - Current stage
  - Next action

## Suggested sequencing

- Weeks 1-2: Tier C + select Tier B (warm-up interviews)
- Weeks 3-6: Tier B heavy + start Tier A
- Week 6+: Tier A focus once mock performance is stable

---

## Company Target Bands (Comp + Probability Balanced)

## Tier A (highest upside, harder loops)

- Google, Meta, Uber, Airbnb
- Stripe, Databricks, Snowflake
- Atlassian, Cloudflare, Confluent, ByteDance/TikTok

## Tier B (strong comp + stronger hiring volume)

- Amazon, Microsoft
- Salesforce, Adobe
- LinkedIn, Intuit, PayPal
- Expedia, Booking.com

## Tier C (India high-pay + often faster pipelines)

- Flipkart, PhonePe, Razorpay, Cred
- Swiggy, Meesho, Groww
- Zerodha, Upstox, Dream11, Myntra

## Remote-friendly additions (availability varies)

- GitLab, Automattic, Elastic, HashiCorp, Canonical

---

## Progress Metrics (Track Weekly)

- DSA timed accuracy %
- Number of full OA mocks completed
- LLD mocks done + review quality
- HLD mocks done + design clarity score
- Applications sent
- Referrals requested
- Interviews scheduled

If a metric stalls for 2 weeks, adjust plan immediately.

---

## LLD Design Patterns (SDE2 — India Focus)

You do not need all 23. Master **10 patterns to code** + **3 to recognize**. They cover ~95% of all LLD problems at Flipkart, PhonePe, Cred, Razorpay, Swiggy, Amazon India, etc.

### Tier 1 — Absolute must (appear in almost every LLD problem)

| Pattern | What it does | Where it shows up |
|---|---|---|
| **Singleton** | One global instance | Logger, Config, DB Connection Pool, Cache |
| **Strategy** | Swappable algorithms/behaviors | Payment types, Rate limiter algorithms, Discount engine |
| **Observer** | Notify many objects when state changes | Notification system, Event bus, BookMyShow seat updates |
| **Factory** | Create objects without hardcoding the class | Vehicle factory, Payment gateway, Ride type creation |
| **State** | Object changes behavior based on internal state | Vending Machine, ATM, Elevator, Order lifecycle |

### Tier 2 — Should know fluently

| Pattern | What it does | Where it shows up |
|---|---|---|
| **Builder** | Step-by-step complex object construction | Query builder, Request objects, Pizza toppings |
| **Decorator** | Add behavior dynamically without inheritance | Coffee shop, Logging wrapper, Auth middleware |
| **Chain of Responsibility** | Pass request along a handler chain | ATM dispenser, Logger levels, Approval workflow, Middleware |
| **Command** | Encapsulate actions as objects | Text editor undo/redo, Task queue, Remote control |
| **Adapter** | Make incompatible interfaces work together | Payment gateway wrappers, Third-party API integration |

### Tier 3 — Recognize the concept (read once, can identify in problem)

| Pattern | Where it shows up |
|---|---|
| **Composite** | File system (file vs folder), Menu, Org hierarchy |
| **Template Method** | Framework skeleton with overridable steps (payment flow, game loop) |
| **Proxy** | Lazy loading, access control, caching wrapper |

### Confidently SKIP for SDE2 India

Bridge, Flyweight, Interpreter, Mediator, Memento, Visitor, Prototype, Facade, Abstract Factory (just know it's "factory of factories").

These almost never come up in SDE2 loops at Indian product companies. Don't waste time.

---

## LLD Prep Sequence (8-Week Plan)

### LLD Answer Framework (use for EVERY problem)

1. Clarify requirements (3-4 min) — functional + non-functional + out-of-scope
2. Identify entities (2 min) — extract nouns
3. Class diagram + relationships (5-8 min) — interfaces > classes
4. Apply patterns (during step 3) — justify each one in one sentence
5. API / public methods (3-5 min)
6. Concurrency notes (3-5 min) — where threads collide
7. Code the heart (10-15 min) — not everything, just 2-3 critical classes

### Pattern Learning Order (paired with problems)

Already known: Singleton, Factory, Builder.

| # | Next pattern | Problem that teaches it |
|---|---|---|
| 1 | Strategy | Rate Limiter |
| 2 | Observer | Notification System |
| 3 | State | Vending Machine |
| 4 | Chain of Responsibility | Logger / ATM |
| 5 | Decorator | LRU wrapper / Pizza toy |
| 6 | Command | Text Editor undo/redo |
| 7 | Adapter | Payment gateway wrappers |
| 8 | Composite | File System (recognize) |
| 9 | Template Method | Game flow (recognize) |
| 10 | Proxy | LRU with logging (recognize) |

### Week-by-Week Problem Order

#### Phase 1 — Pattern Foundation (Week 1)
1. **Logging Framework** — Singleton + CoR + Strategy
2. **Rate Limiter** — Strategy + Singleton + concurrency
3. **Notification System** — Observer + Strategy + Factory

#### Phase 2 — Multi-Pattern Classics (Week 2-3)
4. **Vending Machine** — State + Strategy (canonical State problem)
5. **Parking Lot** — Strategy + Factory + Singleton + Observer (THE most-asked)
6. **Tic Tac Toe** — Strategy + State + Factory
7. **Snake & Ladders** — Strategy + State + Observer + Factory
8. **ATM Machine** — State + Singleton + CoR (denomination dispenser)

#### Phase 3 — Concurrency-Heavy (Week 4)
9. **LRU Cache** — Decorator/Proxy + Strategy + thread safety
10. **Splitwise** — Strategy + Observer + graph
11. **BookMyShow** — Observer + Factory + State + seat locking

#### Phase 4 — Big Domain (Week 5)
12. **Elevator System** — State + Strategy (SCAN/LOOK) + Observer
13. **Cab Booking (Uber)** — Strategy + Observer + State + Factory
14. **Online Food Ordering** — Strategy + Observer + State + Decorator

#### Phase 5 — Advanced (Week 6)
15. **Text Editor (Undo/Redo)** — Command + Memento
16. **Library / Hotel Booking** — State + Strategy
17. **Stack Overflow** — Composite + Observer + Strategy

#### Phase 6 — Stretch (Week 7-8, optional)
- Stock Exchange / Order Matching
- Concurrent HashMap from scratch
- File System (Composite showcase)
- In-Memory Key-Value Store

### Daily Rhythm

**Design day (60 min)** — 5m re-read, 10m entities + class diagram, 10m patterns + justification, 20m code the heart, 10m concurrency walkthrough, 5m self-critique
**Code day (60 min)** — Full implementation in IDE + unit tests + commit to `lld-prep` repo

### Non-Negotiable Habits

1. Talk out loud while designing
2. Ask 3 clarifying questions before drawing anything
3. Justify every pattern in one sentence
4. Composition over inheritance — always
5. Always mention concurrency, even briefly
6. Never memorize solutions — redo from scratch on paper

### Red Flags to Avoid

- Jumping into code before clarifying
- Over-applying patterns to show off
- Public fields / no encapsulation
- All concrete classes, no interfaces
- Ignoring concurrency
- Designing for imagined future requirements (YAGNI)

### Resources (pick ONE primary)

- **Concept**: *Head First Design Patterns* — read only the chapters for patterns above
- **Problems**: github.com/prasadgujar/low-level-design-primer
- **YouTube**: Concept && Coding (Shreyansh Jain) — best free Indian LLD channel
- **Output**: personal `lld-prep` GitHub repo, one folder per problem

### Pattern-to-Problem Map (use this for prep)

| LLD Problem | Patterns needed |
|---|---|
| Parking Lot | Singleton + Strategy + State |
| Rate Limiter | Singleton + Strategy |
| BookMyShow | Observer + Factory + State |
| Notification System | Observer + Strategy + Factory |
| Splitwise | Strategy + Observer |
| Elevator | State + Observer + Strategy |
| ATM Machine | State + Singleton + **Chain of Responsibility** (cash dispenser) |
| LRU Cache | Decorator/Proxy + Strategy |
| Text Editor | Command (undo/redo) + Memento (state snapshots) |
| Payment System | Strategy + Factory + Adapter |
| Logger | Singleton + **Chain of Responsibility** (log levels) |
| File System | **Composite** + Visitor (skip) |
| Chess / Snakes & Ladders | Strategy + State + Factory |

### How to practice each pattern

1. Write the interface/abstract class
2. Write 2 concrete implementations
3. Wire it into a small LLD problem class design
4. Explain: "I used Strategy here so I can swap X without changing the caller"

The verbal explanation in the interview matters as much as the code.

---

## Final Focus Rule

Do not optimize for reading volume.  
Optimize for:

- timed execution
- repeatable templates
- mock interview performance
- ability to explain tradeoffs clearly

That is what converts prep effort into SDE2 offers.

