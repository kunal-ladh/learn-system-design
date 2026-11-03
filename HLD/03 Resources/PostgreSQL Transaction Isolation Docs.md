---
title: PostgreSQL Transaction Isolation Docs
type: resource
resource_type: documentation
tags: [resource, documentation]
url: https://www.postgresql.org/docs/current/transaction-iso.html
author: "The PostgreSQL Global Development Group"
year: ""
status: unread
priority: high
related_concepts: ["[[Isolation Levels]]", "[[Transactions]]", "[[Optimistic and Pessimistic Locking]]"]
related_problems: ["[[Design a Digital Wallet and Ledger]]", "[[Design a Distributed Job Scheduler]]", "[[Design a Ticket Booking System]]", "[[Design an Inventory Management System]]"]
summary: "The manual section on Read Committed, Repeatable Read and Serializable in PostgreSQL — how each is implemented with MVCC and which anomalies each still allows."
created: 2026-11-03
updated: 2026-11-03
---

# PostgreSQL Transaction Isolation Docs

> [!abstract] Why this resource
> Most interview answers about isolation are textbook; this page tells you what the database you actually run does, including that Repeatable Read is snapshot isolation and that Serializable uses predicate locks and may abort you with SQLSTATE 40001. [[Design a Ticket Booking System]] and the ledger problems depend on these facts.

**Link:** [https://www.postgresql.org/docs/current/transaction-iso.html](https://www.postgresql.org/docs/current/transaction-iso.html)  
**Type:** Primary documentation · **Author:** The PostgreSQL Global Development Group

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Official manual (currently PostgreSQL 18), free. Link fetched and verified 2026-11-03.

## When to read it

Sections 13.2.1–13.2.3 with [[Isolation Levels]] in week 4 of [[Start Here]]; then reproduce the website.hits example in psql with two sessions.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Table 13.1: Read Committed allows non-repeatable reads, phantoms and serialization anomalies; Repeatable Read prevents all but serialization anomalies.
- PostgreSQL implements only three distinct levels: Read Uncommitted behaves like Read Committed.
- Read Committed takes a new snapshot per statement, so a single UPDATE can observe an inconsistent snapshot (the website.hits example).
- Repeatable Read is snapshot isolation; a conflicting concurrent update fails with 'could not serialize access due to concurrent update'.
- Serializable uses non-blocking predicate (SIRead) locks and raises serialization failures (SQLSTATE 40001) that the application must retry; sequence changes are never rolled back.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[ACID]]
- [[Isolation Levels]]
- [[Optimistic and Pessimistic Locking]]
- [[Transactions]]

### Design problems

- [[Design a Digital Wallet and Ledger]]
- [[Design a Distributed Job Scheduler]]
- [[Design a Ticket Booking System]]
- [[Design an Inventory Management System]]
