---
title: gRPC Core Concepts
type: resource
resource_type: documentation
tags: [resource, documentation]
url: https://grpc.io/docs/what-is-grpc/core-concepts/
author: "gRPC authors"
year: ""
status: unread
priority: medium
related_concepts: ["[[gRPC]]", "[[REST]]", "[[HTTP and HTTPS]]"]
related_problems: []
summary: "Service definitions, the four RPC kinds, the RPC lifecycle, deadlines, cancellation, metadata and channels — the facts you need to compare gRPC with REST honestly."
created: 2026-11-03
updated: 2026-11-03
---

# gRPC Core Concepts

> [!abstract] Why this resource
> 'REST or gRPC?' comes up in almost every API-design discussion. The deadline and cancellation semantics on this page are the real differentiators, and the line that cancellations do not roll back work is the one most people miss.

**Link:** [https://grpc.io/docs/what-is-grpc/core-concepts/](https://grpc.io/docs/what-is-grpc/core-concepts/)  
**Type:** Primary documentation · **Author:** gRPC authors

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Official docs page, free. Link fetched and verified 2026-11-03.

## When to read it

The RPC life cycle section (unary through channels) with [[gRPC]] and [[Designing APIs and Contracts]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Protocol buffers are the default IDL; the four method kinds are unary, server streaming, client streaming and bidirectional streaming.
- gRPC guarantees message ordering within an individual RPC; in bidirectional streams the two directions are independent.
- An RPC that runs past the client deadline ends with DEADLINE_EXCEEDED; the server can query the remaining time.
- Client and server decide success independently, so one side can see success while the other sees failure; changes made before a cancellation are not rolled back.
- Metadata keys are case-insensitive, must not start with 'grpc-', and binary values use keys ending in '-bin'.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Designing APIs and Contracts]]
- [[HTTP and HTTPS]]
- [[REST]]
- [[gRPC]]

### Design problems

- *(none link here yet)*
