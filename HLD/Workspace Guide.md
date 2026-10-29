---
title: Workspace Guide
aliases: [How this vault works, Maintenance guide]
type: guide
tags: [guide]
summary: "How the workspace is organised, which features depend on which plugin, how to add and maintain notes, and the daily and weekly workflow."
created: 2026-10-30
updated: 2026-10-30
---

# Workspace Guide

> [!abstract] One-line summary
> Notes carry properties; Bases turn properties into live tables; the dashboard and the maps are just views over those tables. Keep the properties honest and the rest takes care of itself. For *how to study*, read [[Start Here]]; this page is about *how the machinery works*.

## Folder structure

| Folder | What lives there | Entry point |
|---|---|---|
| `00 Dashboard/` | [[Engineering Dashboard]] and the seven `.base` files that power every table | [[Engineering Dashboard]] |
| `01 HLD & System Design/` | 166 concept notes in eleven category folders, each with its own MOC, plus [[HLD MOC]] and [[Knowledge Graph]] | [[HLD MOC]] |
| `02 System Design Problems/` | 28 design problems as decision sheets | [[Problem Bank MOC]] |
| `03 Resources/` | One note per resource, linked both ways to concepts | [[Resources MOC]] |
| `04 Projects/` | 16 projects; `Decisions/` holds architecture decision records | [[Projects MOC]] |
| `05 Reviews/` | Weekly reviews | [[Reviews MOC]] |
| `Templates/` | Concept, Design Problem, Project, Resource, Weekly Review, Architecture Decision | — |
| `Attachments/` | Excalidraw drawings and images | — |
| `Start Here.md`, `Workspace Guide.md` | Reading guide and this page | [[Start Here]] |

The `system-design` vault next to this one stays untouched; concept and problem notes point to its deep-dive notes by path in a tip callout.

## Note types and their properties

Every note has a `type` and a matching tag, which is what the Bases views filter on.

| Type | Tag | Tracking properties | Status values |
|---|---|---|---|
| concept | `concept` + category tag | `priority`, `status`, `confidence` (1–5), `date_started`, `last_reviewed`, `next_review`, `related` | `not-started` → `reading` → `understood` → `applied`; `needs-revision` |
| problem | `problem` + category | `priority`, `status`, `confidence`, `last_worked`, `next_review`, `related_concepts`, `related_projects` | `not-started` → `exploring` → `designing` → `implementing` → `reviewed`; `needs-revision` |
| project | `project` + category | `status`, `priority`, `current_milestone`, `next_task`, `blocker`, `repo`, `demo`, `related_concepts`, `related_problems`, `started`, `updated` | `idea` → `planned` → `active` → `shipped`; `paused`, `archived` |
| resource | `resource` + resource type | `resource_type`, `url`, `author`, `status`, `priority`, `related_concepts`, `related_problems` | `unread` → `reading` → `reviewed` |
| review | `review` | `week`, `period_start`, `period_end` | — |
| decision | `decision` | `project`, `status`, `related_concepts` | `proposed` → `accepted`; `superseded`, `rejected` |

**Confidence scale** (track it separately from status): 1 unfamiliar · 2 understand the basic idea · 3 can explain the mechanics · 4 can apply it independently · 5 can evaluate alternatives and defend design decisions.

**Knowledge-gap tags**: add `gap/explain`, `gap/decision`, `gap/internals` or `gap/hands-on` to a concept's `tags` and it appears in the dashboard's gap table; remove the tag when the gap closes.

## Which features depend on which plugin

| Feature | Needs | Notes |
|---|---|---|
| All tables on the dashboard and in the MOCs (`![[Concepts.base#View]]`, inline `base` blocks) | **Bases** — core plugin, Obsidian 1.9+ | Enabled by default; nothing to install. Date filters such as `next_review <= today()` need the date properties to be typed *Date*, which `.obsidian/types.json` already declares. |
| Templates (`{{title}}`, `{{date:YYYY-MM-DD}}`) | **Templates** — core plugin | Settings → Core plugins → Templates; folder is preset to `Templates` in `.obsidian/templates.json`. Use *Insert template* in a new note. |
| Diagrams | **Mermaid** — built in | Every concept note's *Architecture* section. Redraw them in **Excalidraw** (community plugin you already use) when you want to test recall; keep drawings in `Attachments/`. |
| Properties panel, tags, graph colours | Core | `.obsidian/graph.json` colours nodes by folder; `types.json` sets property types. |
| Open tasks across projects, confidence distribution, per-category table, "problems and projects" counts on the dashboard; the automatic sections in weekly reviews | **Dataview** — community plugin | Pre-placed in `.obsidian/plugins/dataview/` (v0.5.70). Turn on community plugins (Settings → Community plugins → *Turn on community plugins*) and enable it; JavaScript queries are enabled in its settings. Without it those panels show as code blocks and everything else still works. |
| Quick creation of concept/problem/project notes with the right template in the right folder | **QuickAdd** — optional | Not required. If you want it: three *Template* choices (Concept → `01 HLD & System Design/<category>`, Problem → `02 System Design Problems`, Project → `04 Projects`) each pointing at the matching template. |
| Computed `next_review` dates in templates | **Templater** — optional | The core Templates plugin cannot do date arithmetic, so `next_review` is set by hand. With Templater, replace the property line with `next_review: <% tp.date.now("YYYY-MM-DD", 7) %>` in the concept template. |
| Task management with due dates | **Tasks** — optional | The Dataview task rollups cover the dashboard's needs; add Tasks only if you want due dates and recurrence on project tasks. |

