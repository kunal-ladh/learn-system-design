---
title: Build vs Buy Decisions
type: concept
category: Product Engineering
tags: [concept, product-engineering]
aliases: [Build vs buy]
priority: medium
status: not-started
confidence: 1
date_started: 
last_reviewed: 
next_review: 
related: ["[[Operational Simplicity]]", "[[Prioritizing Reliability Performance Cost and Time to Market]]", "[[Evaluating Architectural Complexity]]", "[[Technical Debt]]", "[[Designing Systems for Changing Requirements]]", "[[Performance vs Cost Trade-offs]]"]
summary: "Build what differentiates your product and buy or adopt everything else; decide with total cost of ownership over several years, the operational burden and the exit path in view, and settle genuine uncertainty with a time-boxed evaluation against your real workload rather than a feature matrix."
created: 2026-10-19
updated: 2026-10-19
---

# Build vs Buy Decisions

> [!abstract] In one sentence
> Build versus buy (or adopt open source) is the question of where your engineers' time creates the most value: build the core that makes the product different, buy or adopt the context that any competitor would have too, and decide the grey zone with total cost of ownership and an exit path rather than instinct.

**Map:** [[Product Engineering MOC]] → [[HLD MOC]]

## Definition

A **build-vs-buy decision** chooses, for a needed capability, between writing it yourself, paying a vendor for a product or managed service, or adopting open-source software and running it. It hinges on whether the capability is **core** (differentiating, where control pays) or **context** (necessary but undifferentiated, where someone else's economies of scale win), and on **total cost of ownership** (TCO): not the licence or the build estimate, but the multi-year sum of building, integrating, operating, upgrading, securing and eventually replacing. Adopting open source often has the lowest licence cost and the highest operational cost.

## Problem solved

Engineers default to building (it is interesting, and the estimate is optimistic); managers default to buying (the price is known, the integration cost is not). Both ignore the real drivers: operations over years, lock-in at the exit, and the opportunity cost of engineers not working on the product. The framework forces the question "what would we not build if we built this?".

## How it works

1. **Classify core versus context.** Would a customer choose you because of this capability? A ledger engine in a fintech is core; the flag service, the mail sender and the metrics backend are context. Core leans build; context leans buy or adopt. Most teams overestimate how much of their system is core.
2. **Estimate TCO over three to five years per option.** Build: design, implementation, tests, then on-call, dependency upgrades and feature requests for years. Buy: fees at projected scale, integration, the glue you still build, vendor-driven migrations. Adopt: integration plus the full operational burden ([[Operational Simplicity]]) and keeping up with upstream.
3. **Price the lock-in and the exit.** For each option, write what leaving in two years would take: data export, API surface used, time to replace. A component behind a thin internal interface is cheap to leave; one woven through every service is not. Lock-in is a cost, not a veto.
4. **Weigh the opportunity cost.** The build estimate is engineer-months not spent on core work; multiply by the realistic overrun and compare with the fees. At small scale buying usually wins.
5. **Run a time-boxed evaluation for the grey zone.** One or two weeks, two or three candidates, your real workload, a scorecard agreed in advance, and a decision at the end. A proof of concept with no deadline is a build project with no approval.
6. **Decide, record and set a revisit.** An ADR with the classification, the TCO table, the exit path and a revisit trigger (scale, cost threshold, vendor change).

```text
Capability: feature flag service           Classification: context
Option      3-yr TCO (eng-months + fees)   Ops burden   Exit path                 Notes
Build       6 build + 1/qtr maintain       ours         n/a (we own it)           no targeting UI, no audit at first
Buy (SaaS)  1 integrate + fees at scale    vendor       SDK behind our interface  fees rise with evaluations/month
Adopt OSS   2 integrate + 0.5/qtr ops      ours         same as buy               Unleash; we run Postgres already
Decision:   adopt behind an internal flag interface; revisit if fees-equivalent < ops cost or
            if targeting needs exceed what the OSS edition offers (ADR-021, revisit at 12 months)
```

## Architecture

No runtime shape; the structural artifact is the decision record with its TCO table, plus one architectural rule: put bought or adopted components behind a thin internal interface (an adapter in [[Hexagonal Architecture]] terms) so the exit stays cheap and an evaluation can swap candidates.

## Use cases

- A proposal to build an in-house version of something mature products exist for — flags, queues, auth, search — "because ours would be simpler".
- A vendor bill that has grown to the size of a team: the revisit trigger for an earlier buy.
- A new product line deciding what to build first and what to defer to a service ([[Prioritizing Reliability Performance Cost and Time to Market]]).
- An open-source component about to be self-hosted without the operations priced.

## Limitations

- TCO estimates are uncertain both ways: the build estimate is optimistic and the vendor's roadmap is a promise. Use ranges and revisit.
- "Core" shifts: a context capability becomes core when it becomes the product's bottleneck (search at a marketplace).
- Some buys are unavailable: regulation, data residency or the absence of a credible vendor force a build regardless.

## Trade-offs

| You gain | You sacrifice |
|---|---|
| Engineer time focused on what differentiates the product | Control — a bought capability's roadmap and limits are someone else's |
| Predictable cost and a mature product for context capabilities | Fit — a bought tool solves the general problem, not exactly yours |
| A recorded, revisitable decision instead of a default | Speed — the analysis takes days and an evaluation takes weeks |

## Failure modes

- **Building context** → a home-grown queue or auth service that is nobody's product → noticed when its one engineer leaves; blast radius is every dependant.
- **Buying core** → the differentiating capability is constrained by a vendor's roadmap → noticed when the product needs something the vendor will not build and the exit was never priced.
- **Evaluation without a deadline** → the proof of concept becomes the implementation by default → noticed when the "trial" has run in production for a year.
- **Lock-in unpriced** → the vendor raises prices or sunsets the product → a forced migration with no budget.

## Performance

A bought or adopted component brings documented limits; a built one brings whatever you measured. Evaluate against your traffic shape and tail latency, not the vendor's benchmark, and record the limit at which to revisit ("fine to 5k evaluations per second per node").

## Scalability

Buy decisions scale in cost (fees grow with usage); build decisions scale in operations (your team runs it at 10x). The crossover belongs in the ADR: at what volume do fees exceed running it yourself, people included? Most teams never reach it.

## Operational impact

Buying converts on-call hours into a bill and a dependency on someone else's incident response — read their SLA and status history. Adopting keeps the on-call hours and adds upstream tracking. Building keeps everything, including the security advisories you now issue yourself.

## Alternatives

- **Buy everything, build nothing** — wins for very early products validating a market; the glue becomes the real codebase.
- **Platform team builds shared services** — wins in large organisations where internal reuse amortises the build; the platform team is the vendor and needs the same SLA conversation.
- **Fork and own open source** — wins when upstream is stable and your changes are small; costs drift and a security-patch burden.

## Real-world examples

- Geoffrey Moore's core-versus-context distinction is the standard framing; Amazon's "undifferentiated heavy lifting" is the same idea from the vendor side.
- Fowler's toggle writing ([[Martin Fowler - Feature Toggles]]) and the existence of Unleash ([[Unleash Feature Flags]]) show how a capability teams once built became an adopt-or-buy default.
- In a stack like yours: an IAM service built in-house because identity, policies and audit are core to a fintech, while flags, metrics and the queue are adopted or managed — each behind an interface in the Spring Boot and NestJS services so a broker or flag-provider change is an adapter swap.

## Practical exercise

1. List the five largest non-product components your team runs or pays for; classify each as core or context and as built, bought or adopted.
2. For one mismatch (built context, or bought core), fill the TCO table above over three years, including on-call hours.
3. Write the exit path for the current choice in five lines; if you cannot, add the adapter interface that would make it possible.
4. Draft the ADR with a revisit trigger; the goal is a recorded decision, not necessarily a change.

## Related concepts

### Prerequisites
- [[Operational Simplicity]] — the operational side of TCO
- [[Prioritizing Reliability Performance Cost and Time to Market]] — the ranking that decides what deserves build time
- [[Performance vs Cost Trade-offs]] — the cost axis in detail

### Related
- [[Evaluating Architectural Complexity]] — each option adds a different kind of complexity
- [[Technical Debt]] — a built context component is often debt in disguise
- [[Capacity Planning]] — the volume at which fees and operations cross over
- [[Designing Systems for Changing Requirements]] — keeping the exit path cheap is keeping a decision reversible

### Builds on this
- [[Strangler Pattern]] — how a built or bought component is replaced later
- [[Feature Flags]] — the canonical capability that moved from build to adopt

### Design problems where it applies
- [[Design a Feature Flag Service]]
- [[Design an IAM Service]]
- [[Design a Metrics and Monitoring System]]

### Projects where it can be implemented
- [[Feature Flag Service and SDK]]
- [[LLM Gateway]]

### Resources
- [[Fundamentals of Software Architecture]] — architecture decisions and ADRs
- [[Unleash Feature Flags]] — an adopt candidate to evaluate against a build
- [[The Twelve-Factor App]] — backing services as attached, swappable resources

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

- [ ] How do you estimate the ongoing maintenance cost of a built component honestly, given that the initial estimate is always optimistic?
- [ ] Is an in-house IAM service core for a fintech, or is only the policy model core while token issuance is context?
- [ ] What scorecard would a two-week evaluation of Redis Streams versus SQS use for a workload in a stack like yours, and what result would end the trial early?
