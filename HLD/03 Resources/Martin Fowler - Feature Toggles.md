---
title: Martin Fowler - Feature Toggles
type: resource
resource_type: blog
tags: [resource, blog]
url: https://martinfowler.com/articles/feature-toggles.html
author: "Pete Hodgson"
year: "2017"
status: unread
priority: medium
related_concepts: ["[[Feature Flags]]", "[[Deployment and Rollback Strategies]]", "[[Incremental Delivery]]"]
related_problems: ["[[Design a Feature Flag Service]]"]
summary: "The taxonomy of toggles (release, experiment, ops, permissioning), how to keep toggle logic out of business code, and how to stop flags becoming permanent debt."
created: 2026-11-03
updated: 2026-11-03
---

# Martin Fowler - Feature Toggles

> [!abstract] Why this resource
> It is the design reference for [[Feature Flag Service and SDK]] and [[Design a Feature Flag Service]], and the carrying-cost section is the part most teams ignore.

**Link:** [https://martinfowler.com/articles/feature-toggles.html](https://martinfowler.com/articles/feature-toggles.html)  
**Type:** Engineering blog post · **Author:** Pete Hodgson · **Year:** 2017

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free article. Link fetched and verified 2026-11-03.

## When to read it

The categories and the decoupling sections (twenty minutes) with [[Feature Flags]] in week 12 of [[Start Here]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Toggles are categorised by longevity and dynamism into release, experiment, ops and permissioning toggles; release toggles should not live much beyond a week or two.
- Put toggle decisions behind a FeatureDecisions object so toggle points do not depend on the router or the flag name.
- Inversion of decision: inject the decision via configuration so modules are testable without flag infrastructure.
- For long-lived toggles, pass a strategy function instead of sprinkling if/else.
- Treat toggles as inventory with a carrying cost: removal tasks, expiry dates, tests that fail after a time-bomb date.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Build vs Buy Decisions]]
- [[Deployment and Rollback Strategies]]
- [[Feature Flags]]
- [[Incremental Delivery]]

### Design problems

- [[Design a Feature Flag Service]]
