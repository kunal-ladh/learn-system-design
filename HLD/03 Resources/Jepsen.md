---
title: Jepsen
type: resource
resource_type: open-source
tags: [resource, open-source]
url: https://jepsen.io/
author: "Kyle Kingsbury (Jepsen, LLC)"
year: ""
status: unread
priority: low
related_concepts: ["[[Consistency Models]]", "[[Network Partitions]]", "[[Isolation Levels]]"]
related_problems: []
summary: "The site that publishes safety analyses of real databases and queues under partitions and crashes, plus the open-source testing library behind them."
created: 2026-11-03
updated: 2026-11-03
---

# Jepsen

> [!abstract] Why this resource
> Each analysis is a case study in what [[Network Partitions]] do to a system that claims a consistency level. Read one for a database you use; it will change how you read vendor documentation.

**Link:** [https://jepsen.io/](https://jepsen.io/)  
**Type:** Open-source implementation · **Author:** Kyle Kingsbury (Jepsen, LLC)

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Landing page with news; analyses live under /analyses. Link fetched and verified 2026-11-03.

## When to read it

One analysis of a system you run (the /analyses index) after [[Consistency Models]]; the consistency reference is a separate note.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Recent analyses on the page report dozens of safety issues in current database releases and how vendors fixed them.
- Analyses are published free; the testing library is open source.
- The /consistency section is the reference map of models (see [[Jepsen Consistency Models]]).
- Talks such as 'Lessons From Testing Distributed Systems' are linked as slides.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Consensus]]
- [[Consistency Models]]
- [[Isolation Levels]]
- [[Network Partitions]]
- [[Split-Brain]]

### Design problems

- *(none link here yet)*
