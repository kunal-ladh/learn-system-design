---
title: Plugins to Install
aliases: [Plugins, Obsidian plugins, Setup checklist]
type: guide
tags: [guide, setup]
summary: "The complete list of Obsidian plugins this vault uses — which are core, which are community, which are optional — with the install steps and the settings that matter, as a checklist."
created: 2026-11-03
updated: 2026-11-03
---

# Plugins to Install

> [!abstract] Ten minutes, once
> Two community plugins carry everything visual and numeric in this vault: **Excalidraw** (244 drawings — every map and every diagram has an editable canvas) and **Dataview** (the numbers on the dashboard). Everything else is a core plugin that only needs to be switched on. Work through the checklist top to bottom; the last section tells you how to confirm each one is working.

**Map:** [[Engineering Dashboard]] · [[Workspace Guide]] · [[HLD Vault Status]]

## Checklist

- [ ] Settings → **Community plugins** → *Turn on community plugins* (one-time confirmation)
- [ ] Install and enable **Excalidraw**
- [ ] Enable **Dataview** (already in the vault folder, only needs the toggle)
- [ ] Confirm the core plugins below are on
- [ ] Settings → Appearance → CSS snippets → `hld-workspace` is enabled
- [ ] Run the three checks at the bottom

## Required: community plugins

| Plugin | Why the vault needs it | Install | Settings that matter |
|---|---|---|---|
| **Excalidraw** (by Zsolt Viczián) | Renders the 244 `.excalidraw.md` files in `Excalidraw/`: the eleven category concept maps, the HLD category map and one canvas per diagram in the concept, problem, project and guide notes. Without it the embeds show as raw text and the "Open in Excalidraw" links open a markdown file. | Settings → Community plugins → *Browse* → search **Excalidraw** → *Install* → *Enable*. Or copy `.obsidian/plugins/obsidian-excalidraw-plugin/` from your `system-design` vault into this vault's `.obsidian/plugins/` and enable it. | *Excalidraw folder*: `Excalidraw` (so new drawings land beside the generated ones). *Links → Ctrl/Cmd-click to open links* on (every box in the maps links to its note). *Appearance → Match Obsidian theme* if you use dark mode. Leave *Compress Excalidraw JSON* on; the plugin rewrites the files on first save, which is expected. |
| **Dataview** | The folded "numbers" panels on [[Engineering Dashboard]] (confidence distribution, per-category table, open tasks, problems-and-projects counts), the automatic sections in weekly reviews and the mock-score trend table. Everything else uses core Bases. | Already in `.obsidian/plugins/dataview/` (v0.5.70). Settings → Community plugins → toggle **Dataview** on. If it is missing, *Browse* → **Dataview** → *Install*. | *Enable JavaScript queries* and *Enable inline JavaScript queries* on (the dashboard panels are `dataviewjs`). |

## Required: core plugins (switch on, nothing to install)

| Core plugin | Used for | Where |
|---|---|---|
| **Bases** (Obsidian 1.9 or newer) | Every table: the nine `.base` files in `00 Dashboard/` embedded on the dashboard, the MOCs and the interview notes | Settings → Core plugins → Bases. Date filters such as `next_review <= today()` rely on `.obsidian/types.json`, which already types the properties. |
| **Templates** | The seven templates in `Templates/` (`{{title}}`, `{{date:YYYY-MM-DD}}`) | Settings → Core plugins → Templates; the folder is preset in `.obsidian/templates.json`. |
| **Properties view** | Editing `status`, `confidence`, `next_review` in the side panel | Settings → Core plugins → Properties view |
| **Bookmarks** | The pinned entry points in `.obsidian/bookmarks.json` (Dashboard, Start Here, Learn / Design and build / Interview / Review groups) | Settings → Core plugins → Bookmarks |
| **Outline** | Jumping around the long MOCs and the dashboard | Settings → Core plugins → Outline |
| **Backlinks** and **Graph view** | Seeing which problems pull a concept in; `.obsidian/graph.json` colours nodes by folder and greys the `Excalidraw/` drawings | Settings → Core plugins |
| **Page preview** | Hovering any wikilink shows the note's abstract — the fastest way to revise | Settings → Core plugins → Page preview |
| **Search**, **Tags view**, **Quick switcher** | Finding notes by `#concept`, `#problem`, `#resource`, `#excalidraw` | On by default |

Mermaid is built into Obsidian; no plugin is needed for the diagrams that sit in the notes themselves.

## Optional

| Plugin | When to add it |
|---|---|
| **QuickAdd** | When creating notes by hand becomes tedious: three *Template* choices (Concept → `01 HLD & System Design/<category>`, Problem → `02 System Design Problems`, Project → `04 Projects`), each pointing at the matching template. |
| **Templater** | If you want `next_review` computed on creation: replace the property line in the concept template with `next_review: <% tp.date.now("YYYY-MM-DD", 7) %>`. The core Templates plugin cannot do date arithmetic. |
| **Tasks** | Only if you want due dates and recurrence on project tasks; the Dataview rollups already cover the dashboard. |
| **Spaced Repetition** | An alternative to setting `next_review` by hand — but [[Spaced Repetition Review CLI]] is the planned route, because it keeps the vault's own properties as the source of truth. |

Not needed: any Mermaid plugin (built in), any CSS theme (the `hld-workspace` snippet is enough), Calendar or Periodic Notes (weekly reviews are created from the template).

## Check that everything works

1. **Excalidraw** — open [[Caching MOC]]: the "Excalidraw canvas" callout under the map should show a drawing with five dashed rooms and a code box. Ctrl/Cmd-click the `Cache-Aside` box; the note opens. Then open any concept note, for example [[Cache Stampedes]], expand the folded "Excalidraw version" callout under its Architecture diagram and click *Open in Excalidraw*.
2. **Dataview** — on [[Engineering Dashboard]], expand "Confidence at a glance" (section 1): it should show a histogram-style table, not a code block.
3. **Bases** — the Concepts table at the top of the dashboard shows rows with `status`, `confidence` and `next_review` columns, and the revision queue filters to notes due today or earlier.

If a drawing opens as text, the plugin is not enabled (or the file is in a folder the plugin does not watch — it watches the whole vault by default). If the graph looks crowded, filter it with `-path:Excalidraw` or `-tag:#excalidraw`.
