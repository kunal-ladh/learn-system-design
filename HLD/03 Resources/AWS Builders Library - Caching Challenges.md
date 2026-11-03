---
title: AWS Builders Library - Caching Challenges
type: resource
resource_type: deep-dive
tags: [resource, deep-dive]
url: https://aws.amazon.com/builders-library/caching-challenges-and-strategies/
author: "Jas Chhabra and Matt Brinkley (AWS)"
year: "2019"
status: unread
priority: high
related_concepts: ["[[Cache-Aside]]", "[[Cache Stampedes]]", "[[Cache Consistency]]"]
related_problems: ["[[Design Search Autocomplete]]", "[[Design a Distributed Cache]]", "[[Design a URL Shortener]]"]
summary: "The risks of adding a cache and the practices that tame them: soft and hard TTLs, negative caching, request coalescing against thundering herds, and what to do when the cache itself fails."
created: 2026-11-03
updated: 2026-11-03
---

# AWS Builders Library - Caching Challenges

> [!abstract] Why this resource
> It covers the parts of caching that interviewers actually probe — stampedes, stale data, and the failure mode where the cache dies and the database gets the full load — with the patterns Amazon's IAM client uses. Read it with [[Cache Stampedes]] and keep the two-TTL idea for every design.

**Link:** [https://aws.amazon.com/builders-library/caching-challenges-and-strategies/](https://aws.amazon.com/builders-library/caching-challenges-and-strategies/)  
**Type:** Architecture deep dive · **Author:** Jas Chhabra and Matt Brinkley (AWS) · **Year:** 2019

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free article; aws.amazon.com redirects to builder.aws.com with the same text. Link fetched and verified 2026-11-03.

## When to read it

Whole article (twenty-five minutes) in week 6 of [[Start Here]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Two TTLs: refresh on a soft TTL, but keep serving until a hard TTL if the downstream service is unavailable (used in the IAM client).
- Negative caching: cache error responses with their own TTL, because not caching errors amplified failure rates.
- Thundering herd and cold start are mitigated with request coalescing — one in-flight fetch per uncached key.
- External caches (Memcached, Redis) avoid coherence problems and load proportional to fleet size, but add a fleet that can fail; in-memory caches have coherence and cold-start issues.
- When the cache fails, prefer an in-memory backstop or load shedding with a capped downstream rate over falling back to calling the dependency directly.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[CDNs]]
- [[Cache Consistency]]
- [[Cache Invalidation]]
- [[Cache Stampedes]]
- [[Cache-Aside]]
- [[Multi-Level Caching]]
- [[TTL]]
- [[Write-Through Caching]]

### Design problems

- [[Design Search Autocomplete]]
- [[Design a Distributed Cache]]
- [[Design a URL Shortener]]
