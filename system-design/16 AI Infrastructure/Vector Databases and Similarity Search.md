---
title: Vector Databases and Similarity Search
tags: [system-design, ai-infra]
type: note
status: evergreen
created: 2026-10-01
updated: 2026-10-01
summary: Why nearest-neighbor search over embeddings needs special indexes (HNSW, IVF), how the recall-vs-latency knob works, and when pgvector-in-your-existing-DB beats a dedicated vector engine.
related: ["[[RAG Architecture]]", "[[LLM Serving Infrastructure]]", "[[Database Indexes]]", "[[Choosing a Database]]"]
---

# Vector Databases and Similarity Search

A vector database stores **embeddings** — lists of numbers that represent the *meaning* of a piece of text, an image, or a user — and answers one query shape fast: "give me the stored items most similar to this one." No ML knowledge is required to design around them: an embedding model is a black box that turns content into coordinates, and everything after that is a classic system design problem — indexing, recall-vs-latency trade-offs, filtering, and operational cost. The 2026-interview-relevant insight is that this is a *database indexing* problem wearing an AI costume, and most of your existing intuitions from [[Database Indexes]] transfer.

## The problem that created it

Start with a product need that predates LLMs: "find things *like* this thing." Similar support tickets, similar products, near-duplicate images, documents relevant to a question. Keyword search ([[RAG Architecture|BM25-style]]) matches literal words, so "my card is blocked" never finds a document titled "debit freeze resolution" — same meaning, zero shared terms.

Embedding models fixed the representation problem. An embedding model maps content to a point in a high-dimensional space (typically 384–3072 dimensions) such that **semantically similar content lands close together**. "Card blocked" and "debit freeze" become nearby points. Similarity is now geometry: compute the distance (usually **cosine similarity** or dot product) between two vectors, and closer means more related. You don't need to know how the model does this — treat it as a deterministic function `embed(text) -> float[1536]` and design around its output.

That created a new database problem. The query is **k-nearest-neighbor (kNN)**: given a query vector, return the k closest stored vectors. The naive implementation compares the query against *every* stored vector — a full scan. At 10,000 vectors that's fine. At 100 million vectors × 1536 dimensions, every query is hundreds of gigabytes of floating-point math; latency is seconds and the CPU bill is absurd. And the trusty B-tree can't help: B-trees index *one ordered dimension*, and there is no single ordering of points in 1536-dimensional space that preserves "closeness" (this is the **curse of dimensionality** — in very high dimensions, classic spatial indexes like R-trees degrade toward full scans too).

So a new index family was needed: one that answers "what's near this point?" in sublinear time, by giving up a little correctness.

## What it solves

- **Semantic retrieval at scale**: meaning-based search over millions to billions of items with single- to double-digit-millisecond latency.
- **The retrieval half of RAG**: every [[RAG Architecture]] pipeline needs exactly this query served fast; the vector index is its hot path.
- **Recommendation and deduplication**: "users like you", "near-duplicate KYC document detection" — any *similarity* problem, not just chat-with-your-docs.
- **A tunable accuracy dial**: unlike a B-tree (always exact), ANN indexes let you trade recall for latency per query — a knob most databases never give you.

## How it works

The core move is **approximate nearest neighbor (ANN)** search: return *mostly* the true top-k, much faster. The quality metric is **recall@k** — what fraction of the true k nearest neighbors the index actually returned. Production systems typically run at 0.9–0.99 recall; the application decides how much approximation it can tolerate.

One practical preliminary: the **distance metric** is part of the contract. Cosine similarity (angle between vectors — the usual choice for text embeddings), dot product, and Euclidean distance rank neighbors differently; the index is built for one metric, and it must match what the embedding model was trained for (most text-embedding models publish "use cosine"). Many systems sidestep the cosine-vs-dot distinction by storing vectors **normalized to unit length**, where the two become equivalent. Getting this wrong doesn't error — it just quietly ranks results worse, a recurring theme in this note.

Two index families dominate; both are worth knowing at concept level only.

### HNSW — the graph approach (the modern default)

**HNSW (Hierarchical Navigable Small World)** builds a graph where each vector is a node connected to its near neighbors, organized in layers like a skip list: the top layer has few nodes with long-range links, lower layers get denser and more local. A query enters at the top, greedily hops toward the target ("which of my neighbors is closest to the query?"), and drops a layer each time it can't improve — coarse navigation first, fine navigation last. Search cost grows roughly logarithmically with collection size.

