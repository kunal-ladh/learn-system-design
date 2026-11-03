---
title: AWS Architecture Blog - Exponential Backoff and Jitter
type: resource
resource_type: blog
tags: [resource, blog]
url: https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/
author: "Marc Brooker (AWS)"
year: "2015"
status: unread
priority: medium
related_concepts: ["[[Timeouts and Retries]]", "[[Retry Strategies]]", "[[Retry Storms]]"]
related_problems: ["[[Design a Distributed Job Scheduler]]", "[[Design a Webhook Delivery System]]"]
summary: "A simulation of contending clients showing that exponential backoff alone leaves clusters of retries, and that full jitter halves the work and finishes sooner."
created: 2026-11-03
updated: 2026-11-03
---

# AWS Architecture Blog - Exponential Backoff and Jitter

> [!abstract] Why this resource
> It is the proof behind 'add jitter', with a simulator you can rerun. When an interviewer asks why jitter, the N² work argument here is the answer.

**Link:** [https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/](https://aws.amazon.com/blogs/architecture/exponential-backoff-and-jitter/)  
**Type:** Engineering blog post · **Author:** Marc Brooker (AWS) · **Year:** 2015

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free blog post; the sleep formulas are images, so note them down by hand. Link fetched and verified 2026-11-03.

## When to read it

Whole post (ten minutes) with [[Retry Strategies]]; try the simulator from the linked repo if you want the plots yourself.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- With N clients contending for one row, total client work grows with N² and completion time grows linearly.
- Capped exponential backoff alone reduces work only slightly — clusters of calls remain.
- With 100 contending clients, full jitter more than halved the call count and improved time to completion.
- Equal jitter does slightly more work and takes longer; decorrelated jitter makes more calls than full jitter.
- A 2023 update notes most AWS SDKs now implement exponential backoff with jitter in their standard and adaptive retry modes.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Client State and Reconnection]]
- [[Retry Storms]]
- [[Retry Strategies]]
- [[Timeouts and Retries]]

### Design problems

- [[Design a Distributed Job Scheduler]]
- [[Design a Webhook Delivery System]]
