---
title: Knowledge Graph
aliases: [Concept chains, Learning graph]
type: moc
tags: [moc, graph]
summary: "The four concept chains that connect most of the curriculum — Kafka, database scaling, caching and real-time systems — drawn as navigable graphs."
created: 2026-10-30
updated: 2026-11-03
---

# Knowledge Graph — the chains that connect everything

> [!abstract] How to use this page
> System design concepts are not a list; they are a graph. Each chain below is a path you can walk note by note — every node in every diagram is a clickable link, and every concept note carries its own *Prerequisites → Related → Builds on this* section so you can keep walking. If you can draw one of these chains from memory and say why each arrow exists, you understand that area.

**Map:** [[HLD MOC]] · [[Start Here]] · [[Engineering Dashboard]]

## Chain 1 — Kafka: from partitions to safe consumers

```mermaid
flowchart TB
    K[Kafka] --> P[Topic Partitions]
    P -- "parallelism = partition count" --> CG[Consumer Groups]
    P -- "only within a partition" --> O[Ordering Guarantees]
    O --> D[Delivery Semantics]
    D -- "at-least-once in practice" --> I[Idempotency]
    I --> R[Retry Strategies]
    R -- "failure recovery" --> F[Dead-Letter Queues]
    class CG,D,F,I,K,O,P,R internal-link;
```

> [!example]- Excalidraw version — the same diagram as an editable canvas
> ![[Knowledge Graph — Chain 1 — Kafka- from partitions to safe consumers.excalidraw]]
> [[Knowledge Graph — Chain 1 — Kafka- from partitions to safe consumers.excalidraw|Open in Excalidraw]] — needs the Excalidraw plugin (see [[Plugins to Install]]).


[[Kafka]] → [[Topic Partitions]] → [[Consumer Groups]] (parallelism is capped by partition count) → [[Ordering Guarantees]] (only within a partition) → [[Delivery Semantics]] (at-least-once in practice) → [[Idempotency]] (the consumer owns correctness) → [[Retry Strategies]] → [[Dead-Letter Queues]] for failure recovery. Apply it in [[Design a Message Queue]] and [[Design a Notification System]]; build it in [[Streaming Fraud Rules Engine]].

## Chain 2 — Database scaling: from one box to many

```mermaid
flowchart TB
    S[Database Scaling] --> R[Replication]
    R -- "read scaling, replica lag" --> RR[Read Replicas]
    RR -- "what lag costs you" --> C[Consistency Models]
    C --> CAP[CAP Theorem]
    CAP --> PA[PACELC]
    S --> P[Partitioning]
    P --> SH[Sharding]
    SH -- "the hotspot problem" --> H[Hot Keys]
    SH --> CH[Consistent Hashing]
    class C,CAP,CH,H,P,PA,R,RR,S,SH internal-link;
```

> [!example]- Excalidraw version — the same diagram as an editable canvas
> ![[Knowledge Graph — Chain 2 — Database scaling- from one box to many.excalidraw]]
> [[Knowledge Graph — Chain 2 — Database scaling- from one box to many.excalidraw|Open in Excalidraw]] — needs the Excalidraw plugin (see [[Plugins to Install]]).


[[Database Scaling]] → [[Replication]] → [[Read Replicas]] (read scaling, replica lag) → [[Consistency Models]] (what lag costs you) → [[CAP Theorem]] (and [[PACELC]]) → [[Partitioning]] → [[Sharding]] → [[Hot Keys]] (the hotspot problem) → [[Consistent Hashing]]. Apply it in [[Design a URL Shortener]], [[Design a Social Feed]] and [[Design a Distributed Key-Value Store]]; build it in [[Mini-KV Store]].

## Chain 3 — Caching: from a quick win to a correctness problem

```mermaid
flowchart LR
    CA[Cache-Aside] --> INV[Cache Invalidation]
    INV --> CON[Cache Consistency]
    CA --> TTL[TTL]
    TTL --> EV[Eviction Policies]
    CA --> ST[Cache Stampedes]
    ST --> DC[Distributed Caching]
    DC --> HK[Hot Keys]
    DC --> ML[Multi-Level Caching]
    class CA,CON,DC,EV,HK,INV,ML,ST,TTL internal-link;
```

> [!example]- Excalidraw version — the same diagram as an editable canvas
> ![[Knowledge Graph — Chain 3 — Caching- from a quick win to a correctness problem.excalidraw]]
> [[Knowledge Graph — Chain 3 — Caching- from a quick win to a correctness problem.excalidraw|Open in Excalidraw]] — needs the Excalidraw plugin (see [[Plugins to Install]]).


