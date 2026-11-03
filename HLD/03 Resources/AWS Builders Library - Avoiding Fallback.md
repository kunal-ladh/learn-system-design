---
title: AWS Builders Library - Avoiding Fallback
type: resource
resource_type: deep-dive
tags: [resource, deep-dive]
url: https://aws.amazon.com/builders-library/avoiding-fallback-in-distributed-systems/
author: "Jacob Gabrielson (AWS)"
year: "2019"
status: unread
priority: medium
related_concepts: ["[[Graceful Degradation]]", "[[Circuit Breakers]]", "[[Fault Tolerance]]"]
related_problems: ["[[Design a Real-Time Fraud Detection Pipeline]]"]
summary: "Why Amazon avoids fallback code paths — they are untested until the worst moment and have caused outages — and what to do instead: retries, proactive retry, failover, pushed data."
created: 2026-11-03
updated: 2026-11-03
---

# AWS Builders Library - Avoiding Fallback

> [!abstract] Why this resource
> It is the sharpest critique of the reflexive 'fall back to X' answer in design interviews. After reading it you will say 'convert the fallback into a continuously exercised failover' instead, which is a stronger answer.

**Link:** [https://aws.amazon.com/builders-library/avoiding-fallback-in-distributed-systems/](https://aws.amazon.com/builders-library/avoiding-fallback-in-distributed-systems/)  
**Type:** Architecture deep dive · **Author:** Jacob Gabrielson (AWS) · **Year:** 2019

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free article; aws.amazon.com redirects to builder.aws.com with the same text. Link fetched and verified 2026-11-03.

## When to read it

Whole article (fifteen minutes) with [[Graceful Degradation]] after week 9 of [[Start Here]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Four strategies for critical failures: retry, proactive retry, failover and fallback.
- The 2001-era retail outage: per-server caches of shipping speeds all failed at once, every server hit the supply-chain database directly, and the site and fulfilment centres went down.
- Alternatives: make the primary path more reliable, let the caller handle errors, push data proactively (IAM credentials pushed to EC2 instances, valid for hours).
- Convert fallback into failover by exercising the fallback path continuously — for instance randomly choosing between the two.
- Monitor retry rates and alarm when retries are frequent, so retries do not silently become fallback; proactive retry follows a 'constant work' design.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Circuit Breakers]]
- [[Fault Tolerance]]
- [[Graceful Degradation]]

### Design problems

- [[Design a Real-Time Fraud Detection Pipeline]]
