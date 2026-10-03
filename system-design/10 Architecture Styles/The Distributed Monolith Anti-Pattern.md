---
title: The Distributed Monolith Anti-Pattern
tags: [system-design, architecture]
type: note
status: evergreen
created: 2026-10-01
updated: 2026-10-02
summary: The failure mode where a system has microservices' network costs and a monolith's coupling — how teams get there, how to detect it, and why the fix is boundaries rather than more services.
related: ["[[Monolith vs Microservices]]", "[[Service Boundaries and Domain-Driven Design]]", "[[Strangler Fig Migration]]"]
---

# The Distributed Monolith Anti-Pattern

A **distributed monolith** is a system split into separately deployed services that nonetheless behave as one unit: they must be deployed together, they fail together through deep synchronous chains, and they often share a database. It is the worst quadrant of the architecture space — you pay microservices' full tax (network latency, partial failure, observability infrastructure, operational surface) while keeping the monolith's defining cost (everything coupled to everything). Understanding this anti-pattern matters because it is not rare: it is the *default outcome* of splitting a system without first learning to draw boundaries.

## Why this matters

The [[Monolith vs Microservices]] ledger only balances if the microservices side actually delivers its benefits — independent deployment, isolated failures, autonomous teams. The distributed monolith delivers none of them, so every line of its cost column is pure loss. Worse, it is self-concealing: the architecture *diagram* looks like microservices (many boxes, arrows, Kubernetes), so leadership believes the modernization succeeded while delivery keeps slowing. Teams in this state frequently conclude "microservices don't work" or "we need to split further", both wrong. Being able to name the pattern, detect it with concrete signals, and point at the real fix — boundaries — is a senior-engineer skill interviewers explicitly look for.

## How it works

### The failure mode, concretely

Four coupling mechanisms define it; a system showing two or more is in the quadrant:

1. **Lockstep deployment.** Service A's release requires B and C to release at the same time (shared release trains, "deployment coordination" meetings, version compatibility matrices). Independent deployability was the *definition* of microservices — without it, you have a monolith whose compiler checks were replaced by production incidents.
2. **Deep synchronous call chains.** User request → A → B → C → D, all blocking. Availability multiplies down the chain and the slowest hop sets the latency ([[Communication Patterns Between Services]] has the math: five 99.9% services ≈ 99.5% chain). One degraded service browns out the whole product — a monolith at least failed as one process, without four network hops of added latency on the way down.
3. **Shared database.** Several services read/write the same tables. Every schema change is a multi-team negotiation; services bypass each other's business logic by writing directly to the data; transactions quietly span "boundaries". The services are one application with extra steps — the database is the real (and unversioned) API.
4. **Breaking-change APIs.** Producers change contracts and consumers must upgrade in the same window ([[API Design and Versioning]] discipline absent). This is what *creates* lockstep deployment: a breaking contract is a deploy-time coupling.

```mermaid
graph LR
    U["User request"] --> A["Service A"]
    A -->|sync| B["Service B"]
    B -->|sync| C["Service C"]
    C -->|sync| D["Service D"]
    B --> DB["Shared database"]
    C --> DB
    D --> DB
```

### How teams get there

- **Splitting by technical layer.** An "API service", "business-logic service", and "data service" — every feature crosses all three, so all three deploy together forever. Same for entity-per-service decomposition (`UserService`, `OrderService`): business operations span entities, producing the chatty joins and shared-ish data. [[Service Boundaries and Domain-Driven Design]] explains why only capability-aligned cuts avoid this.
- **Premature decomposition.** Splitting before the domain is understood ([[Monolith vs Microservices|monolith-first]] argument): boundaries guessed on day one land mid-operation, and since moving a network boundary is expensive, teams compensate with more sync calls and shared data instead of redrawing — hardening the mistake.
- **Lift-and-shift extraction.** Carving code out of a monolith *without separating the data* ("we'll split the database later" — later never comes). The services remain welded through the schema.
- **In-process calls naively become HTTP.** The monolith's method calls get transliterated 1:1 into REST calls, preserving the old call graph — which was fine in-memory and is now a latency/availability liability. Nobody redesigned interactions for the network ([[The Fallacies of Distributed Computing]] assumed away).
- **Org pressure.** A platform mandate ("everything must be a microservice") or résumé-driven design creates services without the team structure (Conway) to own them independently.

