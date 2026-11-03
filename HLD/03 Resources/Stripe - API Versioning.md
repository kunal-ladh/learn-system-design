---
title: Stripe - API Versioning
type: resource
resource_type: blog
tags: [resource, blog]
url: https://stripe.com/blog/api-versioning
author: "Brandur Leach (Stripe)"
year: "2017"
status: unread
priority: medium
related_concepts: ["[[Versioning and Migrations]]", "[[Backward Compatibility]]", "[[Designing APIs and Contracts]]"]
related_problems: []
summary: "Stripe's date-named rolling API versions, per-account pinning, and the 'version change modules' that transform current responses backwards to any older version."
created: 2026-11-03
updated: 2026-11-03
---

# Stripe - API Versioning

> [!abstract] Why this resource
> It is the most concrete design for backward-compatible APIs in public, and the answer to 'how would you evolve this API without breaking clients?' in [[Designing APIs and Contracts]].

**Link:** [https://stripe.com/blog/api-versioning](https://stripe.com/blog/api-versioning)  
**Type:** Engineering blog post · **Author:** Brandur Leach (Stripe) · **Year:** 2017

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free blog post. Link fetched and verified 2026-11-03.

## When to read it

The Versioning under the hood and Changes with side effects sections (fifteen minutes) with [[Backward Compatibility]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Versions are named by release date (e.g. 2017-05-24), each holding a small set of backwards-incompatible changes.
- An account is pinned to the newest version on its first request; a request can override with the Stripe-Version header; OAuth apps have their own version.
- Each incompatible change is a module with a description and a transformation; responses are built at the current version and transformed backwards to the target.
- Changes with side effects are flagged and checked explicitly, which the post calls harder to maintain and avoids where possible.
- Stripe kept compatibility with every version since 2011 across almost a hundred incompatible upgrades.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Backward Compatibility]]
- [[Designing APIs and Contracts]]
- [[Designing for Extensibility]]
- [[Event Schema Evolution]]
- [[Event Schemas]]
- [[REST]]
- [[Schema Evolution]]
- [[Versioning and Migrations]]

### Design problems

- *(none link here yet)*
