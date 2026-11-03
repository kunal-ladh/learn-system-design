---
title: Stripe - Online Migrations at Scale
type: resource
resource_type: blog
tags: [resource, blog]
url: https://stripe.com/blog/online-migrations
author: "Jacqueline Xu (Stripe)"
year: "2017"
status: unread
priority: medium
related_concepts: ["[[Schema Evolution]]", "[[Versioning and Migrations]]", "[[Backward Compatibility]]"]
related_problems: []
summary: "The four-phase dual-write migration (dual write → switch reads → switch writes → remove old) that moved hundreds of millions of records with no downtime."
created: 2026-11-03
updated: 2026-11-03
---

# Stripe - Online Migrations at Scale

> [!abstract] Why this resource
> 'How do you change the schema of a live table?' is a common SDE-2 probe, and the dual-write pattern here is the standard answer. The Scientist-style read comparison is the detail that makes it safe.

**Link:** [https://stripe.com/blog/online-migrations](https://stripe.com/blog/online-migrations)  
**Type:** Engineering blog post · **Author:** Jacqueline Xu (Stripe) · **Year:** 2017

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free blog post. Link fetched and verified 2026-11-03.

## When to read it

Parts 1–4 of the walkthrough (fifteen minutes) with [[Versioning and Migrations]] in week 12 of [[Start Here]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Four phases: dual-write to old and new tables, move all reads to the new table, move all writes to the new table, remove the old data.
- The backfill ran offline with MapReduce over subscription IDs instead of querying the production database.
- GitHub's Scientist library compared old and new reads and alerted on any mismatch before reads were switched.
- Dual-write load was ramped by percentage while watching metrics; existing records were copied lazily on their next update.
- The old accessor was changed to raise a hard error to catch leftover uses; no change touched more than a few hundred lines at a time.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Backward Compatibility]]
- [[Designing Systems for Changing Requirements]]
- [[Event Schema Evolution]]
- [[Schema Evolution]]
- [[Strangler Pattern]]
- [[Versioning and Migrations]]

### Design problems

- *(none link here yet)*
