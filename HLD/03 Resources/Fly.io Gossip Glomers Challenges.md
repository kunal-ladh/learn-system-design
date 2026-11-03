---
title: Fly.io Gossip Glomers Challenges
type: resource
resource_type: course
tags: [resource, course]
url: https://fly.io/dist-sys/
author: "Fly.io with Kyle Kingsbury"
year: ""
status: unread
priority: medium
related_concepts: ["[[Gossip Protocols]]", "[[Distributed ID Generation]]", "[[Consensus]]"]
related_problems: ["[[Design a Distributed ID Generator]]"]
summary: "Six hands-on distributed-systems challenges — echo, unique IDs, broadcast/gossip, a grow-only counter, a Kafka-style log and totally-available transactions — checked by Maelstrom."
created: 2026-11-03
updated: 2026-11-03
---

# Fly.io Gossip Glomers Challenges

> [!abstract] Why this resource
> They are the cheapest way to get real experience of partitions and duplicate messages: Maelstrom injects the failures and verifies your node. Challenge 2 is [[Design a Distributed ID Generator]] as a program; challenge 5 is a mini Kafka.

**Link:** [https://fly.io/dist-sys/](https://fly.io/dist-sys/)  
**Type:** Course / lecture series · **Author:** Fly.io with Kyle Kingsbury

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free challenge series; each challenge has its own page. Link fetched and verified 2026-11-03.

## When to read it

Challenges 1–3 over two weekends after week 10 of [[Start Here]]; use Go or Java (Maelstrom is language-agnostic).

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Built with Kyle Kingsbury (Jepsen) on top of Maelstrom, which routes messages between nodes, injects failures and verifies results.
- Docs are in Go, but Maelstrom is language-agnostic.
- Six challenges: Echo; Unique ID Generation; Broadcast; Grow-Only Counter; Kafka-Style Log; Totally-Available Transactions.
- Each node is a binary that reads JSON messages on stdin.
- Help is on community.fly.io under the dist-sys-challenge tag.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Consensus]]
- [[Distributed ID Generation]]
- [[Eventual Consistency]]
- [[Gossip Protocols]]

### Design problems

- [[Design a Distributed ID Generator]]
