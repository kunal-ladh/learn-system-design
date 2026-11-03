---
title: PostgreSQL Indexes Docs
type: resource
resource_type: documentation
tags: [resource, documentation]
url: https://www.postgresql.org/docs/current/indexes.html
author: "The PostgreSQL Global Development Group"
year: ""
status: unread
priority: medium
related_concepts: ["[[Indexing]]", "[[Query Execution and Optimization]]"]
related_problems: []
summary: "Chapter 11 of the PostgreSQL manual: the six index types, multicolumn and partial indexes, index-only scans, and how to check whether an index is used."
created: 2026-11-03
updated: 2026-11-03
---

# PostgreSQL Indexes Docs

> [!abstract] Why this resource
> [[Indexing]] stays abstract until you know what B-tree, GIN and BRIN actually index and when a partial or covering index wins. This chapter is short and every section ends in something you can try with EXPLAIN.

**Link:** [https://www.postgresql.org/docs/current/indexes.html](https://www.postgresql.org/docs/current/indexes.html)  
**Type:** Primary documentation · **Author:** The PostgreSQL Global Development Group

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Official manual chapter index; the detail is in the linked subsections. Link fetched and verified 2026-11-03.

## When to read it

11.1–11.3 and 11.8–11.9 (index types, multicolumn, partial, index-only scans) with [[Indexing]]; 11.12 (Examining Index Usage) with [[Query Execution and Optimization]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Six index types, each with its own section: B-tree, Hash, GiST, SP-GiST, GIN and BRIN.
- The chapter's framing: indexes find rows much faster but add overhead to the database as a whole.
- Later sections cover multicolumn indexes, indexes and ORDER BY, combining indexes, unique, expression and partial indexes, index-only scans and covering indexes.
- The final section, Examining Index Usage, is how you check with EXPLAIN whether the planner uses your index at all.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Bottleneck Analysis]]
- [[Full-Text Search]]
- [[Indexing]]
- [[Inverted Indexes]]
- [[Query Execution and Optimization]]

### Design problems

- *(none link here yet)*
