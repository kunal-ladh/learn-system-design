---
title: OAuth 2.0 RFC 6749
type: resource
resource_type: documentation
tags: [resource, documentation]
url: https://datatracker.ietf.org/doc/html/rfc6749
author: "D. Hardt (ed.), IETF"
year: "2012"
status: unread
priority: medium
related_concepts: ["[[OAuth]]", "[[Authentication]]", "[[Authorization]]"]
related_problems: ["[[Design an IAM Service]]"]
summary: "The OAuth 2.0 authorization framework itself: the four roles, the grant types, access and refresh tokens, and the security considerations."
created: 2026-11-03
updated: 2026-11-03
---

# OAuth 2.0 RFC 6749

> [!abstract] Why this resource
> OAuth is explained badly almost everywhere; sections 1 and 4 of the RFC are clearer than most tutorials and are what [[Design an IAM Service]] needs. Knowing that refresh tokens never go to resource servers, or that the password grant is a last resort, is the difference between hand-waving and understanding.

**Link:** [https://datatracker.ietf.org/doc/html/rfc6749](https://datatracker.ietf.org/doc/html/rfc6749)  
**Type:** Primary documentation · **Author:** D. Hardt (ed.), IETF · **Year:** 2012

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> IETF datatracker HTML; updated by RFC 8252, RFC 8996 and RFC 9700. Link fetched and verified 2026-11-03.

## When to read it

Sections 1 (roles, protocol flow, grants, tokens) and 4.1 (authorization code grant) with [[OAuth]]; section 10 (security) with [[Threat Modeling]]. Note the page lists RFC 8252, 8996 and 9700 as updates.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Four roles: resource owner, resource server, client and authorization server (section 1.1).
- Four grant types: authorization code, implicit, resource-owner password credentials and client credentials (section 1.3).
- The password grant should only be used with a high degree of trust between resource owner and client.
- Refresh tokens are for the authorization server only and are never sent to resource servers (section 1.5).
- A client identifier must not be used alone for client authentication; authorization codes should expire quickly (10 minutes maximum recommended).

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[API Keys]]
- [[Authentication]]
- [[Authorization]]
- [[JWT]]
- [[OAuth]]

### Design problems

- [[Design an IAM Service]]