- **Knob**: `efSearch` — how many candidate nodes to explore during a query. Higher = better recall, more latency. This is the recall-vs-latency dial, settable *per query*.
- **Character**: excellent recall/latency; the index lives in RAM (graph + vectors), so memory is the cost driver; inserts are supported but deletes/updates degrade the graph over time and may need periodic rebuilds.

### IVF — the clustering approach

**IVF (Inverted File index)** partitions vectors into clusters up front (k-means over the dataset), each cluster owning a centroid. A query finds the few nearest centroids and scans only those clusters' members — like [[Sharding|sharding]] the vector space by region and querying only the relevant shards.

- **Knob**: `nprobe` — how many clusters to scan. More clusters = better recall, more latency. Same dial, different mechanism.
- **Character**: smaller memory footprint, pairs well with compression (**product quantization**, which shrinks vectors lossily to fit billion-scale datasets in RAM or on disk); recall is more sensitive to data distribution, and the clustering must be retrained if the data drifts.

```mermaid
graph TD
    Q["Query vector"] --> C{"Index type"}
    C --> H["HNSW - greedy hops through layered graph"]
    C --> I["IVF - pick nearest clusters, scan only those"]
    H --> K["Approximate top k"]
    I --> K
    K --> R["Recall vs latency tuned by efSearch or nprobe"]
```

**Rule of thumb**: HNSW for best recall/latency when the dataset fits in memory (most workloads); IVF + quantization when it doesn't. Both are configuration details behind the same contract: approximate top-k with a tunable dial.

A sizing instinct worth carrying: raw vectors cost `count × dimensions × 4 bytes` (float32). Ten million 1536-dimensional vectors ≈ 61 GB before the graph overhead — which is why quantization (compressing each vector to a fraction of its size, trading a little recall) and dimension choice are capacity decisions, not tuning trivia, and why "just put it all in HNSW" stops being an answer somewhere in the tens of millions. Past RAM entirely, disk-resident ANN designs (the DiskANN family) keep a compressed sketch in memory and fetch full vectors from SSD — the by-now-familiar RAM-vs-SSD cost trade, applied to vector search.

### The write path — builds, inserts, deletes

Reads get the attention, but the write path shapes operations. HNSW supports incremental inserts (each new vector wires itself into the graph), so streaming upserts work — but **deletes are soft**: the node is tombstoned, still occupying memory and still visited during traversal, so a high-churn corpus degrades until a periodic rebuild compacts the graph. IVF must *train* its clusters on representative data before serving, so it naturally fits a bulk-build-then-serve lifecycle, with retraining when the data distribution drifts. Either way, the operational pattern for big changes is the same as [[Rebalancing Partitions]]: build the new index alongside the old, verify recall on a golden set, cut over, drop the old.

### Metadata filtering — the pre- vs post-filter problem

Real queries are never pure similarity: "nearest documents *where tenant_id = 42 and lang = 'en'*." Combining a structural filter with an ANN index is genuinely awkward:

- **Post-filtering**: run ANN first, then drop results failing the filter. Simple, but if the filter is selective (say it matches 1% of data), your top-100 may contain zero survivors — you return too few results or must re-query with a much larger k.
- **Pre-filtering**: apply the filter first, then search only matching vectors. Correct results, but the ANN index was built over *all* vectors — a graph traversal restricted to a sparse subset keeps hitting excluded nodes and degrades toward a scan of the filtered set.

Dedicated engines invest heavily here (filter-aware HNSW traversal, per-tenant segmenting/partitioning); this is a genuine differentiator to probe when [[Choosing a Database|choosing]]. Compare [[Secondary Indexes on Sharded Data]] — it's the same "two access paths that don't compose cheaply" problem.

## Trade-offs

| You gain | You pay |
|---|---|
| Sublinear semantic search at huge scale | Approximate results — recall < 100% by design |
| Per-query recall/latency dial | RAM-heavy indexes (HNSW); index build time on bulk loads |
| Meaning-based matching keyword search can't do | A new stateful system to operate (if dedicated engine) |
| Filterable, hybrid-capable retrieval | Pre/post-filter complexity; worst cases on selective filters |
| | **Embedding-model coupling** — see failure modes |

The honest market take, as of ~2026:

