---
title: Martin Fowler - Strangler Fig Application
type: resource
resource_type: blog
tags: [resource, blog]
url: https://martinfowler.com/bliki/StranglerFigApplication.html
author: "Martin Fowler"
year: "2024"
status: unread
priority: low
related_concepts: ["[[Strangler Pattern]]", "[[Designing Systems for Changing Requirements]]"]
related_problems: []
summary: "Grow new functionality around a legacy system and replace it piece by piece, using seams and transitional architecture instead of a big-bang rewrite."
created: 2026-11-03
updated: 2026-11-03
---

# Martin Fowler - Strangler Fig Application

> [!abstract] Why this resource
> It is the one-page argument for incremental replacement that [[Strangler Pattern]] relies on, and the vocabulary (seams, transitional architecture) is worth having when a probe asks how you would migrate a monolith.

**Link:** [https://martinfowler.com/bliki/StranglerFigApplication.html](https://martinfowler.com/bliki/StranglerFigApplication.html)  
**Type:** Engineering blog post · **Author:** Martin Fowler · **Year:** 2024

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free bliki post (updated 2024). Link fetched and verified 2026-11-03.

## When to read it

Whole post (five minutes) with [[Strangler Pattern]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Simple replacement is said to fail most of the time: it takes too long, users cannot wait, and much existing behaviour is unwanted.
- Four activities: understand the outcomes, decide how to break the problem up, deliver the parts, change the organisation.
- Seams are the points where the legacy system can be split so components can be replaced independently.
- Transitional architecture is temporary code that lets new and old coexist; its cost is outweighed by reduced risk and earlier value.
- Organisational change is needed too (Conway's law).

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Designing Systems for Changing Requirements]]
- [[Incremental Delivery]]
- [[Strangler Pattern]]

### Design problems

- *(none link here yet)*
