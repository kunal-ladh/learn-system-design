# learn-system-design

A system design curriculum written as an Obsidian vault — 151 notes across 16 modules, from
foundations through planet-scale case studies and interview problems.

## Start here

Open the `system-design/` folder as an Obsidian vault, then open:

- **`System Design Index.md`** — the master map of content: all 16 modules, study tracks, progress tracker.
- **`Practice Drills.md`** — retrieval questions per module, design problems by tier, a 10-week schedule.

## How the notes are structured

Every component note follows the same skeleton, so the shape is predictable once you've read one:

> the problem that created it → what it solves → how it works → trade-offs → failure modes →
> when not to use it → in your stack → interview angle → check yourself → sources

Concept, case-study, and design-problem notes use adapted versions of the same spine.

Notes link to each other with wikilinks and carry YAML frontmatter (`tags`, `type`, `status`,
`summary`, `related`), so Dataview and Bases queries work across the vault. Diagrams are Mermaid,
restricted to the subset Obsidian renders reliably (`graph TD`/`LR`, `sequenceDiagram`,
`stateDiagram-v2`).
