---
title: Resources MOC
aliases: [Resource Library, Resources]
type: moc
tags: [moc, resource]
summary: "Curated library of primary documentation, papers, engineering write-ups, books, talks, courses and open-source implementations, each linked to the concepts and problems it explains."
created: 2026-10-30
updated: 2026-10-30
---

# Resource Library

> [!abstract] How to use it
> Nothing here is marked `reviewed` until **you** have read it — every note starts `unread`. Each resource note says *why* it is worth your time and which concept notes it supports; the concept notes link back. Read a resource only when a concept note sends you there; reading resources before concepts is how people spend six months on DDIA and still cannot design a rate limiter.

**Map:** [[HLD MOC]] · [[Engineering Dashboard]]

> [!tip] The five that pay for themselves first
> [[Designing Data-Intensive Applications]] (chapters 5–9 after weeks 4–8 of [[Start Here]]) · [[AWS Builders Library - Timeouts Retries and Backoff]] · [[Stripe - Idempotency]] · [[Google SRE Workbook - Implementing SLOs]] · [[System Design Interview (Alex Xu)]] for the problem format.

## Books

- [[Designing Data-Intensive Applications]] — Martin Kleppmann · [[Replication]], [[Sharding]], [[Consistency Models]]
- [[System Design Interview (Alex Xu)]] — Alex Xu · [[Back-of-the-Envelope Estimation]], [[Rate Limiting]], [[Consistent Hashing]]
- [[Release It!]] — Michael Nygard · [[Circuit Breakers]], [[Bulkheads]], [[Timeouts and Retries]]
- [[Site Reliability Engineering (Google)]] — Google · [[SLI SLO and SLA]], [[Error Budgets]], [[Monitoring]]
- [[Building Microservices]] — Sam Newman · [[Microservices]], [[Service Discovery]], [[Saga Pattern]]
- [[Database Internals]] — Alex Petrov · [[Relational Database Internals]], [[Indexing]], [[Replication]]
- [[Understanding Distributed Systems]] — Roberto Vitillo · [[Consistency Models]], [[Leader Election]], [[Failure Detection]]
- [[Fundamentals of Software Architecture]] — Mark Richards and Neal Ford · [[Evaluating Architectural Complexity]], [[Modular Monoliths]], [[Designing Systems for Changing Requirements]]

## Papers

- [[Dynamo Paper]] — Amazon (DeCandia et al.) · [[Consistent Hashing]], [[Quorums]], [[Gossip Protocols]]
- [[Raft Paper]] — Diego Ongaro and John Ousterhout · [[Consensus]], [[Leader Election]], [[Replication]]
- [[Spanner Paper]] — Google · [[Distributed Transactions]], [[Clocks and Time]], [[Consistency Models]]
- [[The Tail at Scale]] — Jeffrey Dean and Luiz Barroso · [[Latency]], [[Fan-Out]], [[Timeouts and Retries]]
- [[PACELC Paper]] — Daniel Abadi · [[PACELC]], [[CAP Theorem]], [[Coordination vs Availability]]
- [[Gorilla Time-Series Paper]] — Facebook (Pelkonen et al.) · [[Time-Series Databases]], [[Metrics]]
- [[Scaling Memcache at Facebook]] — Facebook (Nishtala et al.) · [[Cache Consistency]], [[Cache Stampedes]], [[Distributed Caching]]

## Primary documentation and specs

