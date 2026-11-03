---
title: Martin Fowler - CQRS
type: resource
resource_type: blog
tags: [resource, blog]
url: https://martinfowler.com/bliki/CQRS.html
author: "Martin Fowler"
year: "2011"
status: unread
priority: low
related_concepts: ["[[CQRS]]", "[[Event Sourcing]]"]
related_problems: []
summary: "The short definition of Command Query Responsibility Segregation and Fowler's warning to be very cautious about using it beyond a single bounded context."
created: 2026-11-03
updated: 2026-11-03
---

# Martin Fowler - CQRS

> [!abstract] Why this resource
> CQRS is over-applied in interview answers; this page is the authoritative caution, and the simpler reporting-database alternative it suggests is usually the better design. Read before claiming CQRS in [[Design a Social Feed]].

**Link:** [https://martinfowler.com/bliki/CQRS.html](https://martinfowler.com/bliki/CQRS.html)  
**Type:** Engineering blog post · **Author:** Martin Fowler · **Year:** 2011

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free bliki post. Link fetched and verified 2026-11-03.

## When to read it

Whole post (five minutes) with [[CQRS]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- CQRS splits the conceptual model into a command (update) model and a query (display) model, possibly in separate processes or databases.
- A separate query database becomes a real-time reporting database that needs a sync mechanism.
- It fits event-based programming, event sourcing and eventual consistency.
- Benefits: scale reads and writes separately and optimise each side differently.
- The warning: be very cautious, limit it to a bounded context, and prefer a reporting database for most cases.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[CQRS]]
- [[Event Sourcing]]

### Design problems

- *(none link here yet)*
