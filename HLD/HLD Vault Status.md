---
title: HLD Vault Status
aliases: [Vault Status, Status and conventions, Changelog]
type: guide
tags: [guide, status]
summary: "What the vault contains, the conventions every note follows, the interview plan it is built around, and a dated log of what changed — the page to read before extending the vault."
created: 2026-11-03
updated: 2026-11-03
---

# HLD Vault Status

> [!abstract] What this page is for
> A one-page inventory of the vault, the property conventions that keep the Bases views working, the plan the vault is organised around, and a changelog. Read it before adding a note type or restructuring anything; update the log when you do. [[Workspace Guide]] explains *how* the machinery works; this page records *what is there and why*.

**Map:** [[Engineering Dashboard]] · [[Start Here]] · [[Workspace Guide]] · [[HLD MOC]] · [[Interview Prep MOC]]

**Vault:** `learn-system-design/HLD` — an Obsidian vault inside the `learn-system-design` git repository, next to the older `system-design` vault (referenced by path only, never linked). Commit after each study session.

**Goal:** SDE-2 system design interview readiness in 4–6 months, and the product-engineering judgement that outlasts the interview.

## What exists

325 notes · 9 Bases files · 7 templates · 248 Mermaid diagrams (all render on Mermaid 11) · 244 Excalidraw drawings · 0 broken wikilinks.

| Folder | Contents | Entry point |
|---|---|---|
| `00 Dashboard/` | [[Engineering Dashboard]] (quick navigation, roadmap, sections 1–7 with 7 = interview readiness) and the Bases: Concepts, Problems, Projects, Resources, Reviews, Revision Queue, Decisions, Interview Readiness, Mock Interviews | [[Engineering Dashboard]] |
| `01 HLD & System Design/` | 166 concept notes on the 17-section template, every one with an Architecture diagram, in eleven category folders; each category has a MOC whose map has one note per clickable node; [[HLD MOC]] (category map) and [[Knowledge Graph]] (six chains) | [[HLD MOC]] |
| `02 System Design Problems/` | 28 decision-sheet problems, each with an architecture diagram and an `interview_tier` (10 core · 10 likely · 8 stretch) | [[Problem Bank MOC]] |
| `03 Resources/` | 79 resource notes — every URL verified on 2026-11-03, all `unread`, "What to look for" pre-filled from the page, two-way links to concepts and problems | [[Resources MOC]] |
| `04 Projects/` | 16 project notes, all `status: idea`, each with an architecture diagram and measured milestones; `Decisions/` for ADRs | [[Projects MOC]] |
| `05 Reviews/` | Weekly reviews created from the template | [[Reviews MOC]] |
| `06 Interview Prep/` | [[Interview Prep MOC]], [[SDE-2 Interview Framework]], [[Interview Probe Questions]], [[Estimation Cheat Sheet]], [[Component Choice Cheat Sheet]], [[SDE-2 Readiness Checklist]], [[Interview Sprint Plan]]; `Mocks/` for mock logs | [[Interview Prep MOC]] |
| `Templates/` | Concept, Design Problem, Project, Resource, Weekly Review, Architecture Decision, Mock Interview | — |
| `Excalidraw/` | 244 drawings in the Obsidian Excalidraw format: `Maps/` — 11 category concept maps + the HLD category map, hand-laid-out (rooms, reading order, one concrete example per room, every box linked to its note); `Concepts/` 172, `Problems/` 29, `Projects/` 17, `Guides/` 14 — one canvas per Mermaid diagram, generated from it and embedded under it in a folded callout | [[HLD Category Map.excalidraw]] |
| root | [[Start Here]], [[Workspace Guide]], [[Plugins to Install]], this page | [[Start Here]] |

## Conventions every note follows

- Every note has a `type` and a matching tag — `concept`, `problem`, `project`, `resource`, `review`, `decision`, `mock`, `moc`, `guide`. The Bases views filter on the tag; dates are ISO (`YYYY-MM-DD`); note names are unique across the vault because wikilinks resolve by basename.
- **Concept:** `category` (the exact folder name), `priority`, `status` (`not-started` → `reading` → `understood` → `applied`; `needs-revision`), `confidence` 1–5, `date_started`, `last_reviewed`, `next_review`, `related`.
- **Problem:** `priority`, `interview_tier` (`core` / `likely` / `stretch`), `status` (`not-started` → `exploring` → `designing` → `implementing` → `reviewed`; `needs-revision`), `confidence`, `last_worked`, `next_review`, `related_concepts`, `related_projects`.
- **Project:** `category` (`backend` / `distributed` / `realtime` / `financial` / `ai` / `devtools` / `productivity` / `experimental`), `status` (`idea` → `planned` → `active` → `shipped`; `paused`, `archived`), `current_milestone`, `next_task`, `blocker`, `repo`, `demo`.
- **Resource:** `resource_type` (`book` / `paper` / `documentation` / `deep-dive` / `case-study` / `blog` / `course` / `talk` / `open-source`), `url`, `author`, `year`, `status` (`unread` → `reading` → `reviewed`), `priority`. Nothing is `reviewed` until you have read it.
- **Mock:** `problem` (a wikilink), `date`, `format` (`solo` / `peer` / `paid`), `duration_min`, six `score_*` properties (1–5) and `overall`.
- `.obsidian/types.json` declares the property types; Dataview (pre-placed) powers only the folded "numbers" panels and the mock trend table; everything else is core Bases and built-in Mermaid.