- **Modest scale (≤ a few million vectors, moderate QPS): use an extension of the database you already run.** **pgvector** (Postgres), MongoDB's vector search (an Atlas feature), Redis's vector search, OpenSearch/Elasticsearch kNN — all give you HNSW inside an engine your team already operates, with real transactions, backups, and filtering via normal predicates. One fewer system, one fewer consistency boundary.
- **Large scale (tens of millions to billions, high QPS, heavy filtering, multi-tenant): dedicated engines earn their keep** — the Pinecone / Milvus / Qdrant / Weaviate class. They offer segment-based scaling, filter-aware indexes, quantization options, and separation of index building from serving.
- The failure pattern interviewers like to hear named: teams adopt a dedicated vector DB at 50K vectors "for the future", inheriting a second datastore, a second [[Replication|replication]] story, and a dual-write consistency problem ([[The Outbox Pattern]] territory) that pgvector would have made a non-issue.

## Failure modes

- **Embedding version skew — the hidden operational trap.** Vectors are only comparable if produced by the *same model version*. Upgrade or switch your embedding model and every stored vector is silently incompatible with new query vectors — distances become meaningless, retrieval quality collapses *without errors in any log*. The fix is a **full re-embed and reindex** of the corpus: for 100M documents that's a large batch job with real GPU/API cost and hours-to-days of runtime, usually run as a dual-index blue/green cutover (like [[Rebalancing Partitions]], but you must also recompute every value). Tag every vector with its model version and treat model upgrades as migrations, not config changes.
- **Silent recall degradation.** ANN quality drifts as data grows or distribution shifts (IVF centroids go stale; HNSW accumulates delete tombstones). Nothing errors — relevance just gets worse. Mitigation: a golden set of queries with known-correct neighbors, evaluated continuously, exactly like a latency SLO ([[SLA SLO SLI and Error Budgets]]).
- **Filtered-query starvation.** A selective filter plus post-filtering returns 3 results where the user expected 50. Detected by monitoring result-set sizes per filter class; mitigated by pre-filter support or partitioning by tenant.
- **Memory blowouts.** HNSW RAM ≈ vectors + graph edges; growth is linear in corpus size and people forget the graph overhead. OOM on the index node takes retrieval down entirely.
- **Slow recovery after restart.** The HNSW graph lives in RAM; after a node restart it must be reloaded from a persisted snapshot — or, if nobody persisted it, *rebuilt from scratch*, which at tens of millions of vectors is minutes to hours of degraded or absent retrieval. Persist the built index and measure reload time; "rebuild on boot" is not a recovery plan.
- **Dual-store drift.** When the source of truth (your document DB) and the vector index are separate systems, missed updates leave the index serving deleted or stale content — see [[RAG Architecture]] freshness and [[Change Data Capture]].

## When NOT to use it

- **Exact or keyword lookup suffices.** "Find invoice by number" or "filter transactions by status" is B-tree work. Don't embed what an exact index answers.
- **Tiny corpora.** Under ~100K vectors, brute-force exact kNN is single-digit milliseconds on one CPU core — skip the ANN index and its recall complications entirely.
- **Hard correctness requirements.** If missing a true match is unacceptable (compliance screening, sanctions matching), approximate retrieval alone is the wrong tool — use it as a *candidate generator* in front of an exact verification step, never as the authority. In a fintech context this is a fail-closed rule.
- **As your primary datastore.** Vector engines are retrieval indexes, not systems of record: weak or absent transactions, limited query languages. Keep the source of truth in a real database and treat the vector index as derived data (rebuildable from the source).

## In your stack

- **Start with what you run.** You operate MongoDB and Redis heavily; both have vector search offerings (MongoDB's is an Atlas feature — self-managed Mongo doesn't ship it, so for your clusters Redis or pgvector is the realistic first step). A dedicated engine is a new stateful system on EKS or a new SaaS vendor — justify it with scale numbers, not fashion.
- **Fintech similarity use cases** are mostly candidate generation: similar support tickets for agents, near-duplicate KYC document detection, transaction-description clustering. Each one tolerates approximate recall because a human or a deterministic rule verifies downstream — which is exactly where ANN belongs under an RBI-regulated correctness bar.
- **Treat the vector index as derived data.** Source of truth stays in MongoDB; the index rebuilds from it. That one decision defuses the scariest failure modes (drift, version skew) into "rerun the pipeline" instead of "restore the database".
- **Cost shape**: HNSW cost scales with *corpus size × dimensions* (RAM), query cost with *QPS × efSearch*; re-embedding cost scales with *corpus size × model price* and recurs on every model upgrade. Name all three in any proposal.

## Interview angle