- [[Jepsen Consistency Models]] — Kyle Kingsbury · [[Consistency Models]], [[Isolation Levels]], [[Eventual Consistency]]
- [[Kafka Design Documentation]] — Apache Kafka · [[Kafka]], [[Topic Partitions]], [[Consumer Groups]]
- [[Redis Eviction Policies Docs]] — Redis · [[Eviction Policies]], [[Redis]], [[TTL]]
- [[Redis Streams Docs]] — Redis · [[Redis]], [[Consumer Groups]], [[Message Queues]]
- [[PostgreSQL Transaction Isolation Docs]] — PostgreSQL · [[Isolation Levels]], [[Transactions]], [[Optimistic and Pessimistic Locking]]
- [[PostgreSQL Indexes Docs]] — PostgreSQL · [[Indexing]], [[Query Execution and Optimization]]
- [[Debezium Outbox Event Router Docs]] — Debezium · [[Outbox Pattern]], [[Change Data Capture]]
- [[OpenTelemetry Observability Primer]] — OpenTelemetry · [[Distributed Tracing]], [[Tracing]], [[Logging]]
- [[gRPC Core Concepts]] — gRPC · [[gRPC]], [[REST]], [[HTTP and HTTPS]]
- [[OAuth 2.0 RFC 6749]] — IETF · [[OAuth]], [[Authentication]], [[Authorization]]
- [[JWT Best Current Practices RFC 8725]] — IETF · [[JWT]], [[Authentication]]
- [[OWASP Threat Modeling Cheat Sheet]] — OWASP · [[Threat Modeling]], [[Input Validation]], [[Least Privilege]]
- [[OWASP Secrets Management Cheat Sheet]] — OWASP · [[Secrets Management]], [[Encryption at Rest and in Transit]], [[API Keys]]
- [[Temporal Docs on Durable Execution]] — Temporal · [[Saga Pattern]], [[Event-Driven Architecture]], [[Retry Strategies]]
- [[The Twelve-Factor App]] — Adam Wiggins · [[Operational Simplicity]], [[Deployment and Rollback Strategies]], [[Secrets Management]]
- [[microservices.io - Saga Pattern]] — Chris Richardson · [[Saga Pattern]], [[Distributed Transactions]], [[Event-Driven Architecture]]
- [[microservices.io - Transactional Outbox]] — Chris Richardson · [[Outbox Pattern]], [[Change Data Capture]], [[At-Least-Once Delivery]]
- [[Latency Numbers Every Programmer Should Know]] — Jonas Bonér · [[Latency]], [[Back-of-the-Envelope Estimation]]

## Architecture deep dives

- [[AWS Builders Library - Timeouts Retries and Backoff]] — AWS (Marc Brooker) · [[Timeouts and Retries]], [[Retry Strategies]], [[Retry Storms]]
- [[AWS Builders Library - Caching Challenges]] — AWS (Matt Brinkley and Jas Chhabra) · [[Cache-Aside]], [[Cache Stampedes]], [[Cache Consistency]]
- [[AWS Builders Library - Load Shedding]] — AWS (David Yanacek) · [[Graceful Degradation]], [[Backpressure]], [[Retry Storms]]
- [[AWS Builders Library - Avoiding Fallback]] — AWS (Jacob Gabrielson) · [[Graceful Degradation]], [[Circuit Breakers]], [[Fault Tolerance]]
- [[AWS Builders Library - Shuffle Sharding]] — AWS (Colm MacCarthaigh) · [[Bulkheads]], [[Fault Tolerance]], [[Hot Keys]]
- [[AWS Builders Library - Idempotent APIs]] — AWS (Malcolm Featonby) · [[Idempotency]], [[Timeouts and Retries]], [[Designing APIs and Contracts]]
- [[Confluent - Exactly-Once Semantics in Kafka]] — Confluent (Neha Narkhede) · [[Exactly-Once Semantics]], [[Kafka]], [[Idempotency]]
- [[Google SRE Workbook - Implementing SLOs]] — Google · [[SLI SLO and SLA]], [[Error Budgets]], [[Alerting]]
- [[Google SRE Book - Monitoring Distributed Systems]] — Google (Rob Ewaschuk) · [[Monitoring]], [[Metrics]], [[Alerting]]

## Real-world engineering case studies

- [[Discord - How Discord Stores Trillions of Messages]] — Discord · [[Hot Keys]], [[Data Modeling]], [[Sharding]]
- [[Uber - Domain-Oriented Microservice Architecture]] — Uber · [[Microservices]], [[Domain-Driven Design]], [[API Gateways]]
- [[Netflix - Fault Tolerance in a High Volume Distributed System]] — Netflix · [[Circuit Breakers]], [[Bulkheads]], [[Graceful Degradation]]
- [[Cloudflare - Rate Limiting at Scale]] — Cloudflare · [[Rate Limiting]], [[Hot Keys]], [[Distributed Caching]]
- [[Shopify - Deconstructing the Monolith]] — Shopify · [[Modular Monoliths]], [[Monoliths]], [[Separation of Concerns]]

## Engineering blog posts

