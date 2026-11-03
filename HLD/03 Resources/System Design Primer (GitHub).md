---
title: "System Design Primer (GitHub)"
type: resource
resource_type: open-source
tags: [resource, open-source]
url: https://github.com/donnemartin/system-design-primer
author: "Donne Martin"
year: ""
status: unread
priority: high
related_concepts: ["[[Scalability]]", "[[Back-of-the-Envelope Estimation]]", "[[Load Balancers]]"]
related_problems: ["[[Design a Video Streaming Service]]"]
summary: "The most-starred system design study guide: topic summaries, an interview method, worked solutions for classic problems, and Anki decks."
created: 2026-11-03
updated: 2026-11-03
---

# System Design Primer (GitHub)

> [!abstract] Why this resource
> Its worked solutions are the closest thing to a public answer key for [[Design a URL Shortener]] and [[Design a Social Feed]], and its availability table (99.9% = 8h 46m a year) is worth memorising. Use it to check your own designs, not to replace them.

**Link:** [https://github.com/donnemartin/system-design-primer](https://github.com/donnemartin/system-design-primer)  
**Type:** Open-source implementation · **Author:** Donne Martin

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Public GitHub repository (README is the content). Link fetched and verified 2026-11-03.

## When to read it

The interview method and the availability/latency sections in week 1 of [[Start Here]]; one worked solution after each of your own designs. The Anki decks are optional.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Interview method: outline use cases and constraints, create a high-level design, design core components, then scale — and everything is a trade-off.
- Study guide by timeline: short for breadth, medium and long for depth.
- Worked solutions: Pastebin/Bit.ly, Twitter timeline and search, web crawler, Mint.com, social-network data structures, a key-value store for search, Amazon sales rank, scaling on AWS.
- Availability arithmetic: 99.9% allows about 8h 46m of downtime per year; 99.99% about 52m 36s.
- Cache-aside costs three trips on a miss; the OO design section is marked as under development.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Back-of-the-Envelope Estimation]]
- [[CDNs]]
- [[DNS]]
- [[Database Scaling]]
- [[HTTP and HTTPS]]
- [[Horizontal vs Vertical Scaling]]
- [[Keep-Alive Connections]]
- [[Load Balancers]]
- [[Object Storage]]
- [[Reverse Proxies]]
- [[Scalability]]
- [[TCP and UDP]]
- [[WebSockets]]

### Design problems

- [[Design a Video Streaming Service]]
