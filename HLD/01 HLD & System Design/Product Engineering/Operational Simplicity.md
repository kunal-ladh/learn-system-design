---
title: Operational Simplicity
type: concept
category: Product Engineering
tags: [concept, product-engineering]
aliases: [Boring technology, Operability]
priority: high
status: not-started
confidence: 1
date_started: 
last_reviewed: 
next_review: 
related: ["[[Evaluating Architectural Complexity]]", "[[Performance vs Cost Trade-offs]]", "[[Designing for Maintainability]]", "[[Build vs Buy Decisions]]", "[[Technical Debt]]", "[[Monitoring]]"]
summary: "Operational simplicity treats on-call load and the number of moving parts as design inputs: choose boring, well-understood technology, prefer managed services for anything that is not your core, and charge every new component its full operational cost before admitting it."
created: 2026-10-19
updated: 2026-10-19
---

# Operational Simplicity

> [!abstract] In one sentence
> Every component you add must be deployed, monitored, upgraded, secured, backed up and understood at 3 a.m., so operational simplicity — boring technology, fewer moving parts, managed services where they fit — is a feature you build deliberately, not a lack of ambition.

**Map:** [[Product Engineering MOC]] → [[HLD MOC]]

## Definition

**Operational simplicity** is the property of a system that is cheap and safe to run: few distinct technologies, each well understood by the team; few components, each with a clear job; known failure modes; routine upgrades, backups and scaling. It is the operations-side counterpart of [[Designing for Maintainability|maintainability]] and the outcome of a disciplined [[Evaluating Architectural Complexity|complexity evaluation]]. Dan McKinley's "choose boring technology" is the usual phrasing: each team has a few **innovation tokens**, and spending one on a new database or broker means not spending it on the product.

## Problem solved

Each additional technology brings its own failure modes, upgrade cadence, security advisories, monitoring gaps and tribal knowledge. A team of eight running PostgreSQL, Redis, Kafka, Elasticsearch and three languages is on-call for all of them while expert in none. Incidents take longer because the person paged does not know the component; upgrades are deferred because nobody dares; every design review adds "just one more" service. Operational simplicity makes the running cost visible when a component is proposed — the only moment it can still be refused.

## How it works

1. **Count the moving parts.** Every distinct technology, deployable, scheduled job and external dependency can page you. The list is the operational surface and should fit on one page.
2. **Charge each new component its full cost.** Before adding one, write down who upgrades it, how it is monitored, backed up and restored, what its failures do to the user, and who understands it at 3 a.m. If the answers are "we will figure it out", it is not earned yet.
3. **Prefer what the team already runs.** A second use of PostgreSQL (as a queue, a key-value store, with `pgvector`) is often cheaper to operate than the "right" specialised tool, up to a scale you should name. The known limits of a boring tool are an asset; the unknown limits of a new one are the risk.
4. **Buy the undifferentiated parts.** Managed databases, brokers and caches convert on-call load into a bill; for anything that is not your core, that is usually the right trade ([[Build vs Buy Decisions]]), with the exit path priced.
5. **Make on-call load a design metric.** A component that generates a disproportionate share of pages is a candidate for replacement with something simpler, or removal.
6. **Standardise the boring parts.** One way to deploy, emit logs and metrics, do health checks and config ([[The Twelve-Factor App]]); uniformity lets one on-call engineer cover many services.

```text
Component cost sheet (fill before adding a component)
Component:         Kafka (self-managed on EKS)
Replaces:          SQS + a Postgres outbox poller
Why not existing:  need replay and ordered partitions for the fraud stream
Run by:            platform team (2 people), both new to Kafka
Upgrades:          quarterly, rolling, requires broker restart drills
Monitoring:        consumer lag, under-replicated partitions, disk -- not yet dashboarded
Backup/restore:    none; data is replayable from the source topic? (unverified)
Failure impact:    fraud scoring stops; transfers fall back to rules-only
Exit path:         MSK or Redis Streams; consumers use a thin abstraction
Decision:          not yet -- use MSK (managed) and revisit self-hosting at 10x volume
```

## Architecture

No runtime diagram; the structural artifact is the component inventory with a cost sheet and an owner per entry. Architecturally the goal is a small set of well-known building blocks (one relational database, one cache, one queue, one deploy path) reused across services, with specialised components admitted only with a named justification and a named scale at which they become necessary.

## Use cases

- A design review proposing a new datastore, broker or framework "because it fits better": is the fit worth an innovation token?
- A small team with a large operational surface, where pages are spread across technologies nobody owns deeply.
- A new product line choosing its first stack: boring choices here compound for years.
- A self-host versus managed decision for a component that is not the product.

## Limitations

- "Boring" is relative to the team: Kafka experts should use Kafka; the principle is to minimise *unfamiliar* parts, not to avoid good tools.
- Managed services have real limits (version lag, missing features, cost at scale) and a lock-in that must be priced.
- Some products *are* their infrastructure (a search product, a streaming platform); there the specialised tool is the core and simplicity applies around it.

## Trade-offs

