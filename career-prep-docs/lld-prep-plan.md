# LLD Prep Plan — SDE2 / SSE Mentor Playbook

> **For**: 5 YOE backend engineer targeting SDE2 at high-comp Indian + global product companies.
> **Starting point**: SOLID done, basic concurrency done, HashMap/ArrayList impl done, Singleton + Factory + Builder known.
> **Goal**: Crack LLD rounds at Flipkart, PhonePe, Cred, Razorpay, Swiggy, Amazon India, Atlassian, Uber India, etc.

---

## Table of Contents

1. [Mental Model First](#1-mental-model-first-dont-skip)
2. [The LLD Answer Framework](#2-the-lld-answer-framework-use-for-every-problem)
3. [Pattern Learning Order](#3-pattern-learning-order-paired-with-problems)
4. [8-Week Problem Sequence](#4-the-8-week-lld-problem-order)
5. [Daily Rhythm](#5-daily-rhythm)
6. [Non-Negotiable Habits](#6-non-negotiable-habits)
7. [Red Flags to Avoid](#7-red-flags-interviewers-watch-for)
8. [Resources](#8-resources-dont-drown-in-these)
9. [Self-Assessment Checkpoints](#9-self-assessment-checkpoints)

---

## 1. Mental Model First (Don't Skip)

LLD interviews are **not** "do you know 23 patterns" tests.

They are **"can you take a vague problem and produce clean, extensible, thread-safe class design in 45 minutes"** tests.

### The two failure modes

| Failure mode | What it looks like | How to avoid |
|---|---|---|
| **Under-engineering** | Public fields, no interfaces, monolithic class | Apply SOLID, default to interfaces |
| **Over-engineering** | 5 patterns where 2 would do, abstract factories everywhere | Justify each pattern; remove if unjustified |

Patterns are tools you reach for **when needed**, not a checklist you cram in. The mistake 90% of candidates make: they over-apply patterns to look smart, and the interviewer marks them down for over-engineering.

### How to actually learn patterns

You learn them **through problems**, not in isolation. After 8-10 problems, patterns will start clicking on their own. Don't sit and read all 23 patterns — you'll forget them in a week.

---

## 2. The LLD Answer Framework (Use For EVERY Problem)

Memorize this 7-step structure. Use it in mocks. Use it in interviews. Never deviate.

### Step 1: Clarify requirements (3-4 min)
- **Functional**: what does it do? Ask 3-5 questions.
- **Non-functional**: scale, concurrent users, latency expectations
- **Out-of-scope**: explicitly state what you're NOT designing (payments, auth, etc.)

### Step 2: Identify entities (2 min)
- Extract nouns from requirements
- Note relationships: "has-a", "is-a", "uses-a"

### Step 3: Class diagram + relationships (5-8 min)
- Inheritance vs composition (prefer composition)
- Interfaces over concrete classes
- Draw on paper or whiteboard

### Step 4: Apply patterns (during step 3)
- Justify each pattern in **one sentence**
- "I'm using Strategy here because we may want to swap X without touching Y"

### Step 5: API / public method signatures (3-5 min)
- Names + parameters + return types
- Skip implementation

### Step 6: Concurrency notes (3-5 min)
- Where could threads collide?
- Locks, `synchronized`, `AtomicInteger`, `ConcurrentHashMap`, optimistic vs pessimistic

### Step 7: Code the heart (10-15 min)
- Don't code everything — just the **critical 2-3 classes**
- Show clean, well-named, well-encapsulated code

> The interviewer wants to see this structure. Skipping step 1 = instant red flag.

---

## 3. Pattern Learning Order (Paired With Problems)

You already have **Singleton, Factory, Builder**. Pick up the rest in this order. Each pattern is paired with the smallest problem that teaches it cleanly.

| # | Pattern | Tier | Smallest problem | Why this order |
|---|---|---|---|---|
| 1 | **Strategy** | 1 | Rate Limiter | Most important next pattern. Touches everything. |
| 2 | **Observer** | 1 | Notification System | Pairs with Strategy in real designs |
| 3 | **State** | 1 | Vending Machine | Critical for state-machine problems |
| 4 | **Chain of Responsibility** | 2 | Logger | Easy to grasp, frequent in interviews |
| 5 | **Decorator** | 2 | Pizza shop toy → LRU wrapper | Conceptual, easy |
| 6 | **Command** | 2 | Text Editor undo/redo | Single-purpose, easy once you see it |
| 7 | **Adapter** | 2 | Payment gateway wrappers | Trivial concept, useful vocabulary |
| 8 | **Composite** | 3 | File System | Recognize-only, 30 min |
| 9 | **Template Method** | 3 | Game flow / Payment processor skeleton | Recognize-only, 20 min |
| 10 | **Proxy** | 3 | LRU with access logging | Recognize-only, 20 min |

### Patterns to confidently SKIP

Bridge, Flyweight, Interpreter, Mediator, Memento (except for Text Editor), Visitor, Prototype, Facade, Abstract Factory (just know it's "factory of factories").

These almost never come up in SDE2 loops in India. Don't waste time on them.

---

## 4. The 8-Week LLD Problem Order

Estimated pace: **1 problem per 2 days** (one day designing, one day coding + critique).

### Phase 1 — Pattern Foundation (Week 1)

> Goal: lock in Strategy, Observer, Chain of Responsibility on small problems where the pattern *is* the problem.

| Day | Problem | Patterns | Key learnings |
|---|---|---|---|
| 1-2 | **Logging Framework** | Singleton + CoR + Strategy | CoR for log-level chain (DEBUG → INFO → WARN → ERROR), Strategy for appenders (File/Console/Network). Tiny domain, focus on patterns. |
| 3-4 | **Rate Limiter** | Strategy + Singleton | Token Bucket vs Leaky Bucket vs Fixed Window vs Sliding Window. Add concurrency (`AtomicLong` counter). Perfect Strategy training. |
| 5-6 | **Notification System** | Observer + Strategy + Factory | Multi-channel (SMS/Email/Push) = Strategy. Subscribers = Observer. Factory for channel creation. You'll build this with eyes closed given your work. |
| 7 | **Review** | — | Re-implement Rate Limiter from scratch without notes. Write a pattern summary doc. |

### Phase 2 — Multi-Pattern Classics (Week 2-3)

> Goal: handle 3-4 patterns in one design + introduce State.

| Day | Problem | Patterns | Key learnings |
|---|---|---|---|
| 8-9 | **Vending Machine** | State (idle → coin → select → dispense) + Strategy (payment) | THE canonical State problem. Must do. State transitions matter. |
| 10-12 | **Parking Lot** | Strategy (spot allocation) + Factory (vehicle types) + Singleton (lot) + Observer (spot freed) | THE most-asked LLD problem in India. Do it **twice** — once basic, once with multi-floor + concurrency. |
| 13-14 | **Tic Tac Toe** | Strategy (win check) + State + Factory | Small enough to finish in one sitting. Good warmup. |
| 15-16 | **Snake & Ladders** | Strategy (dice) + State + Observer (turns) + Factory (board entities) | Step up from TTT. |
| 17-18 | **ATM Machine** | State + Singleton + CoR (denomination dispenser ₹2000 → ₹500 → ₹100) | The CoR use case becomes obvious here. |
| 19-21 | **Mock LLD #1 + buffer** | — | Time-boxed 45-min mock. Brutal self-critique. |

### Phase 3 — Concurrency-Heavy (Week 4)

> Goal: real thread-safety. This is where SDE2 candidates separate from SDE1.

| Day | Problem | Patterns + concepts | Key learnings |
|---|---|---|---|
| 22-23 | **LRU Cache** | Decorator/Proxy + Strategy (eviction) + `ConcurrentHashMap` + read-write locks | Asked at every FAANG India + Cred/Razorpay. HashMap + Doubly Linked List combo. Must be fluent. |
| 24-25 | **Splitwise** | Strategy (equal/exact/percentage split) + Observer (balance updates) + graph for settlement | Classic. Lead with the data model. |
| 26-28 | **BookMyShow / Movie Booking** | Observer + Factory + State + **distributed locking on seats** | The big one. Concurrency on seat selection is the actual test. Discuss optimistic vs pessimistic locking. |

### Phase 4 — Big Domain Models (Week 5)

> Goal: handle large entity graphs without losing structure.

| Day | Problem | Patterns | Key learnings |
|---|---|---|---|
| 29-30 | **Elevator System** | State + Strategy (SCAN/LOOK scheduling) + Observer | Multi-elevator scheduling is the trick. |
| 31-32 | **Cab Booking (Uber/Ola)** | Strategy (pricing, matching) + Observer + State (trip lifecycle) + Factory (cab type) | Connects to fintech/marketplace experience. |
| 33-34 | **Online Food Ordering (Swiggy)** | Strategy + Observer + State + Factory + Decorator (offers) | End-to-end flow practice. |
| 35 | **Mock LLD #2** | — | Pick a problem you haven't done. Time it. |

### Phase 5 — Advanced (Week 6)

> Goal: command-style problems + unfamiliar domains.

| Day | Problem | Patterns | Key learnings |
|---|---|---|---|
| 36-37 | **Text Editor (with Undo/Redo)** | **Command** + Memento (state snapshots) | The ONE problem where Command is the answer. |
| 38-39 | **Library / Hotel Booking** | State + Strategy + reservation conflicts | Easier than BMS, good consolidation. |
| 40-41 | **Stack Overflow / Reddit (LLD)** | Composite (comments tree) + Observer (notifications) + Strategy (ranking) | Tree structures + voting. |
| 42 | **Mock LLD #3** | — | 45 min strict. |

### Phase 6 — Stretch (Week 7-8, optional)

Pick based on company. Skip if you're already interviewing well.

- **Stock Exchange / Order Matching** — heavy concurrency, priority queues
- **Concurrent HashMap from scratch** — for infra-leaning companies (Confluent, MongoDB, Databricks)
- **File System** — Composite pattern showcase
- **In-Memory Database / Key-Value Store** — bridges into HLD
- **Distributed Job Scheduler (LLD parts)** — Cron, retry, idempotency

---

## 5. Daily Rhythm

Slot into your existing `tech-refined.md` schedule: **8:15-9:15 morning block** on alternate days + Tue/Thu evenings.

### Design day (60 min)

| Time | Activity |
|---|---|
| 0-5 min | Re-read problem statement, ask yourself clarifying questions |
| 5-15 min | List entities + draw class diagram on paper |
| 15-25 min | Pick patterns + justify each one **out loud** |
| 25-45 min | Code the heart (2-3 critical classes) |
| 45-55 min | Concurrency walkthrough |
| 55-60 min | Write 5 bullets — "what I'd improve / what I missed" |

### Code day (60 min)

- Fully implement in Java in your IDE
- Run unit tests for at least the core flow
- Commit to a personal `lld-prep` repo — **one folder per problem**

---

## 6. Non-Negotiable Habits

1. **Talk out loud while designing.** Interviews are verbal. Silent thinking ≠ communicated thinking. Record yourself once a week and listen back.
2. **Ask 3 clarifying questions before drawing anything.** Diving in is the #1 candidate failure mode.
3. **Justify every pattern in one sentence.** "I'm using Strategy for parking spot allocation because we may want to swap nearest-first vs cheapest-first without touching the lot class."
4. **Never use inheritance when composition works.** Default to interfaces + composition. Saying *"prefer composition over inheritance"* is an SDE2 signal.
5. **Always discuss concurrency, even if briefly.** *"In a single-threaded version this is fine, but if multiple users book simultaneously we'd need either pessimistic row-level locks or optimistic with version checks."* That sentence alone shifts you from SDE1 to SDE2.
6. **Don't memorize solutions.** If you've seen Parking Lot 5 times, do it from scratch on paper without referencing anything.
7. **Write tests for the core flow.** Even a 5-line `@Test` per problem proves you think about testability.

---

## 7. Red Flags Interviewers Watch For

| Red flag | Fix |
|---|---|
| Jumping into code without clarifying | Always start with 3 questions |
| Using 5 patterns when 2 would do | Justify each pattern; remove if unjustified |
| Public fields everywhere | Encapsulation. Always. |
| No interfaces, all concrete classes | Default to `interface PaymentStrategy`, then `class CardPayment implements PaymentStrategy` |
| Forgetting nulls / edge cases | Mention them even if you don't code them |
| No mention of concurrency | One sentence on it, minimum |
| Designing for imagined future needs | YAGNI. Solve the asked problem. |
| Reading from prepared notes | Practice without notes from day 1 |
| Vague class diagrams | Specific names, types, relationships |
| Long monologues, no checkpoints | Pause every 5 min, ask "does this approach make sense?" |

---

## 8. Resources (Don't Drown In These)

Pick **ONE** primary, don't channel-hop:

### Concept
- **Head First Design Patterns** (Freeman & Robson) — best book for patterns, Java-based, story-driven. Don't read cover-to-cover; read only the chapters for patterns in section 3 above.

### Problems
- [github.com/prasadgujar/low-level-design-primer](https://github.com/prasadgujar/low-level-design-primer) — comprehensive problem list with starter code
- Coding Ninjas LLD course (paid, if you prefer guided)
- Awesome Low Level Design — [github.com/ashishps1/awesome-low-level-design](https://github.com/ashishps1/awesome-low-level-design)

### Video
- **Concept && Coding** (Shreyansh Jain) on YouTube — best free Indian channel for LLD interview prep. Watch one pattern video, then code it immediately. Don't binge.

### Output
- **Your own GitHub repo** is worth more than 10 books. Make one called `lld-prep`. One folder per problem. README per folder explaining: problem, patterns used, trade-offs, what you'd improve.

---

## 9. Self-Assessment Checkpoints

Run these every 2 weeks. If you fail any, slow down and consolidate.

### End of Week 2

- [ ] Can implement Rate Limiter (Token Bucket) from scratch in 30 min
- [ ] Can explain Strategy pattern with 2 real examples without notes
- [ ] Have committed at least 3 problems to `lld-prep` repo

### End of Week 4

- [ ] Can implement Parking Lot from scratch in 45 min with multi-floor + concurrency
- [ ] Can implement LRU Cache (HashMap + DLL, thread-safe) in 30 min
- [ ] Comfortable verbalizing class design before coding

### End of Week 6

- [ ] Cleared 1+ mock LLD interview with strong feedback
- [ ] Can pick correct patterns for an unseen problem in 5 min
- [ ] Always include concurrency discussion unprompted

### End of Week 8

- [ ] Ready for real LLD interviews — give first round
- [ ] At least 12 problems in `lld-prep` repo
- [ ] Can take any of the 8-week problems and re-do it from scratch

---

## Quick Reference: One-Page Cheat Sheet

```
ANSWER FRAMEWORK (45 min):
  1. Clarify     (4 min)  — functional, non-functional, scope
  2. Entities    (2 min)  — nouns
  3. Class diag  (8 min)  — interfaces, composition
  4. Patterns    (during 3) — justify each
  5. APIs        (5 min)  — signatures only
  6. Concurrency (5 min)  — where threads collide
  7. Code heart  (15 min) — 2-3 critical classes
  Buffer         (6 min)  — extensions, trade-offs

MUST-CODE PATTERNS (10):
  Tier 1: Singleton, Factory, Strategy, Observer, State
  Tier 2: Builder, Decorator, CoR, Command, Adapter

RECOGNIZE-ONLY (3):
  Composite, Template Method, Proxy

SKIP:
  Bridge, Flyweight, Interpreter, Mediator, Memento,
  Visitor, Prototype, Facade, Abstract Factory

GOLDEN SDE2 PHRASES:
  - "Prefer composition over inheritance"
  - "Single-threaded works, but for concurrent X we'd use Y"
  - "I'm using <pattern> here because <one sentence>"
  - "If we later need <feature>, we'd extend <class>"
  - "Out of scope for now, but worth flagging: <X>"
```

---

*Companion docs: `tech-refined.md` (overall SDE2 plan), `advice-refined.md` (job search execution).*
