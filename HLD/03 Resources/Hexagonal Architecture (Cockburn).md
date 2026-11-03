---
title: "Hexagonal Architecture (Cockburn)"
type: resource
resource_type: blog
tags: [resource, blog]
url: https://alistair.cockburn.us/hexagonal-architecture/
author: "Alistair Cockburn"
year: "2005"
status: unread
priority: low
related_concepts: ["[[Hexagonal Architecture]]", "[[Separation of Concerns]]", "[[Testing Strategies]]"]
related_problems: []
summary: "The original ports-and-adapters article: isolate the application from its drivers and its devices so it can be driven by tests, users or scripts alike."
created: 2026-11-03
updated: 2026-11-03
---

# Hexagonal Architecture (Cockburn)

> [!abstract] Why this resource
> It is the source for [[Hexagonal Architecture]] and the reason the project notes keep domain logic away from Spring and JDBC. Short and still the clearest statement of the idea.

**Link:** [https://alistair.cockburn.us/hexagonal-architecture/](https://alistair.cockburn.us/hexagonal-architecture/)  
**Type:** Engineering blog post · **Author:** Alistair Cockburn · **Year:** 2005

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free article (the 2005 original). Link fetched and verified 2026-11-03.

## When to read it

Intent, the inside/outside asymmetry and the worked example (twenty minutes) with [[Hexagonal Architecture]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Intent: let an application be driven equally by users, programs, tests or scripts, and be developed in isolation from its run-time devices and databases.
- The asymmetry to exploit is inside versus outside, not left versus right; inside code must not leak outward.
- Primary (driving) ports and adapters on one side, secondary (driven) on the other; fixtures drive the primary side, mocks replace the secondary side.
- Worked example: a discount calculation with a RateRepository port and a mock adapter.
- Use cases written at the application boundary stay shorter and more stable across technology changes.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Designing Systems for Changing Requirements]]
- [[Hexagonal Architecture]]
- [[Separation of Concerns]]
- [[Testing Strategies]]

### Design problems

- *(none link here yet)*