| You gain | You sacrifice |
|---|---|
| Fewer pages, faster recovery, upgrades that actually happen | The best-fit tool for each problem — the second-best tool you already run often wins |
| A team that deeply understands its stack | Novelty and the recruiting appeal of a modern stack |
| Predictable cost and capacity behaviour | Some performance or features at the extreme end |

## Failure modes

- **Component sprawl** → six datastores for six features → noticed when an upgrade advisory lands for a component nobody owns; blast radius is a security incident or a long outage on an orphaned system.
- **Managed service hits its limit silently** (connection caps, quotas) → the one component you did not monitor because "it is managed" → noticed as throttling under load.
- **Boring tool pushed past its known scale** (PostgreSQL as a queue at thousands of messages per second) → lock contention and bloat → the deferred specialised tool arrives under pressure.
- **Knowledge in one head** → the only person who understands the broker leaves → noticed at the next incident; blast radius is every service on that broker.

## Performance

Simplicity rarely costs performance at the scales most products reach: a well-tuned PostgreSQL, a Redis cache and a managed queue cover an enormous range. The cost appears at the extremes, so name the number — "PostgreSQL-as-queue is fine to about N jobs per second; beyond that we move to SQS or Redis Streams" — and the specialised tool arrives when earned.

## Scalability

Operational simplicity scales a team more than a system: uniform services let on-call cover more of them, and reused components let capacity knowledge transfer. At 10x load a simple system is also easier to scale, because its bottleneck is in a component the team understands.

## Operational impact

This note *is* operational impact: fewer runbooks, upgrade cycles and alert taxonomies, and a rota that can be staffed. The discipline costs a conversation at every design review and the occasional refusal of a better tool; the return is measured in pages per week and how calmly the team ships.

## Alternatives

- **Best tool for every job** (polyglot persistence) — wins for a large organisation with a platform team that runs each tool as a service; the cost is that team.
- **Serverless and fully managed everything** — wins for small teams with spiky load; costs predictability, debuggability and the bill at steady high load.
- **Single vendor platform** — wins on integration and operations; costs the exit path.

## Real-world examples

- Dan McKinley's "Choose Boring Technology" (Etsy) introduced the innovation-token framing; Etsy's stack discipline (PHP, MySQL, Memcached) was a deliberate operational choice.
- Rich Hickey's [[Simple Made Easy]] separates simple (not entangled) from easy (familiar): boring is about simple, not merely familiar.
- In a stack like yours: EKS, PostgreSQL on RDS, Redis on ElastiCache and a managed queue, where a broker history like RabbitMQ to Kafka to SQS, with Redis Streams under consideration, is the live example of each broker change having to pay its operational cost in dashboards, runbooks and expertise before it is justified.

## Practical exercise

1. List every technology and deployable your team is on-call for, with the owner, the last upgrade date and the pages in the last quarter.
2. Mark the components with no owner or no dashboard; those are your highest operational risk, not your newest ones.
3. Fill the cost sheet above for the next component someone proposes (or a planned broker move); decide with the sheet, not in the meeting.
4. Pick one component used by a single feature and estimate replacing it with something already in the stack; if it is a week, do it.

## Related concepts

### Prerequisites
- [[Evaluating Architectural Complexity]] — the moving-parts count this note acts on
- [[Performance vs Cost Trade-offs]] — the cost side includes the people who run it

### Related
- [[Modular Monoliths]] — one deployable as the simplest operational shape
- [[Technical Debt]] — an unowned component is operational debt
- [[Monitoring]] — every component needs it; fewer components, fewer gaps
- [[Prioritizing Reliability Performance Cost and Time to Market]] — simplicity serves cost and speed at once
- [[Capacity Planning]] — knowing the limits of boring tools is how you name the switch point

### Builds on this
- [[Designing for Maintainability]] — the code-level counterpart
- [[Build vs Buy Decisions]] — managed versus self-hosted is a simplicity question

### Design problems where it applies
- [[Design a Message Queue]]
- [[Design a Distributed Job Scheduler]]
- [[Design a Notification System]]

### Projects where it can be implemented
- [[Durable Job Scheduler]]
- [[Personal Notification Hub]]

### Resources
- [[Simple Made Easy]] — simple versus easy
- [[The Twelve-Factor App]] — uniform, operable services
- [[Fundamentals of Software Architecture]] — architecture characteristics including operability

## Personal understanding

> [!question] Explain it without the source
> Write your own explanation here, from memory. Then compare it with the sections above and note what you missed.

### Mastery checklist
- [ ] I can explain it without the original resource
- [ ] I can describe the problem it solves
- [ ] I can compare it with its alternatives
- [ ] I can name its limitations and failure modes
- [ ] I can recognise when it is appropriate in a real system
- [ ] I have applied it in a system design
- [ ] I have validated my assumptions with an experiment or implementation

## Open questions

- [ ] At what message rate and retention does PostgreSQL-as-a-queue stop being the boring choice for a team in a stack like yours, and what evidence would settle it?
- [ ] How do you price managed-service lock-in honestly — what is the cost of the exit path versus the on-call hours saved per year?
- [ ] Is "one language per stack" an operational simplicity rule worth enforcing, or does it cost more in hiring than it saves in operations?