## UX machinery

- **Clickable diagrams** — any flowchart node whose label is exactly a note title carries `class <id> internal-link` (343 nodes across 32 notes). Keep node labels identical to note titles in new diagrams.
- **Diagram sizing** — the `hld-workspace` CSS snippet (pre-enabled in `.obsidian/appearance.json`) renders Mermaid at natural size with sideways scrolling instead of shrinking; long chains are drawn top-to-bottom, so nothing is wider than about 900px. Use `flowchart TB` for chains longer than five nodes.
- **Wide pages** — `cssclasses: [wide]` on the dashboard, the MOCs and the interview cheat sheets gives tables and Bases the full pane width.
- **Bookmarks and graph** — `.obsidian/bookmarks.json` groups the entry points (Learn · Design and build · Interview · Review); `.obsidian/graph.json` colours nodes by folder, including `06 Interview Prep`, and greys `Excalidraw/`.
- **Excalidraw drawings** — generated from the Mermaid sources with the excalidraw-diagram skill's conventions (semantic colours: orange start, blue concept, darker blue hub or store, amber decision, red failure mode, green destination; clean strokes; Cascadia text). Boxes link to notes through the drawing's `link` property and the `## Element Links` section, so backlinks and the graph see them. Regenerate a drawing by deleting it and re-running the generator, or just edit it in Excalidraw — the Mermaid block stays the reference. The one diagram without a canvas is the Gantt chart in [[SDE-2 Interview Framework]].

## The plan the vault is organised around

- **Timeline:** 4–6 months. Phase 1 is the twelve-week order in [[Start Here]]; Phase 2 (weeks 13–18) is second passes on the core problems, one project to its measured milestone and the likely-tier problems; Phase 3 is the six-week sprint in [[Interview Sprint Plan]] — daily drills, timed designs, two mocks a week, a taper.
- **Core problems** (design twice, mock once): [[Design a URL Shortener]], [[Design a Rate Limiter]], [[Design a Notification System]], [[Design a Chat System]], [[Design a Social Feed]], [[Design a Distributed Cache]], [[Design a Ticket Booking System]], [[Design a Payment System]], [[Design a Ride-Sharing Service]], [[Design Search Autocomplete]].
- **Readiness bar:** every core problem `reviewed` with confidence 4, every must-know concept at confidence 3 or more, six logged mocks with the last two scoring 4+ on trade-offs and communication — see [[SDE-2 Readiness Checklist]].

## Open items

- [ ] Commit the vault after this session (`git add HLD && git commit`).
- [ ] Confirm the CSS snippet shows as enabled (Settings → Appearance → CSS snippets); if Obsidian is older than 1.9, update it so the Bases embeds resolve.
- [ ] Install and enable the Excalidraw plugin, then run the three checks in [[Plugins to Install]].
- [ ] Optional: QuickAdd and Templater setup as described in [[Workspace Guide]]; an `asked: true` property on problems after the first real loop.
- [ ] Build [[Spaced Repetition Review CLI]] once setting `next_review` by hand becomes a chore.

## Changelog

- **2026-10-19 → 2026-10-30** — Curriculum built: 166 concept notes, 11 category MOCs, HLD MOC, Knowledge Graph, 28 problem decision sheets, dashboard with seven Bases, templates, Start Here and Workspace Guide; Webhook Relay project.
- **2026-11-03** — Gaps closed and interview layer added: 79 resource notes with verified links; 15 project notes; `06 Interview Prep/` (framework, probe bank, estimation and component cheat sheets, readiness checklist, sprint plan, mock template, two Bases); `interview_tier` on every problem; dashboard section 7; guides updated.
- **2026-11-03 (UX pass)** — Diagrams added to the nine Product Engineering concepts, every MOC, Start Here, the dashboard and the interview notes; category maps rewritten with one note per clickable node; 30 over-wide chains redrawn top-to-bottom; CSS snippet, bookmarks and graph colours added; this page created.
- **2026-11-03 (Excalidraw pass)** — `Excalidraw/` added: 12 hand-laid-out maps (eleven categories + HLD) and 232 generated canvases, one per Mermaid diagram, each embedded in its note; [[Plugins to Install]] written; guides, bookmarks and graph colours updated.
