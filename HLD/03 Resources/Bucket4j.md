---
title: Bucket4j
type: resource
resource_type: open-source
tags: [resource, open-source]
url: https://github.com/bucket4j/bucket4j
author: "Vladimir Bukhtoyarov"
year: ""
status: unread
priority: medium
related_concepts: ["[[Rate Limiting]]", "[[Redis]]"]
related_problems: ["[[Design a Rate Limiter]]"]
summary: "A Java token-bucket rate-limiting library with lock-free local buckets and distributed back-ends on Redis, JCache, Hazelcast, JDBC and more."
created: 2026-11-03
updated: 2026-11-03
---

# Bucket4j

> [!abstract] Why this resource
> It is the reference implementation to compare [[Rate Limiter Library]] against, and its back-end list is a map of where distributed limiter state can live.

**Link:** [https://github.com/bucket4j/bucket4j](https://github.com/bucket4j/bucket4j)  
**Type:** Open-source implementation · **Author:** Vladimir Bukhtoyarov

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Public GitHub repository; reference docs at bucket4j.github.io. Link fetched and verified 2026-11-03.

## When to read it

The README quick start and the distributed back-ends section with [[Rate Limiting]]; benchmark your own token bucket against it in the library project.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Quick start: Bucket.builder().addLimit(capacity 20, greedy refill 10 per minute).build(), then tryConsume(1).
- Integer arithmetic only (no floats) to avoid rounding errors; default concurrency is lock-free.
- Distributed back-ends: JCache, Hazelcast, Ignite, Infinispan, Coherence, Couchbase, Redis (Lettuce, Jedis, Redisson), Valkey, MongoDB, Memcached, Cassandra, JDBC.
- A Caffeine back-end exists for sticky-routing cases where a local cache suffices.
- Apache-2.0; artifact com.bucket4j:bucket4j_jdk17-core.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Rate Limiting]]
- [[Redis]]

### Design problems

- [[Design a Rate Limiter]]