[[Cache-Aside]] → [[Cache Invalidation]] → [[Cache Consistency]] (what a cache can and cannot promise) → [[TTL]] → [[Eviction Policies]] → [[Cache Stampedes]] → [[Distributed Caching]] → [[Hot Keys]] → [[Multi-Level Caching]]. Apply it in [[Design a Distributed Cache]] and [[Design a Rate Limiter]]; build it in [[Rate Limiter Library]] and [[LLM Gateway]].

## Chain 4 — Real-time systems: from a socket to a client that survives disconnects

```mermaid
flowchart TB
    RT[Real-Time Systems] --> WS[WebSockets]
    WS -- "event streaming" --> PS[Pub-Sub]
    PS --> FO[Fan-Out]
    FO --> BP[Backpressure]
    BP --> CS[Client State and Reconnection]
    CS --> ORD[Ordering Guarantees]
    ORD --> DUP[Duplicate Message Processing]
    DUP -- "replay from a sequence number" --> CS
    class BP,CS,DUP,FO,ORD,PS,RT,WS internal-link;
```

> [!example]- Excalidraw version — the same diagram as an editable canvas
> ![[Knowledge Graph — Chain 4 — Real-time systems- from a socket to a client that survives disconnects.excalidraw]]
> [[Knowledge Graph — Chain 4 — Real-time systems- from a socket to a client that survives disconnects.excalidraw|Open in Excalidraw]] — needs the Excalidraw plugin (see [[Plugins to Install]]).


[[Real-Time Systems]] is the hub: [[WebSockets]] → event streaming ([[Pub-Sub]], [[Kafka]]) → [[Fan-Out]] → [[Backpressure]] → [[Client State and Reconnection]] → [[Ordering Guarantees]] → [[Duplicate Message Processing]] → recovery after disconnect (replay from a sequence number). Apply it in [[Design a Chat System]] and [[Design a Metrics and Monitoring System]]; build it in [[Live Ops Dashboard]] and [[Presence and Chat Service]].

## Two chains you will meet at work

```mermaid
flowchart TB
    I[Idempotency] --> T[Transactions]
    T --> IL[Isolation Levels]
    IL --> L[Optimistic and Pessimistic Locking]
    L --> O[Outbox Pattern]
    O --> S[Saga Pattern]
    S --> A[Audit Logs]
    class A,I,IL,L,O,S,T internal-link;
```

> [!example]- Excalidraw version — the same diagram as an editable canvas
> ![[Knowledge Graph — Two chains you will meet at work.excalidraw]]
> [[Knowledge Graph — Two chains you will meet at work.excalidraw|Open in Excalidraw]] — needs the Excalidraw plugin (see [[Plugins to Install]]).


**Money moves safely:** [[Idempotency]] → [[Transactions]] → [[Isolation Levels]] → [[Optimistic and Pessimistic Locking]] → [[Outbox Pattern]] → [[Saga Pattern]] → [[Audit Logs]]. Apply in [[Design a Payment System]] and [[Design a Digital Wallet and Ledger]]; build in [[Double-Entry Ledger Service]].

```mermaid
flowchart TB
    T[Timeouts and Retries] --> RS[Retry Storms]
    RS --> CB[Circuit Breakers]
    CB --> B[Bulkheads]
    B --> RL[Rate Limiting]
    RL --> G[Graceful Degradation]
    G --> H[Health Checks]
    H --> A[Alerting]
    class A,B,CB,G,H,RS,T internal-link;
```

> [!example]- Excalidraw version — the same diagram as an editable canvas
> ![[Knowledge Graph — Two chains you will meet at work (2).excalidraw]]
> [[Knowledge Graph — Two chains you will meet at work (2).excalidraw|Open in Excalidraw]] — needs the Excalidraw plugin (see [[Plugins to Install]]).


**Nothing falls over at once:** [[Timeouts and Retries]] → [[Retry Storms]] → [[Circuit Breakers]] → [[Bulkheads]] → [[Rate Limiting]] → [[Graceful Degradation]] → [[Health Checks]] → [[Alerting]]. Apply in [[Design an API Gateway]]; build in [[Chaos Lab]].

## Navigating from a concept to a real system

1. Open the concept → *Related concepts* → *Design problems where it applies*.
2. Open the problem → its *Related* section names the concepts it needs and a project that implements it.
3. Open the project → *Relevant HLD concepts* closes the loop, and *Decisions* is where you record what you learned by building.

Obsidian's graph view, filtered with `tag:#concept` and coloured by folder (see [[Workspace Guide]]), shows the same structure as a picture.
