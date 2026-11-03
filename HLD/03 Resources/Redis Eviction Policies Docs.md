---
title: Redis Eviction Policies Docs
type: resource
resource_type: documentation
tags: [resource, documentation]
url: https://redis.io/docs/latest/develop/reference/eviction/
author: "Redis documentation"
year: ""
status: unread
priority: high
related_concepts: ["[[Eviction Policies]]", "[[Redis]]", "[[TTL]]"]
related_problems: ["[[Design a Distributed Cache]]"]
summary: "How maxmemory and maxmemory-policy decide what Redis throws away: the allkeys/volatile LRU, LFU, random and TTL policies and the sampled approximation behind them."
created: 2026-11-03
updated: 2026-11-03
---

# Redis Eviction Policies Docs

> [!abstract] Why this resource
> 'Which eviction policy and why' is a standard follow-up in any caching design, and the honest answer depends on details on this page — such as volatile-* policies behaving like noeviction when nothing has a TTL. It is a ten-minute read that closes a common gap.

**Link:** [https://redis.io/docs/latest/develop/reference/eviction/](https://redis.io/docs/latest/develop/reference/eviction/)  
**Type:** Primary documentation · **Author:** Redis documentation

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Official docs page, free. Link fetched and verified 2026-11-03.

## When to read it

Whole page (short) with [[Eviction Policies]] in week 6 of [[Start Here]]. Then set a maxmemory on a local Redis and watch INFO while filling it.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- maxmemory defaults to 0 (no limit) on 64-bit systems; 32-bit systems have an implicit 3 GB limit.
- Policies: noeviction, allkeys-lru, allkeys-lfu, allkeys-random, volatile-lru, volatile-lfu, volatile-random, volatile-ttl (plus allkeys-lrm / volatile-lrm, added in Redis 8.6).
- volatile-* policies behave like noeviction when no keys have a TTL set — a frequent production surprise.
- LRU is approximated by sampling: maxmemory-samples defaults to 5; 10 samples gets very close to true LRU.
- LFU defaults: the counter saturates near one million requests and decays every minute (lfu-decay-time 1, lfu-log-factor 10); replication and AOF buffers are not counted toward maxmemory.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Distributed Caching]]
- [[Eviction Policies]]
- [[Redis]]
- [[TTL]]
- [[Write-Back Caching]]

### Design problems

- [[Design a Distributed Cache]]
