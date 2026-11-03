---
title: Toxiproxy
type: resource
resource_type: open-source
tags: [resource, open-source]
url: https://github.com/Shopify/toxiproxy
author: "Shopify"
year: ""
status: unread
priority: medium
related_concepts: ["[[Network Partitions]]", "[[Timeouts and Retries]]", "[[Circuit Breakers]]"]
related_problems: []
summary: "A TCP proxy with an HTTP API that injects latency, bandwidth limits, packet loss, slow closes and resets between your service and its dependencies — the engine of [[Chaos Lab]]."
created: 2026-11-03
updated: 2026-11-03
---

# Toxiproxy

> [!abstract] Why this resource
> Every failure-mode section in this vault can be reproduced on a laptop with Toxiproxy. It is how [[Timeouts and Retries]], [[Circuit Breakers]] and [[Network Partitions]] move from confidence 3 to 4.

**Link:** [https://github.com/Shopify/toxiproxy](https://github.com/Shopify/toxiproxy)  
**Type:** Open-source implementation · **Author:** Shopify

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Public GitHub repository (MIT). Link fetched and verified 2026-11-03.

## When to read it

The README's toxics list and HTTP API (fifteen minutes); then follow the [[Chaos Lab]] milestones.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- latency toxic adds latency ± jitter in milliseconds; packet_loss takes loss_rate (0–1) and correlation for bursty loss.
- Other toxics: bandwidth, slow_close, timeout, reset_peer, slicer, limit_data; toxicity defaults to 1.0 (all connections), stream selects upstream or downstream.
- Taking a dependency down is POST enabled:false on /proxies/{proxy}, not a toxic.
- The HTTP API listens on 8474 with /populate, /reset, /version and Prometheus /metrics.
- Claimed overhead under 100 µs with no toxics; 2.x is not API-compatible with 1.x; MIT licence.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[API Gateways]]
- [[Alerting]]
- [[Bulkheads]]
- [[Circuit Breakers]]
- [[Consensus]]
- [[Coordination vs Availability]]
- [[Disaster Recovery]]
- [[Distributed Tracing]]
- [[Error Budgets]]
- [[Eventual Consistency]]
- [[Failure Detection]]
- [[Gossip Protocols]]
- [[Graceful Degradation]]
- [[Microservices]]
- [[Monitoring]]
- [[Network Partitions]]
- [[Quorums]]
- [[Recovery Point Objective]]
- [[Retry Storms]]
- [[Split-Brain]]
- [[TCP and UDP]]
- [[Testing Strategies]]
- [[Timeouts and Retries]]

### Design problems

- *(none link here yet)*