### What this looks like at 3am

02:40. The page is from `statements`, returning 500s. Nobody on the statements team has deployed anything for nine days.

The on-call's first instinct — open the statements dashboards — produces nothing: CPU normal, heap normal, no GC pressure, its own database healthy, inbound traffic unchanged. The only anomaly is the error log, which is one line repeated thousands of times: a deserialization failure on a field read from a payload produced somewhere else.

The timeline that explains it: at 18:30 the previous evening, the `profile` service shipped a change renaming a response field. It passed `profile`'s tests, which were written against `profile`'s own schema. Nothing broke for eight hours because `statements` only reads that field during its 02:30 batch.

Three properties of that night are the anti-pattern's fingerprints, and they are worth memorising because you diagnose from the *shape* long before you find the cause:

- **The service that paged is not the service that changed**, and eight hours separated the two, which is far longer than anyone's instinct to correlate. Deploy annotations drawn on *every* service's dashboard — not just its own — are what close that gap, and they cost almost nothing to add.
- **No test could have caught it**, because the contract existed in no test. `profile`'s suite verified `profile`. `statements`' suite ran against a fixture copied from `profile` months earlier, which is a snapshot of a promise, not a check on it. A consumer-driven contract test is exactly the test neither suite contained.
- **The fix requires the other team, at 3am.** The statements on-call cannot roll forward — their code is correct against the contract they were given — and cannot roll back, since they changed nothing. They can wake the profile team or hand-patch a mapping. Either way, the independent on-call that the split was supposed to buy does not exist.

The sentence to carry away: in a healthy fleet, a page tells you which team *owns* the problem; in a distributed monolith, a page tells you which team *noticed* it. If incident reviews keep producing "root cause was a change in another service", that is not bad luck — it is the measurement.

### What it is *not* — avoiding false positives

The diagnosis is only useful if it discriminates. Four situations look like the pattern and aren't:

- **A shared read-only replica for reporting.** One writer still owns the schema. Readers are coupled to the schema's *shape*, which is real and worth fixing with a published read model, but it is nothing like two services writing the same tables. Multi-*writer* is the threshold in the table below, deliberately.
- **A one-off coordinated release.** Shipping four services together for a product launch is a project artifact. The signal is whether coordination is a *standing ritual* — a recurring calendar slot, a compatibility matrix — not whether it ever happens.
- **Depth that is infrastructure, not peers.** Gateway → service → cache → database is four components and one peer service. Chain depth counts *peer services in series*, because those are the ones with independent deploy cycles and independent failures.
- **A shared DTO or client library.** This is compile-time coupling, which is genuinely weaker: consumers upgrade on their own schedule as long as the library stays backward compatible. It crosses into the anti-pattern only when versions must move in lockstep — at which point the library is a deploy-time contract wearing a dependency's clothes.

### Detection

Every signal is measurable — don't diagnose by vibes. The checklist table below gives each signal a measurement and a worry threshold; the single fastest audit is the database-credential inventory (an afternoon of work that ends most arguments) followed by trace-depth queries on the top user paths ([[Distributed Tracing]]).

### The fix is boundaries, not more services

The instinctive responses — split the slow service further, add caching, add retries — treat symptoms. Each coupling has a structural fix:

- **Lockstep deploys / breaking APIs** → contract discipline: additive-only changes, tolerant readers, versioning with deprecation windows ([[API Design and Versioning]]); consumer-driven contract tests in CI so compatibility is checked before deploy, not during the outage.
- **Sync chains** → re-shape interactions: let services keep local, event-fed read models instead of querying each other live ([[Communication Patterns Between Services]], [[CQRS]]); make state propagation asynchronous via the [[The Outbox Pattern|outbox]]; where a chain is irreducible, protect it with [[Timeouts Retries and Exponential Backoff|timeouts]], [[Circuit Breakers]] and [[Bulkheads and Isolation|bulkheads]].
- **Shared database** → assign each table an owner; route other writers through the owner's API or events; split schemas, then stores ([[Strangler Fig Migration]]'s data strategies apply even without a monolith to strangle).
- **Wrong boundaries** → merge over-split services. **Consolidation is a legitimate fix**: two services that always change together are one service; re-merging them removes network hops, sagas, and coordination at zero benefit lost. The goal is right-sized boundaries, not maximal service count.

Sometimes the honest fix is admitting the org wasn't ready: fold the fragments back into a [[Modular Monolith]] with enforced internal boundaries, and re-extract later along seams proven by use.

### The escape sequence — what order to fix it in

The couplings interlock, so order matters; this sequence lets each step make the next one possible:

1. **Contract discipline first** (weeks): additive-only API changes, tolerant readers, consumer-driven contract tests in CI. Cheap, no data migration, and it's the *precondition* for everything else — you cannot decouple deploys while contracts break, and you cannot introduce event-fed read models while event schemas churn freely.
2. **Break the release train** (next): once contracts are stable-by-default, let the least-entangled service deploy alone, prove nothing breaks, and expand. Deployment independence is the first visible win and builds the political capital for the slower work.
3. **Flatten the worst sync chain** (quarters): pick the user path with the deepest trace, give the chain's readers event-fed local views ([[The Outbox Pattern|outbox]] → materialized views), and guard what remains with [[Circuit Breakers|breakers]] and [[Timeouts Retries and Exponential Backoff|budgeted timeouts]].
4. **Data divorce last** (the long pole): assign single writers to shared tables and migrate foreign writers through APIs/events. It's last because it's slowest and because steps 1–3 built the contract and event machinery it depends on.

Teams that start with the data divorce (the "proper" fix) usually stall: it's a year-long project with no visible wins, attempted while deploys still break weekly. Sequence for momentum, not purity.

## Detection checklist

| Signal | How to measure | Threshold that should worry you |
|---|---|---|
| Deploy coupling | Release calendar; change-failure correlation across services | Any standing "coordinated release" ritual |
| Sync chain depth | [[Distributed Tracing|Trace]] span depth on top user paths | p95 crossing 4+ services in series |
| Shared data | DB credential/connection inventory per schema or collection | More than one *writer* anywhere |
| Contract breakage | Consumer-driven contract tests; API-change postmortems | Consumers forced to upgrade on producer's schedule |
| Feature spread | Repos touched per feature ticket | Routine features spanning 3+ repos |
| Test coupling | Can one service's CI pass without booting the fleet? | Integration env must run everything |
| Shared-library lockstep | Version skew tolerated across services in production | Any service unable to run one minor version behind |
| Event schema coupling | Consumer breakages traced to producer payload changes | Any producer change needing consumer deploys in the same window |

## Trade-offs

Not applicable in the usual sense — an anti-pattern has no "when it wins". The table worth internalizing is the quadrant map:

| | Coupled | Decoupled |
|---|---|---|
| **One deployable** | Big ball of mud | [[Modular Monolith]] |
| **Many deployables** | **Distributed monolith** — worst of both | Microservices done right |

The movement that matters is *rightward* (fixing coupling), not *downward* (adding deployables). Splitting a coupled system only relocates it within the left column — from mud to distributed mud — at higher cost.

## Common misconceptions

