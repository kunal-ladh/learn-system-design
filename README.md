# learn-system-design

A system design curriculum written as an Obsidian vault — 181 topic notes across 18 modules, from
foundations through planet-scale case studies and interview problems.

## Vaults in this repo

Each folder below is its own Obsidian vault (open the folder, not the repo root, in Obsidian):

- **`HLD/`** — high-level design for SDE-2 interviews: 166 concept notes, 28 design problems, resources, projects, Bases dashboard, Excalidraw canvases. Start at `HLD/Start Here.md`.
- **`DSA/`** — data structures & algorithms: 29 reading-style topic deep dives (DP, graphs, tries, Fenwick, segment trees, union-find and the rest), one note per NeetCode 150 problem with a Java solution, a Java collections refresher, an 8-week plan, Bases dashboard and 455 Excalidraw canvases (topic map, problem map, every diagram and data-structure trace). Start at `DSA/Start Here.md`; the plugins to enable are listed in `DSA/Plugins to Install.md`.
- **`system-design/`** — the original system design curriculum described below.

## Start here

Open the `system-design/` folder as an Obsidian vault, then open:

- **`System Design Index.md`** — the master map of content: all 18 modules, study tracks, progress tracker.
- **`Numbers and Formulas.md`** — one-page recall sheet: latency tiers, availability math, estimation and quorum formulas.
- **`Practice Drills.md`** — retrieval questions per module (116), design problems by tier, an 11-week schedule.
- **`Database Scaling Path.md`** — guided trail through the read/write scaling chain, indexing to sagas.
- **`Microservices Patterns Path.md`** — every microservices pattern grouped by the problem it solves, edge to infrastructure.
- **`17 Shipping Code/`** — how a commit becomes a running pod: Bitbucket → Jib → ECR → EKS, with a Kubernetes primer and glossary.
- **`18 AWS Services/`** — the AWS side of each concept, every note with a *cost shape* section; start at `AWS Services Map.md`.

## How the notes are structured

Every component note follows the same skeleton, so the shape is predictable once you've read one:

> the problem that created it → what it solves → how it works → trade-offs → failure modes →
> when not to use it → in your stack → interview angle → check yourself → sources

Concept, case-study, and design-problem notes use adapted versions of the same spine.

Notes link to each other with wikilinks and carry YAML frontmatter (`tags`, `type`, `status`,
`summary`, `related`), so Dataview and Bases queries work across the vault. Diagrams are Mermaid,
restricted to the subset Obsidian renders reliably (`graph TD`/`LR`, `sequenceDiagram`,
`stateDiagram-v2`).
