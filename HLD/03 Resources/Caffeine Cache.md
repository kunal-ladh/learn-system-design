---
title: Caffeine Cache
type: resource
resource_type: open-source
tags: [resource, open-source]
url: https://github.com/ben-manes/caffeine
author: "Ben Manes"
year: ""
status: unread
priority: medium
related_concepts: ["[[Eviction Policies]]", "[[Multi-Level Caching]]", "[[Cache-Aside]]"]
related_problems: []
summary: "The high-performance Java in-memory cache: TinyLFU admission, size and time-based eviction, asynchronous refresh — the local tier in any multi-level cache."
created: 2026-11-03
updated: 2026-11-03
---

# Caffeine Cache

> [!abstract] Why this resource
> It is what the L1 in [[Multi-Level Caching]] looks like in a Java service, and its refreshAfterWrite is a built-in answer to [[Cache Stampedes]] for hot keys. Compare its admission policy with Redis's sampled LRU to understand [[Eviction Policies]].

**Link:** [https://github.com/ben-manes/caffeine](https://github.com/ben-manes/caffeine)  
**Type:** Open-source implementation · **Author:** Ben Manes

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Public GitHub repository (Apache-2.0); the wiki holds the user guide and benchmarks. Link fetched and verified 2026-11-03.

## When to read it

README builder example and the wiki Efficiency page with [[Eviction Policies]]; add it as the local tier in [[LLM Gateway]] or [[Feature Flag Service and SDK]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Builder: maximumSize, expireAfterWrite, refreshAfterWrite, then build(loader).
- Size-based eviction uses frequency and recency via TinyLFU admission.
- Time-based expiration since last access or last write; asynchronous refresh on the first stale request.
- Weak keys, weak or soft values, removal listeners and statistics are supported; JCache and Guava adapters exist.
- Java 11+ needs 3.x; Apache-2.0.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Cache Stampedes]]
- [[Cache-Aside]]
- [[Eviction Policies]]
- [[Multi-Level Caching]]
- [[Write-Through Caching]]

### Design problems

- *(none link here yet)*
