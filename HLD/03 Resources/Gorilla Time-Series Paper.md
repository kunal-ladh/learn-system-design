---
title: Gorilla Time-Series Paper
type: resource
resource_type: paper
tags: [resource, paper]
url: https://www.vldb.org/pvldb/vol8/p1816-teller.pdf
author: "Tuomas Pelkonen et al. (Facebook), VLDB 2015"
year: "2015"
status: unread
priority: low
related_concepts: ["[[Time-Series Databases]]", "[[Metrics]]"]
related_problems: ["[[Design a Metrics and Monitoring System]]"]
summary: "Facebook's in-memory time-series store: delta-of-delta timestamps and XOR-compressed floats bring a data point down to about 1.37 bytes, with availability chosen over consistency."
created: 2026-11-03
updated: 2026-11-03
---

# Gorilla Time-Series Paper

> [!abstract] Why this resource
> It is the clearest explanation of why time-series databases are a different animal from general stores, and the compression ideas reappear in Prometheus and most TSDBs. Read it when [[Design a Metrics and Monitoring System]] asks you to justify storage numbers.

**Link:** [https://www.vldb.org/pvldb/vol8/p1816-teller.pdf](https://www.vldb.org/pvldb/vol8/p1816-teller.pdf)  
**Type:** Paper · **Author:** Tuomas Pelkonen et al. (Facebook), VLDB 2015 · **Year:** 2015

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> PDF (VLDB vol. 8, 2015). Link fetched and verified 2026-11-03.

## When to read it

Sections 3–4 (compression and the in-memory data structures) after [[Time-Series Databases]]; about an hour.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- Timestamps use delta-of-delta encoding: a delta-of-delta of zero is a single '0' bit; values in [-63, 64] take '10' plus 7 bits.
- Values are XOR-compressed against the previous value; about 51% of values compress to a single bit because they repeat.
- Average 1.37 bytes per data point, down from 16 — a 12× reduction; data is grouped in two-hour blocks because larger buckets gain nothing.
- The log is not a write-ahead log: writes buffer up to 64 kB before flushing, trading small data loss on crash for write availability.
- Each series is written to two independent instances in separate regions with no consistency guarantee; reads go to the closest available one.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Metrics]]
- [[Time-Series Databases]]

### Design problems

- [[Design a Metrics and Monitoring System]]
