---
title: etcd Raft Library
type: resource
resource_type: open-source
tags: [resource, open-source]
url: https://github.com/etcd-io/raft
author: "etcd project"
year: ""
status: unread
priority: low
related_concepts: ["[[Consensus]]", "[[Leader Election]]"]
related_problems: []
summary: "The Go Raft library behind etcd and Kubernetes — a deterministic state machine that leaves network and disk to the caller, which makes the algorithm readable in code."
created: 2026-11-03
updated: 2026-11-03
---

# etcd Raft Library

> [!abstract] Why this resource
> After [[Raft Paper]], this README shows what an implementation actually has to persist and in what order (entries and hard state before sending messages). Useful only if you go deep on [[Consensus]].

**Link:** [https://github.com/etcd-io/raft](https://github.com/etcd-io/raft)  
**Type:** Open-source implementation · **Author:** etcd project

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Public GitHub repository (Apache-2.0). Link fetched and verified 2026-11-03.

## When to read it

The README's usage and Ready-loop sections after [[Raft Paper]]; skim raft.go only if you are building the [[Lock Service with Fencing Tokens]] on real consensus.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- As of 2016 the most widely used Raft library in production, serving tens of thousands of clusters a day.
- Minimalist: only the algorithm; network and disk I/O are left to the user.
- A deterministic state machine whose output for a given input is messages, log entries and the next state.
- The Ready loop: persist entries, hard state and snapshot first, and send no messages until the latest hard state is on disk.
- Membership changes are disallowed while an uncommitted change is in the leader's log; three or more nodes are recommended; linearizable reads via read-index or leases.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Consensus]]
- [[Leader Election]]
- [[Service Discovery]]

### Design problems

- *(none link here yet)*
