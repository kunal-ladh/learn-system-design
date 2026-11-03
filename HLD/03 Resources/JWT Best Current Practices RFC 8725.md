---
title: JWT Best Current Practices RFC 8725
type: resource
resource_type: documentation
tags: [resource, documentation]
url: https://datatracker.ietf.org/doc/html/rfc8725
author: "Y. Sheffer, D. Hardt, M. Jones, IETF (BCP 225)"
year: "2020"
status: unread
priority: medium
related_concepts: ["[[JWT]]", "[[Authentication]]"]
related_problems: ["[[Design an API Gateway]]", "[[Design an IAM Service]]"]
summary: "The IETF's list of JWT pitfalls and the rules that avoid them: pin algorithms, never accept 'none', validate audience, avoid passwords as HMAC keys, treat jku/x5u as SSRF risks."
created: 2026-11-03
updated: 2026-11-03
---

# JWT Best Current Practices RFC 8725

> [!abstract] Why this resource
> You use JWT at work, so you will be asked how it goes wrong. This BCP is the authoritative checklist, and its rules map one-to-one onto the failure modes in [[JWT]]. Twenty minutes, read once.

**Link:** [https://datatracker.ietf.org/doc/html/rfc8725](https://datatracker.ietf.org/doc/html/rfc8725)  
**Type:** Primary documentation · **Author:** Y. Sheffer, D. Hardt, M. Jones, IETF (BCP 225) · **Year:** 2020

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> IETF datatracker HTML (Best Current Practice 225). Link fetched and verified 2026-11-03.

## When to read it

Section 3 (best practices) with [[JWT]]; compare each rule against your IAM service's validation code.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Libraries must let the caller specify the allowed algorithms and must not use any others; each key is used with exactly one algorithm.
- Libraries should not generate JWTs with 'none' unless explicitly asked.
- Human-memorisable passwords must not be used directly as keys for keyed-MAC algorithms such as HS256.
- If one issuer issues tokens for several audiences, the JWT must contain aud and the relying party must validate it.
- Blindly following jku or x5u headers enables SSRF: match them against an allow-list; prefer explicit typing via the typ header for new uses.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Authentication]]
- [[Authorization]]
- [[JWT]]
- [[OAuth]]

### Design problems

- [[Design an API Gateway]]
- [[Design an IAM Service]]