- Interviewers probe whether you know **why kNN needs a special index** (no total ordering in high dimensions — B-trees don't apply) and whether you say the word **recall** unprompted. "ANN trades recall for latency, tunable per query" is the sentence that signals understanding.
- **60-second version**: "Embeddings turn content into points where distance means semantic similarity. Exact nearest-neighbor is a full scan, so at scale we use approximate indexes: HNSW navigates a layered neighbor graph — the default, great recall, RAM-hungry; IVF clusters the space and scans only nearby clusters — cheaper memory, pairs with quantization for billion scale. Both expose a recall-vs-latency knob. Metadata filtering is the awkward part — pre-filter degrades the index, post-filter starves selective queries. At modest scale I'd use pgvector or an existing-DB extension; dedicated engines earn their complexity at tens of millions of vectors or heavy filtered multi-tenant load. The hidden trap is embedding versioning: a model upgrade means re-embedding the entire corpus."
- Classic follow-ups: "Why not a B-tree?" (one-dimensional ordering); "What happens when you change embedding models?" (full reindex — have a migration plan); "How do you filter by tenant?" (pre vs post, filter-aware indexes); "How do you *measure* retrieval quality?" (recall@k against a golden set).
- A sizing follow-up worth pre-computing: "How much RAM for N vectors?" — `N × dims × 4 bytes` plus graph overhead; doing the arithmetic aloud (the [[Back of the Envelope Estimation]] habit) is the strongest signal you've operated one rather than read about one.

## Check yourself

> [!question]- Q: Explain to a backend engineer why nearest-neighbor search can't use a B-tree, and what HNSW does instead.
> A B-tree needs one ordered dimension — it answers range and equality queries over a sort key. Points in 1536-dimensional space have no single ordering that preserves closeness, so no sort key exists ("curse of dimensionality"). HNSW instead builds a layered graph of neighbor links: queries enter a sparse top layer, greedily hop toward the target, and descend into denser layers — coarse-to-fine navigation, roughly logarithmic cost, returning *approximately* the nearest neighbors with a tunable candidate-exploration parameter (efSearch) controlling recall vs latency.

> [!question]- Q: What happens if the team upgrades the embedding model but only applies it to newly ingested documents?
> The index now mixes two incompatible coordinate systems. New queries (embedded with the new model) measure meaningless distances against old vectors, so old documents effectively vanish from results — and vice versa. No error is thrown anywhere; retrieval quality silently craters, usually noticed via user complaints or a golden-set recall drop. The fix is a full re-embed of the corpus with a versioned, blue/green reindex and an atomic cutover.

> [!question]- Q: A query filters on `tenant_id` matching 0.5% of a 50M-vector corpus. Why do both pre- and post-filtering struggle, and what do engines do about it?
> Post-filter: ANN returns the global top-k, of which ~0.5% survive the filter — top-100 yields ~0 results, forcing huge-k re-queries. Pre-filter: restricting an HNSW traversal to 250K scattered nodes breaks graph connectivity and degrades toward brute force. Engines solve it structurally: filter-aware traversal (skip-but-route-through excluded nodes), or partitioning/segmenting the index by tenant so each tenant searches its own small index — the [[Sharding]] instinct applied to vector space.

> [!question]- Q: Mini design prompt — semantic search over 2M support tickets, 50 QPS, must filter by product line, team already runs Postgres and MongoDB. Pick the storage and justify.
> pgvector in the existing Postgres: 2M vectors × ~1536 dims fits comfortably in RAM on one node, HNSW gives millisecond ANN at 50 QPS, and product-line filtering is a plain WHERE clause the planner can combine with the vector index. A dedicated engine adds a second stateful system, a dual-write consistency problem, and new ops burden for zero capability the workload needs. Revisit only if the corpus grows ~50×, QPS grows an order of magnitude, or filtered recall measurably degrades. Plan the re-embed path (model version column, rebuildable from source of truth) from day one.

## Sources

- Malkov & Yashunin, "Efficient and Robust Approximate Nearest Neighbor Search Using Hierarchical Navigable Small World Graphs" (the HNSW paper, arXiv ~2016, IEEE TPAMI 2018).
- FAISS documentation (Meta AI) — IVF, product quantization, and the practical index-selection guidelines.
- Subramanya et al., "DiskANN: Fast Accurate Billion-point Nearest Neighbor Search on a Single Node" (NeurIPS 2019) — the disk-resident ANN approach.
- pgvector project documentation — HNSW and IVFFlat index options inside Postgres.
- Pinecone learning center — accessible HNSW and ANN explainers (vendor, but technically solid).
- Qdrant and Milvus official documentation — filterable HNSW and segment-based architecture; useful for seeing how dedicated engines attack the pre-filter problem.
- *Designing Data-Intensive Applications*, Kleppmann — ch. 3 (indexing fundamentals this note contrasts against) and ch. 12 (derived data — the right mental bucket for a vector index).
