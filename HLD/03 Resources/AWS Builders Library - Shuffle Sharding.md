---
title: AWS Builders Library - Shuffle Sharding
type: resource
resource_type: deep-dive
tags: [resource, deep-dive]
url: https://aws.amazon.com/builders-library/workload-isolation-using-shuffle-sharding/
author: "Colm MacCarthaigh (AWS)"
year: "2019"
status: unread
priority: low
related_concepts: ["[[Bulkheads]]", "[[Fault Tolerance]]", "[[Hot Keys]]"]
related_problems: []
summary: "Assign each customer a near-unique combination of workers so a bad actor's blast radius shrinks combinatorially — the Route 53 origin story."
created: 2026-11-03
updated: 2026-11-03
---

# AWS Builders Library - Shuffle Sharding

> [!abstract] Why this resource
> It is the strongest version of [[Bulkheads]] and the standard answer to 'how do you stop one tenant hurting the others?'. The arithmetic (8 workers, 2 per customer, 28 combinations) is worth being able to reproduce.

**Link:** [https://aws.amazon.com/builders-library/workload-isolation-using-shuffle-sharding/](https://aws.amazon.com/builders-library/workload-isolation-using-shuffle-sharding/)  
**Type:** Architecture deep dive · **Author:** Colm MacCarthaigh (AWS) · **Year:** 2019

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free article; aws.amazon.com redirects to builder.aws.com with the same text. Link fetched and verified 2026-11-03.

## When to read it

Whole article (fifteen minutes) with [[Bulkheads]] after week 9 of [[Start Here]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Regular sharding into 4 shards of 2 workers limits impact to a quarter; shuffle sharding with 8 workers and 2 per customer gives 28 combinations, so impact is 1/28.
- The number of possible shuffle shards grows combinatorially, so with enough workers each customer is effectively isolated.
- Route 53 uses 2048 virtual name servers with four per customer domain — about 730 billion possible shuffle shards.
- No customer domain shares more than two virtual name servers with any other.
- Refinements: recursive shuffle sharding across layers and the open-source Route 53 Infima library; shuffle sharding usually costs nothing extra over plain sharding.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Bulkheads]]
- [[Fault Tolerance]]
- [[Hot Keys]]

### Design problems

- *(none link here yet)*
