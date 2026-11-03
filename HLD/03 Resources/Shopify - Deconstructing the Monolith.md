---
title: Shopify - Deconstructing the Monolith
type: resource
resource_type: case-study
tags: [resource, case-study]
url: https://shopify.engineering/deconstructing-monolith-designing-software-maximizes-developer-productivity
author: "Kirsten Westeinde (Shopify)"
year: "2019"
status: unread
priority: medium
related_concepts: ["[[Modular Monoliths]]", "[[Monoliths]]", "[[Separation of Concerns]]"]
related_problems: []
summary: "How Shopify turned a huge Rails monolith into a modular monolith — components by business domain, a dependency-boundary tool in CI, and explicit public APIs — instead of microservices."
created: 2026-11-03
updated: 2026-11-03
---

# Shopify - Deconstructing the Monolith

> [!abstract] Why this resource
> It is the evidence behind [[Modular Monoliths]]: a large company choosing boundaries without network calls, and the tooling needed to keep them. Use it when an interviewer assumes 'scale' means 'microservices'.

**Link:** [https://shopify.engineering/deconstructing-monolith-designing-software-maximizes-developer-productivity](https://shopify.engineering/deconstructing-monolith-designing-software-maximizes-developer-productivity)  
**Type:** Engineering case study · **Author:** Kirsten Westeinde (Shopify) · **Year:** 2019

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free engineering blog post. Link fetched and verified 2026-11-03.

## When to read it

Whole post (twenty minutes) with [[Modular Monoliths]] and [[Evaluating Architectural Complexity]] in week 12 of [[Start Here]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- The componentisation project started in 2017 after a developer survey of pain points.
- Code moved from a Rails MVC layout to real-world components (orders, shipping, inventory, billing) — about 6,000 classes labelled in a spreadsheet.
- The move landed as one big-bang PR of automated file moves; Git history was partly lost.
- The Wedge tool hooks Ruby tracepoints in CI to build a call graph: cross-component associations always violate the rules, and calls are allowed only to explicitly public APIs.
- Payoff: decoupled dependencies let the legacy tax engine be swapped for a new one; the post cites Fowler's Monolith First and Design Stamina Hypothesis.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Designing for Maintainability]]
- [[Evaluating Architectural Complexity]]
- [[Horizontal vs Vertical Scaling]]
- [[Modular Monoliths]]
- [[Monoliths]]
- [[Separation of Concerns]]
- [[Strangler Pattern]]
- [[Technical Debt]]

### Design problems

- *(none link here yet)*
