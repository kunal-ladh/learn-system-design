---
title: Cloudflare - Rate Limiting at Scale
type: resource
resource_type: case-study
tags: [resource, case-study]
url: https://blog.cloudflare.com/counting-things-a-lot-of-different-things/
author: "Julien Desgats (Cloudflare)"
year: "2017"
status: unread
priority: high
related_concepts: ["[[Rate Limiting]]", "[[Hot Keys]]", "[[Distributed Caching]]"]
related_problems: ["[[Design a Rate Limiter]]"]
summary: "How Cloudflare's edge rate limiter counts requests per data centre with a sliding-window approximation on memcache, trading a few percent of accuracy for no central store."
created: 2026-11-03
updated: 2026-11-03
---

# Cloudflare - Rate Limiting at Scale

> [!abstract] Why this resource
> It is the production answer to the hardest follow-up in [[Design a Rate Limiter]] — 'how do you count across a fleet without a single hot counter?' — including measured error rates you can quote.

**Link:** [https://blog.cloudflare.com/counting-things-a-lot-of-different-things/](https://blog.cloudflare.com/counting-things-a-lot-of-different-things/)  
**Type:** Engineering case study · **Author:** Julien Desgats (Cloudflare) · **Year:** 2017

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free engineering blog post. Link fetched and verified 2026-11-03.

## When to read it

Whole post (twenty minutes) in week 7 of [[Start Here]] alongside the rate-limiter problem.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Anycast keeps a client mostly on one data centre, so each PoP runs an isolated counting system instead of a global counter store.
- The memcache cluster offers only GET, SET and atomic INCR, which rules out multi-step algorithms like a true leaky bucket.
- Sliding-window approximation: 42 requests in the previous window plus 18 in the current, 15 seconds into the minute, gives about 49.5 against a limit of 50.
- Measured on 400 million requests from 270,000 sources: 0.003% wrongly allowed or limited, with an average 6% gap between real and approximated rate.
- Increments run off the request path and mitigation flags are cached in server memory, so most requests skip the memcache lookup during an attack.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Distributed Caching]]
- [[Hot Keys]]
- [[Rate Limiting]]

### Design problems

- [[Design a Rate Limiter]]
