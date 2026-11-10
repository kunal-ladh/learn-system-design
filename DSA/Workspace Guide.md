---
title: Workspace Guide
aliases: [How this vault works, Vault machinery]
type: guide
category: Guides
tags: [guide]
summary: "The machinery of this vault: folders, note types and properties, the Bases on the dashboard, the Dataview panels, templates, the Excalidraw canvases, and the conventions shared with the HLD vault."
created: 2026-11-04
updated: 2026-11-04
---

# Workspace Guide

> [!abstract] In one sentence
> The vault is three kinds of content notes (topic, problem, guide) whose frontmatter properties feed native Bases tables on the dashboard, a few Dataview panels for the numbers, templates for new notes, and an Excalidraw folder with an editable canvas for every diagram — the same design as the HLD vault, so nothing here is new if you have used that one.

**Map:** [[DSA Dashboard]] · [[Start Here]]

## Folders

```
DSA/
  00 Dashboard/        DSA Dashboard.md + Problems.base, Topics.base, Revision Queue.base, Reviews.base, Guides.base
  01 Topics/           29 topic notes + Topics MOC
  02 NeetCode 150/     18 category folders, 150 problem notes + NeetCode 150 MOC
  03 Java Refresher/   3 notes + Java Refresher MOC
  04 Plans/            5 notes + Plans MOC
  05 Reviews/          weekly reviews + Reviews MOC
  Excalidraw/          Maps/ (topic map, problem map), Topics/, Problems/, Guides/ — one canvas per diagram and per trace
  Templates/           Problem, Topic, Weekly Review
  Attachments/
  .obsidian/           app, appearance (snippet on), community-plugins, templates, types, graph colours, bookmarks, snippets/dsa-workspace.css
```

## Note types and properties

| type | where | tracked properties | status vocabulary |
|---|---|---|---|
| `topic` | 01 Topics | `category`, `scared`, `week`, `priority`, `status`, `confidence`, `last_reviewed`, `next_review` | not-started · reading · understood · applied · needs-revision |
| `problem` | 02 NeetCode 150 | `lc`, `difficulty`, `category`, `nc_order`, `week`, `patterns`, `attempts`, `last_solved`, `time_minutes`, `leetcode`, plus `status`, `confidence`, `next_review` | not-started · attempted · solved · needs-revision · mastered |
| `guide` | 03, 04, root | none (reference material) | — |
| `moc` | one per folder | none | — |
| `review` | 05 Reviews | `week`, `period_start`, `period_end` | — |

Dates are `YYYY-MM-DD`; `confidence` is 1–5; `tags` carry the type and the category in kebab-case plus pattern tags on problems. `.obsidian/types.json` already types every property so the Bases date filters work.

## The Bases

- **Problems.base** — All (grouped by category with averages), Not started (plan order), By week, In progress, Solved, Solved this week, Needs revision, Hard, Mastered.
- **Topics.base** — All, Scared six, Studying, Not started, Low confidence, Needs revision.
- **Revision Queue.base** — Due now (flagged → due → low confidence → stale, 15 rows), Flagged, Low confidence, Scheduled; mixes topics and problems on purpose.
- **Reviews.base** — Latest, All. **Guides.base** — All.

Open a `.base` file directly to edit cells in place; the embeds on the dashboard (`![[Problems.base#Not started]]`) are the same views.

## Dataview panels

The folded "Numbers" callouts on the dashboard and the automatic sections of weekly reviews use Dataview (`dataviewjs`). Everything else works without the plugin.

## Templates

`Templates/` holds Problem, Topic and Weekly Review templates; the core Templates plugin is pointed at the folder. New problem → new note named `LC <n> <title>` in the right category folder → insert the Problem template → fill `lc`, `difficulty`, `category`, `week`, `nc_order`.

## Excalidraw canvases

Every Mermaid flowchart in a note has an editable Excalidraw twin in a folded callout right under it, every data-structure trace (arrays with pointers, DP tables, trees, tries, graphs with BFS layers, DSU forests, segment and Fenwick trees, bit rows) is a canvas embedded where the text explains it, and the two maps (topics by week, problems by category) link every box to its note. The canvases are generated from the notes; the Mermaid block stays the reference. Layout is done with ELK (layered, orthogonal routing) so arrows do not cross boxes. See [[Plugins to Install]].

## Conventions shared with the HLD vault

Wikilinks only, unique note names, one abstract callout per note, a `**Map:**` line, a Related section, Personal understanding + mastery checklist on every tracked note, Mermaid with quoted labels, Java 21 that compiles standalone. If you add notes, keep the properties above and the Bases keep working.

## Related
- [[Start Here]] · [[Plugins to Install]] · [[DSA Dashboard]]
