---
title: Resilience4j
type: resource
resource_type: open-source
tags: [resource, open-source]
url: https://github.com/resilience4j/resilience4j
author: "Resilience4j project"
year: ""
status: unread
priority: medium
related_concepts: ["[[Circuit Breakers]]", "[[Bulkheads]]", "[[Rate Limiting]]"]
related_problems: []
summary: "The Java fault-tolerance library — circuit breaker, rate limiter, retry, bulkhead, time limiter — as decorators on functional interfaces; the successor to Hystrix."
created: 2026-11-03
updated: 2026-11-03
---

# Resilience4j

> [!abstract] Why this resource
> It is what you would actually use in a Spring Boot service to implement [[Circuit Breakers]] and [[Bulkheads]], and its best-practice section (one instance per protected backend) is a design rule worth knowing.

**Link:** [https://github.com/resilience4j/resilience4j](https://github.com/resilience4j/resilience4j)  
**Type:** Open-source implementation · **Author:** Resilience4j project

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Public GitHub repository (Apache-2.0); the user guide is at resilience4j.readme.io. Link fetched and verified 2026-11-03.

## When to read it

Overview and the Best Practices section after [[Circuit Breakers]]; wire it into [[Webhook Relay]] at M3.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Decorators: higher-order functions that enhance any functional interface, lambda or method reference.
- Retry.ofDefaults gives 3 attempts with a fixed 500 ms interval.
- Resilience4j 3 requires Java 21 and can run its schedulers on virtual threads.
- Create a unique instance per protected remote service; do not share CircuitBreaker, Bulkhead or RateLimiter across backends.
- Modules: circuitbreaker, ratelimiter, bulkhead, retry, timelimiter, cache, plus Spring Boot, Feign, Kotlin and reactive adapters.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Bulkheads]]
- [[Circuit Breakers]]
- [[Rate Limiting]]

### Design problems

- *(none link here yet)*