- **"We have Kubernetes and 30 repos, so we have microservices."** Deployment topology doesn't define the architecture; *independent deployability* does. Thirty lockstep repos are a monolith with network latency.
- **"The fix is splitting the big services."** More cuts through wrong boundaries create more coupling surface. Fix the boundaries first; count services later.
- **"Shared databases are fine temporarily."** The shared DB is the strongest coupling in the system and the hardest to remove later; "temporary" shared schemas are how lift-and-shift extractions fossilize.
- **"Async everywhere fixes it."** Async with shared data and breaking schemas is still coupled — events have contracts too ([[Event Schema Evolution]]). Async is one tool; ownership boundaries are the cure.
- **"It's a technology problem."** Every structural fix in this note — contract discipline, single-writer data ownership, merging services that co-change — is an organisational decision about who owns what, implemented in code. Teams that shop for a tool instead (a schema registry, a tracing vendor, a mesh) buy excellent visibility into the coupling and remove none of it. The tools help you *see* it; only ownership decisions remove it, which is why this work needs a sponsor and not just a backlog.

## In your stack

- The afternoon audit for Niyo: list Mongo databases/collections by *writing* service — any collection with two writers is a coupling to schedule work against; count Kong-internal sync hops on the top user paths via correlation-ID traces; check whether any releases are habitually coordinated across services.
- The 1-replica fleet raises the stakes on sync chains specifically: each hop is a single point of failure, so a 4-service synchronous path has four SPOFs in series. Every chain link converted to an event via [[The Outbox Pattern|outbox]] + SQS/Redis Streams removes one. The arithmetic makes the case better than the adjective does: at 99.9% per hop, `0.999⁴ ≈ 0.99601` is about **173 minutes of monthly downtime against 43 for a single service**, and because each hop contributes roughly its own 0.1%, **every hop you take off the blocking path returns about 43 minutes a month**. That is a number you can put in a ticket, and it is the lever [[Monolith vs Microservices]] prices out in full — the one you pull once, rather than a fleet-wide hardening programme you must then maintain forever.
- Fintech pressure cuts both ways: regulators want clear ownership and auditability (pushes toward real boundaries, e.g. a single owner for ledger data), while "correctness beats latency" tempts teams into synchronous confirmation chains — prefer orchestrated [[Saga Pattern|sagas]] with persisted state over deep blocking call chains for money flows.
- Watch the migration surface: during the SQS → Redis Streams move, if multiple services share stream names and payload schemas informally, a producer-side schema change becomes a fleet-wide lockstep event — the async flavor of this anti-pattern. A schema registry habit ([[Event Schema Evolution]]) is the vaccine.

## Interview angle

The 60-second explanation: *"A distributed monolith is services in topology but a monolith in behavior: lockstep deploys, deep synchronous call chains, shared databases, breaking-change APIs. You pay network latency, partial failure, and platform cost, and get none of microservices' independence. Teams land there by splitting along technical layers or entities, decomposing before understanding the domain, or extracting code while leaving data shared. Detection is measurable: coordinated release calendars, cross-service change-failure correlation, trace depth, multi-writer tables. And the fix is boundaries, not more services — contract discipline to break deploy coupling, event-fed read models to flatten chains, single-writer data ownership, and merging services that always change together. Sometimes the right move is folding back into a modular monolith and re-extracting along proven seams."*

Classic follow-ups: "How would you *detect* one with data?" (traces, DB credential inventory, deploy correlation); "Is merging services ever right?" (yes — services that co-change are one service); "How did they get there?" (layer/entity splits, premature decomposition — shows you understand cause, not just symptom).

## Check yourself

> [!question]- Q: Explain why a distributed monolith is strictly worse than the monolith it replaced.
> The monolith had coupling but compensations: in-memory calls (no network latency or partial failure), one ACID transaction scope, one deploy artifact, one log to read. The distributed monolith keeps the coupling — lockstep releases, shared data, everything-depends-on-everything — but replaces function calls with network hops that add latency and new failure modes, splits transactions into consistency problems, and multiplies operational surface (N pipelines, N on-calls, tracing infrastructure). Same disease, more expensive symptoms: all of the microservices tax buys zero independence.

