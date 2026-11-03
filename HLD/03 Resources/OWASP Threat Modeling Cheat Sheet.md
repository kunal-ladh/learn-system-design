---
title: OWASP Threat Modeling Cheat Sheet
type: resource
resource_type: documentation
tags: [resource, documentation]
url: https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html
author: "OWASP Cheat Sheet Series"
year: ""
status: unread
priority: medium
related_concepts: ["[[Threat Modeling]]", "[[Input Validation]]", "[[Least Privilege]]"]
related_problems: []
summary: "A concise, actionable version of threat modelling: the four questions, data-flow diagrams with trust boundaries, STRIDE, and the mitigate/eliminate/transfer/accept responses."
created: 2026-11-03
updated: 2026-11-03
---

# OWASP Threat Modeling Cheat Sheet

> [!abstract] Why this resource
> Security questions in SDE-2 rounds are usually one probe ('what are the security concerns here?'); STRIDE gives you a structured answer in thirty seconds. The cheat sheet is the fastest way to internalise it.

**Link:** [https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html](https://cheatsheetseries.owasp.org/cheatsheets/Threat_Modeling_Cheat_Sheet.html)  
**Type:** Primary documentation · **Author:** OWASP Cheat Sheet Series

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free cheat sheet (CC BY-SA 4.0). Link fetched and verified 2026-11-03.

## When to read it

The Threat Identification (STRIDE table) and Response sections with [[Threat Modeling]]; then run STRIDE over one problem note's architecture.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Four questions: what are we working on, what can go wrong, what are we going to do about it, did we do a good enough job.
- STRIDE maps threats to violated properties: spoofing → authentication, tampering → integrity, elevation of privilege → authorization, and so on.
- Responses are Mitigate, Eliminate, Transfer, Accept — and transferring risk does not reduce likelihood or harm.
- Ranking by likelihood × impact is the ideal but ignores the effort to fix; the page says so.
- Data-flow diagrams should show trust boundaries, flows, stores, processes and external entities; the model is maintained with the system, not done once.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[API Keys]]
- [[Audit Logs]]
- [[Authentication]]
- [[Authorization]]
- [[Encryption at Rest and in Transit]]
- [[Input Validation]]
- [[Least Privilege]]
- [[Secrets Management]]
- [[Threat Modeling]]

### Design problems

- *(none link here yet)*
