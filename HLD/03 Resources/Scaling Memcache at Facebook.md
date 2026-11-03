---
title: Scaling Memcache at Facebook
type: resource
resource_type: paper
tags: [resource, paper]
url: https://www.usenix.org/system/files/conference/nsdi13/nsdi13-final170_update.pdf
author: "Rajesh Nishtala et al. (Facebook), NSDI 2013"
year: "2013"
status: unread
priority: medium
related_concepts: ["[[Cache Consistency]]", "[[Cache Stampedes]]", "[[Distributed Caching]]"]
related_problems: ["[[Design a Distributed Cache]]", "[[Design a URL Shortener]]"]
summary: "How Facebook ran memcached as a distributed cache across clusters and regions: leases against stampedes, UDP gets, the Gutter pool for failures, and commit-log-driven invalidation."
created: 2026-11-03
updated: 2026-11-03
---

# Scaling Memcache at Facebook

> [!abstract] Why this resource
> It is the production answer to three questions the Caching notes raise: how do you stop a thundering herd (leases), what happens when a cache node dies (Gutter), and how do you keep a cache consistent with the database (invalidation from the commit log). [[Design a Distributed Cache]] is much easier to defend after reading it.

**Link:** [https://www.usenix.org/system/files/conference/nsdi13/nsdi13-final170_update.pdf](https://www.usenix.org/system/files/conference/nsdi13/nsdi13-final170_update.pdf)  
**Type:** Paper · **Author:** Rajesh Nishtala et al. (Facebook), NSDI 2013 · **Year:** 2013

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> PDF (USENIX NSDI 2013). Link fetched and verified 2026-11-03.

## When to read it

Sections 3 (In a Cluster) and 4 (In a Region) after [[Cache Stampedes]] and [[Cache Invalidation]]; about ninety minutes.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Leases: memcached hands out a 64-bit token on a miss; the paper reports the database query rate for a hot key set dropping from 17K/s to 1.3K/s with leases.
- A token is returned only once every 10 seconds per key by default, which bounds thundering herds on hot keys.
- Gets use UDP for latency (about 20% lower) while sets and deletes use TCP via mcrouter; 0.25% of gets are dropped under peak load and treated as misses.
- Gutter is a small pool (about 1% of servers) that takes over for failed servers and cuts client-visible failures by 99%.
- Invalidation daemons (mcsqueal) batch deletes from SQL commit logs — an 18× improvement in median deletes per packet; cross-region consistency uses remote markers and is best-effort eventual.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Cache Consistency]]
- [[Cache Invalidation]]
- [[Cache Stampedes]]
- [[Cache-Aside]]
- [[Distributed Caching]]
- [[Eviction Policies]]
- [[Hot Keys]]
- [[Multi-Level Caching]]
- [[TTL]]
- [[Write-Through Caching]]

### Design problems

- [[Design a Distributed Cache]]
- [[Design a URL Shortener]]
