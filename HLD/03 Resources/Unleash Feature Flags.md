---
title: Unleash Feature Flags
type: resource
resource_type: open-source
tags: [resource, open-source]
url: https://github.com/Unleash/unleash
author: "Unleash"
year: ""
status: unread
priority: low
related_concepts: ["[[Feature Flags]]", "[[Build vs Buy Decisions]]"]
related_problems: ["[[Design a Feature Flag Service]]"]
summary: "An open-source feature-management platform with activation strategies, gradual rollouts, kill switches and an edge component for scaling client SDKs."
created: 2026-11-03
updated: 2026-11-03
---

# Unleash Feature Flags

> [!abstract] Why this resource
> It is the 'buy' side of the build-vs-buy decision in [[Feature Flag Service and SDK]]: run it for an hour, then decide which parts your own service genuinely needs.

**Link:** [https://github.com/Unleash/unleash](https://github.com/Unleash/unleash)  
**Type:** Open-source implementation · **Author:** Unleash

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Public GitHub repository (AGPL-3.0); docs at docs.getunleash.io. Link fetched and verified 2026-11-03.

## When to read it

README quick start (docker compose up) and the SDK model, after [[Feature Flags]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Local setup is docker compose and localhost:4242 with a default admin account.
- Backend SDKs talk to /api/ with a token; frontend SDKs use /api/frontend/.
- Activation strategies enable features per user or segment without redeploying: canaries, gradual rollouts, kill switches, A/B tests.
- Unleash Edge scales frontend clients without loading the main server.
- AGPL-3.0; RBAC, SSO and extra environments are paid features.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Build vs Buy Decisions]]
- [[Feature Flags]]

### Design problems

- [[Design a Feature Flag Service]]
