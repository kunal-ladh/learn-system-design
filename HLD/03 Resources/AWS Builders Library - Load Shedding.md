---
title: AWS Builders Library - Load Shedding
type: resource
resource_type: deep-dive
tags: [resource, deep-dive]
url: https://aws.amazon.com/builders-library/using-load-shedding-to-avoid-overload/
author: "David Yanacek (AWS)"
year: "2019"
status: unread
priority: medium
related_concepts: ["[[Graceful Degradation]]", "[[Backpressure]]", "[[Retry Storms]]"]
related_problems: ["[[Design a Ticket Booking System]]", "[[Design an API Gateway]]"]
summary: "Why overload makes goodput collapse (wasted work past client timeouts plus retries) and how rejecting excess work early keeps accepted requests fast."
created: 2026-11-03
updated: 2026-11-03
---

# AWS Builders Library - Load Shedding

> [!abstract] Why this resource
> 'What happens at 10× traffic?' is a standard probe; the goodput-vs-throughput distinction here is the right way to answer it, and the queue and deadline advice is immediately usable in [[Design a Ticket Booking System]].

**Link:** [https://aws.amazon.com/builders-library/using-load-shedding-to-avoid-overload/](https://aws.amazon.com/builders-library/using-load-shedding-to-avoid-overload/)  
**Type:** Architecture deep dive · **Author:** David Yanacek (AWS) · **Year:** 2019

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free article; aws.amazon.com redirects to builder.aws.com with the same text. Link fetched and verified 2026-11-03.

## When to read it

The sections Keeping an eye on the clock, Watching out for queues and Protecting in layers, with [[Graceful Degradation]] and [[Backpressure]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- The overload feedback loop: once latency exceeds the client timeout the server's work is wasted, and retries multiply offered load — exponentially in deep call graphs.
- Goodput is the part of throughput handled without errors and fast enough for the client to use.
- Give each request a time-to-live and propagate the remaining deadline across hops.
- Discard requests that waited too long in a queue, consider LIFO where the protocol allows, and prefer fast-fail spillover over surge queues.
- Load-balancer health pings are the highest-priority requests; shedding at the same CPU target as autoscaling can stop scale-out from ever triggering.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Backpressure]]
- [[Bottleneck Analysis]]
- [[Capacity Planning]]
- [[Graceful Degradation]]
- [[Performance vs Cost Trade-offs]]
- [[Retry Storms]]
- [[Throughput]]

### Design problems

- [[Design a Ticket Booking System]]
- [[Design an API Gateway]]
