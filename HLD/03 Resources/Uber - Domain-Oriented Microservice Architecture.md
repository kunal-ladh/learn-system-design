---
title: Uber - Domain-Oriented Microservice Architecture
type: resource
resource_type: case-study
tags: [resource, case-study]
url: https://www.uber.com/blog/microservice-architecture/
author: "Adam Gluck (Uber)"
year: "2020"
status: unread
priority: medium
related_concepts: ["[[Microservices]]", "[[Domain-Driven Design]]", "[[API Gateways]]"]
related_problems: ["[[Design a Ride-Sharing Service]]", "[[Design an API Gateway]]"]
summary: "How Uber grouped about 2,200 microservices into 70 domains with layers, per-domain gateways and extension points to contain the complexity of too many services."
created: 2026-11-03
updated: 2026-11-03
---

# Uber - Domain-Oriented Microservice Architecture

> [!abstract] Why this resource
> It is the honest sequel to the microservices article: what happens after you have thousands of services, and why gateways and domain boundaries are the fix. Good background for [[Design a Ride-Sharing Service]] and for any 'how would you split this?' probe.

**Link:** [https://www.uber.com/blog/microservice-architecture/](https://www.uber.com/blog/microservice-architecture/)  
**Type:** Engineering case study · **Author:** Adam Gluck (Uber) · **Year:** 2020

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free engineering blog post. Link fetched and verified 2026-11-03.

## When to read it

The layers, gateways and extensions sections plus Practical Advice (twenty minutes) with [[Microservices]] and [[Domain-Driven Design]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- About 2,200 critical microservices were classified into 70 domains.
- Five layers — infrastructure, business, product, presentation, edge — and each layer may depend only on layers beneath it.
- Gateways give one entry point per domain, so product teams call one service and migrations behind it stay invisible.
- Logic extensions use an interface-driven plugin pattern (the 'go online' endpoint example).
- Reported results: onboarding time cut 25–50%; an estimated service half-life of 1.5 years is the argument for gateways.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[API Gateways]]
- [[Data Ownership and Lifecycle]]
- [[Domain-Driven Design]]
- [[Evaluating Architectural Complexity]]
- [[Microservices]]

### Design problems

- [[Design a Ride-Sharing Service]]
- [[Design an API Gateway]]
