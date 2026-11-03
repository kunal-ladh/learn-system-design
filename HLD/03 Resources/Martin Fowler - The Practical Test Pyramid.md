---
title: Martin Fowler - The Practical Test Pyramid
type: resource
resource_type: blog
tags: [resource, blog]
url: https://martinfowler.com/articles/practical-test-pyramid.html
author: "Ham Vocke (Thoughtworks)"
year: "2018"
status: unread
priority: medium
related_concepts: ["[[Testing Strategies]]", "[[Designing for Maintainability]]"]
related_problems: []
summary: "Unit, integration, contract, UI and end-to-end tests in proportion, with a Spring Boot sample service — the article behind [[Testing Strategies]]."
created: 2026-11-03
updated: 2026-11-03
---

# Martin Fowler - The Practical Test Pyramid

> [!abstract] Why this resource
> Every project note here has a testing-strategy section; this is where the shape of that section comes from, including narrow integration tests and consumer-driven contracts, which are the parts relevant to service design.

**Link:** [https://martinfowler.com/articles/practical-test-pyramid.html](https://martinfowler.com/articles/practical-test-pyramid.html)  
**Type:** Engineering blog post · **Author:** Ham Vocke (Thoughtworks) · **Year:** 2018

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free article. Link fetched and verified 2026-11-03.

## When to read it

Unit tests through contract tests (thirty minutes) with [[Testing Strategies]]; it uses Spring Boot, so the examples transfer directly.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Rule of thumb: many small fast unit tests, some coarser tests, very few end-to-end tests — and beware the ice-cream cone.
- Unit tests use arrange-act-assert, test the public interface and skip trivial getters and setters.
- Narrow integration tests exercise one integration point at a time with doubles for the rest; write one for every place data is serialised.
- Contract tests (Pact): a consumer generates a pact that the provider verifies against its API.
- End-to-end tests are slow and flaky, so keep a small number of high-value user journeys.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Designing for Maintainability]]
- [[Hexagonal Architecture]]
- [[Testing Strategies]]

### Design problems

- *(none link here yet)*
