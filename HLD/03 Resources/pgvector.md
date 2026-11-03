---
title: pgvector
type: resource
resource_type: open-source
tags: [resource, open-source]
url: https://github.com/pgvector/pgvector
author: "pgvector project"
year: ""
status: unread
priority: low
related_concepts: ["[[Search Engines]]", "[[Indexing]]"]
related_problems: ["[[Design a RAG Knowledge Assistant]]"]
summary: "Vector similarity search inside PostgreSQL: vector types, distance operators, and HNSW and IVFFlat approximate-nearest-neighbour indexes."
created: 2026-11-03
updated: 2026-11-03
---

# pgvector

> [!abstract] Why this resource
> It lets [[Vault RAG Assistant]] keep one database, and its HNSW/IVFFlat parameters are a good concrete example of the recall-vs-latency trade-off that [[Indexing]] describes abstractly.

**Link:** [https://github.com/pgvector/pgvector](https://github.com/pgvector/pgvector)  
**Type:** Open-source implementation · **Author:** pgvector project

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Public GitHub repository. Link fetched and verified 2026-11-03.

## When to read it

README index sections (HNSW, IVFFlat, iterative scans) when you reach [[Design a RAG Knowledge Assistant]].

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Distance operators: <-> L2, <#> negative inner product, <=> cosine, <+> L1, plus Hamming and Jaccard for bit vectors.
- HNSW defaults m = 16, ef_construction = 64, hnsw.ef_search = 40.
- IVFFlat guidance: build the index after loading data; lists ≈ rows/1000 up to 1M rows and sqrt(rows) above; probes ≈ sqrt(lists).
- Index limits: vector up to 2,000 dimensions, halfvec up to 4,000.
- Iterative index scans (strict or relaxed order) help filtered queries return enough rows.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Full-Text Search]]
- [[Indexing]]
- [[Search Engines]]

### Design problems

- [[Design a RAG Knowledge Assistant]]
