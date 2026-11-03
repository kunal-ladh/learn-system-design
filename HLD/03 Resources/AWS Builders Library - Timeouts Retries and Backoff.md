---
title: AWS Builders Library - Timeouts Retries and Backoff
type: resource
resource_type: deep-dive
tags: [resource, deep-dive]
url: https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/
author: "Marc Brooker (AWS)"
year: "2019"
status: unread
priority: high
related_concepts: ["[[Timeouts and Retries]]", "[[Retry Strategies]]", "[[Retry Storms]]"]
related_problems: ["[[Design a Notification System]]", "[[Design a RAG Knowledge Assistant]]"]
summary: "How Amazon chooses timeouts from latency percentiles, why layered retries multiply load (243× in the example), and why retries are budgeted with a token bucket and jittered."
created: 2026-11-03
updated: 2026-11-03
---

# AWS Builders Library - Timeouts Retries and Backoff

> [!abstract] Why this resource
> This is the article behind three of the most-asked follow-ups: how do you pick a timeout, what happens when every layer retries, and what makes a retry safe. Its numbers are the ones to quote.

**Link:** [https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/](https://aws.amazon.com/builders-library/timeouts-retries-and-backoff-with-jitter/)  
**Type:** Architecture deep dive · **Author:** Marc Brooker (AWS) · **Year:** 2019

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free article; aws.amazon.com now redirects to builder.aws.com, which serves the same text. Link fetched and verified 2026-11-03.

## When to read it

Whole article (twenty minutes) in week 3 of [[Start Here]] with [[Timeouts and Retries]]; reread before any mock interview.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Choose a timeout from downstream latency percentiles and an acceptable false-timeout rate (such as 0.1%) — the p99.9 in the example.
- A 20 ms timeout failed after deploys because the timer included TLS setup; the fix was pre-establishing connections at startup.
- A five-deep call stack with three retries per layer multiplies database load by 243×, so retry at a single point in the stack.
- Retries are limited with a token bucket; the AWS SDK gained this behaviour in 2016.
- APIs with side effects are not safe to retry unless idempotent (EC2 RunInstances uses client tokens); jitter for scheduled work should be a consistent per-host value, not random.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Retry Storms]]
- [[Retry Strategies]]
- [[Timeouts and Retries]]

### Design problems

- [[Design a Notification System]]
- [[Design a RAG Knowledge Assistant]]