Everything that matters works with core plugins only. Dataview is the one community plugin worth enabling, because it is what turns the dashboard from "lists" into "numbers".

## Adding and maintaining notes

- **New concept**: create the note in the right category folder, insert *Concept Template*, set `category` (exact category name) and the category tag, fill the six first-pass sections, then add one line to that category's MOC and to [[HLD MOC]]. The Bases tables pick it up from the tag and category automatically.
- **New design problem / project / resource**: same pattern with the matching template; add a line to the section MOC. Resources must link to at least one concept and the concept should link back under *Resources*.
- **Renaming**: use Obsidian's rename (it rewrites links). Note names are unique across the vault on purpose — keep them so.
- **Reviewing**: when you re-explain a note, set `last_reviewed` to today and `next_review` to a date (double the previous gap if it went well; three days if not). The revision queue is driven entirely by those two dates, `status: needs-revision` and `confidence ≤ 2`.
- **Weekly**: create the review note from the template; set `period_start`/`period_end`; the Dataview sections fill themselves.
- **Decisions**: whenever a project changes direction because of a measurement, add a note in `04 Projects/Decisions` from the *Architecture Decision Template* and link it from the project's *Decisions* section. The dashboard counts them — that count is the single best proxy for "engineering judgement developed".
- **Git**: the vault lives inside the `learn-system-design` repository; commit after each study session (`git add HLD && git commit`). `.obsidian/workspace.json` is per-machine state — add it to `.gitignore` if it starts showing up in diffs.

## Daily and weekly workflow

**Daily (45 min):** open [[Engineering Dashboard]] → *Today's three moves* → one first-pass read from *High-priority not started* (or the current week in [[Start Here]]) → write three sentences under *Personal understanding* → set `status`/`confidence` → re-explain one item from the revision queue without opening it → update `last_reviewed`/`next_review`.

**Weekly (90 min):** one timed design problem from [[Problem Bank MOC]] → fill its decision lines → weekly review note → three priorities copied to the dashboard.

**Monthly:** glance at the *Stale* view in `Concepts.base`, prune `gap/*` tags that no longer apply, and move one project forward a milestone or archive it honestly.

## Limitations and manual steps

1. **Open `HLD` as its own vault** (Obsidian → *Open folder as vault*). It is self-contained; links do not cross into `system-design`, which is referenced by path only.
2. **Enable community plugins** once to activate the pre-placed Dataview; Obsidian asks on first open. Until then the Dataview panels render as code.
3. **Property types**: if a date filter ever shows nothing, check Settings → Properties that `next_review`, `last_reviewed`, `last_worked`, `date_started`, `updated`, `period_start`, `period_end` are typed *Date* and `confidence` is *Number*.
4. **Bases inside callouts** are not supported by Obsidian, so the dashboard embeds `.base` views as top-level embeds and folds only the Dataview panels into callouts.
5. **Mermaid rendering** was validated with the Mermaid parser, but Obsidian's renderer is occasionally stricter about labels; if a diagram fails, the error message names the line — quoting the label fixes nearly every case.
6. **The vault does not grade you.** Statuses and confidence are self-reported; the weekly review is where honesty happens.

## Recommended next improvements

- Add an Excalidraw drawing per category MOC (your redraw-from-memory of the chain) and embed it under *How the ideas connect*.
- Once ten or more decisions exist, add a `Decisions` section to the dashboard grouped by project.
- Write a tiny script (or the [[Spaced Repetition Review CLI]] project) that bumps `next_review` based on confidence, so the queue needs no manual dates.
- After the first interview loop, tag the problems you were actually asked with `asked` and add a Bases view for them.