- [[Please Stop Calling Databases CP or AP]] — Martin Kleppmann · [[CAP Theorem]], [[Consistency Models]], [[Network Partitions]]
- [[AWS Architecture Blog - Exponential Backoff and Jitter]] — Marc Brooker · [[Timeouts and Retries]], [[Retry Strategies]], [[Retry Storms]]
- [[Stripe - Idempotency]] — Stripe (Brandur Leach) · [[Idempotency]], [[Designing APIs and Contracts]], [[Duplicate Message Processing]]
- [[Stripe - Online Migrations at Scale]] — Stripe · [[Schema Evolution]], [[Versioning and Migrations]], [[Backward Compatibility]]
- [[Stripe - API Versioning]] — Stripe · [[Versioning and Migrations]], [[Backward Compatibility]], [[Designing APIs and Contracts]]
- [[Brandur - Implementing Idempotency Keys]] — Brandur Leach · [[Idempotency]], [[Transactions]], [[Saga Pattern]]
- [[Martin Fowler - CQRS]] — Martin Fowler · [[CQRS]], [[Event Sourcing]]
- [[Martin Fowler - Event Sourcing]] — Martin Fowler · [[Event Sourcing]], [[CQRS]], [[Audit Logs]]
- [[Martin Fowler - Strangler Fig Application]] — Martin Fowler · [[Strangler Pattern]], [[Designing Systems for Changing Requirements]]
- [[Martin Fowler - Feature Toggles]] — Pete Hodgson · [[Feature Flags]], [[Deployment and Rollback Strategies]], [[Incremental Delivery]]
- [[Martin Fowler - Microservices]] — James Lewis and Martin Fowler · [[Microservices]], [[Monoliths]], [[Data Ownership and Lifecycle]]
- [[Martin Fowler - Technical Debt Quadrant]] — Martin Fowler · [[Technical Debt]], [[Designing for Maintainability]]
- [[Martin Fowler - The Practical Test Pyramid]] — Ham Vocke · [[Testing Strategies]], [[Designing for Maintainability]]
- [[Hexagonal Architecture (Cockburn)]] — Alistair Cockburn · [[Hexagonal Architecture]], [[Separation of Concerns]], [[Testing Strategies]]

## Courses and lecture series

- [[MIT 6.5840 Distributed Systems Course]] — MIT · [[Consensus]], [[Replication]], [[Distributed Transactions]]
- [[Fly.io Gossip Glomers Challenges]] — Fly.io and Kyle Kingsbury · [[Gossip Protocols]], [[Distributed ID Generation]], [[Consensus]]
- [[Kleppmann Distributed Systems Lecture Notes]] — Martin Kleppmann · [[Clocks and Time]], [[Consensus]], [[Consistency Models]]

## Conference talks

- [[Turning the Database Inside Out]] — Martin Kleppmann · [[Event Sourcing]], [[Change Data Capture]], [[Kafka]]
- [[Simple Made Easy]] — Rich Hickey · [[Evaluating Architectural Complexity]], [[Operational Simplicity]], [[Designing for Maintainability]]
- [[A Philosophy of Software Design (Talk)]] — John Ousterhout · [[Designing for Maintainability]], [[Separation of Concerns]], [[Technical Debt]]

## Open-source implementations

- [[System Design Primer (GitHub)]] — Donne Martin · [[Scalability]], [[Back-of-the-Envelope Estimation]], [[Load Balancers]]
- [[etcd Raft Library]] — etcd · [[Consensus]], [[Leader Election]]
- [[Resilience4j]] — Resilience4j · [[Circuit Breakers]], [[Bulkheads]], [[Rate Limiting]]
- [[Debezium]] — Debezium · [[Change Data Capture]], [[Outbox Pattern]], [[Search Indexing Pipelines]]
- [[Bucket4j]] — Bucket4j · [[Rate Limiting]], [[Redis]]
- [[Temporal]] — Temporal · [[Saga Pattern]], [[Retry Strategies]], [[Idempotency]]
- [[Toxiproxy]] — Shopify · [[Network Partitions]], [[Timeouts and Retries]], [[Circuit Breakers]]
- [[Caffeine Cache]] — Ben Manes · [[Eviction Policies]], [[Multi-Level Caching]], [[Cache-Aside]]
- [[Unleash Feature Flags]] — Unleash · [[Feature Flags]]
- [[BullMQ]] — Taskforce.sh · [[Message Queues]], [[Retry Strategies]], [[Dead-Letter Queues]]
- [[pgvector]] — pgvector · [[Search Engines]], [[Indexing]]
- [[Jepsen]] — Kyle Kingsbury · [[Consistency Models]], [[Network Partitions]], [[Isolation Levels]]

## Unread, by priority

![[Resources.base#Unread by priority]]