> [!question]- Q: Scenario — every release at your company is a coordinated "release train" where 6 services deploy together, and QA signs off on the combination. Name the root cause you'd look for first and the first two interventions.
> Root cause candidates: breaking-change API habits (contract changes forcing consumer upgrades in the same window) and/or a shared database making schema changes multi-service events — find which by asking "what breaks if service 3 ships alone?". First interventions: (1) contract discipline — additive-only changes, tolerant readers, and consumer-driven contract tests in CI so cross-version compatibility is verified continuously, letting services decouple their release cadence; (2) single-writer data ownership — inventory shared tables/collections and route foreign writers through the owner's API or events. Only after those does "deploy independently" become mechanically possible.

> [!question]- Q: What are three measurable signals (not vibes) that a fleet is a distributed monolith?
> (1) Synchronous trace depth on user paths — p95 requests crossing 4+ services in series; (2) multi-writer datastores — more than one service holding write credentials to the same schema/collections; (3) cross-service deploy coupling — release calendars/compatibility matrices, or change-failure rates in service B spiking when service A deploys. (Also strong: feature PRs habitually spanning 3+ repos; integration environments that must boot the whole fleet to test one service.)

> [!question]- Q: Scenario — a VP points at the architecture diagram: 22 services, Kubernetes, separate repos, separate pipelines. Delivery has slowed every quarter for a year. You have one week to produce evidence. What do you measure, and what does each result mean?
> Four measurements, cheapest first. (1) **Datastore writer inventory** — one afternoon, list every Mongo collection or SQL schema by the services holding *write* credentials. Any collection with two writers is structural coupling; zero multi-writer collections largely rules out the strongest form and redirects you elsewhere. (2) **Peer-service chain depth** — query traces for the top five user paths and record p95 span depth counting peer services only, not caches and databases. Four or more in series means availability and latency are being paid on every request ([[Communication Patterns Between Services]]). (3) **Deploy coupling** — look for a standing coordinated-release ritual, and correlate change-failure rate in each service against *other* services' deploy times; a spike in B after A deploys is deploy coupling even without a formal release train. (4) **Repos touched per feature ticket** — pull the last 50 tickets; a median of three or more repos per routine feature means the boundaries do not match the work. Then report honestly about what the results *rule out* too: if writers are single, chains are shallow, and deploys are independent, the slowdown is not this pattern and the diagnosis belongs somewhere else — test flakiness, review latency, or simply too few people. Evidence that disconfirms is what makes the week credible.

> [!question]- Q: Mini-design — you inherit `user-api`, `user-core`, and `user-db-service`, which call each other in series on every request and deploy together. Propose the fix.
> This is a layer-per-service split — three technical tiers of one capability. Don't "fix" it with caching between tiers: merge all three into one `users` service owning its own store (consolidation is the correct direction; the layers become internal packages of a [[Modular Monolith|modular]] service). Expose one capability API with additive-change discipline, publish `UserUpdated` events via the outbox for consumers who were scraping the DB, and retire the shared schema access. Result: one deploy, zero internal network hops, one owner — and a real boundary where the business has one, around the user capability.

## Sources

- Sam Newman, *Building Microservices*, 2nd ed. (O'Reilly, 2021) — independent deployability as the acid test; shared-database coupling.
- Sam Newman, *Monolith to Microservices* (O'Reilly, 2019) — decomposition mistakes and data-ownership patterns.
- Chris Richardson, microservices.io — "Microservice architecture" pattern prerequisites, and his talks on "microservices as a goal" anti-patterns.
- Martin Fowler, "MicroservicePremium" and "MonolithFirst" (martinfowler.com, 2015) — why premature decomposition produces exactly this.
- Stefan Tilkov, "Don't start with a monolith" (martinfowler.com guest post, 2015) — the counterpoint, useful for the "when boundaries are known" nuance.
