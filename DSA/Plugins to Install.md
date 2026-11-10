---
title: Plugins to Install
aliases: [Plugins, Obsidian plugins, Setup checklist]
type: guide
category: Guides
tags: [guide, setup]
summary: "The plugins this vault uses — Excalidraw and Dataview as community plugins, Bases/Templates/Properties/Bookmarks as core — with the install steps, the settings that matter and three checks that everything works; the same setup as the HLD vault."
created: 2026-11-04
updated: 2026-11-04
---

# Plugins to Install

> [!abstract] Five minutes if you set up the HLD vault already
> Same two community plugins: **Excalidraw** (every map, diagram and data-structure trace is an editable canvas in `Excalidraw/`) and **Dataview** (the numbers panels on the dashboard and the automatic sections of weekly reviews). Everything else is a core plugin that only needs to be on. `community-plugins.json` already lists both, so once installed they are enabled.

**Map:** [[DSA Dashboard]] · [[Workspace Guide]]

## Checklist

- [ ] Settings → **Community plugins** → *Turn on community plugins*
- [ ] Install **Excalidraw** (Browse → search Excalidraw → Install → Enable) — or copy `.obsidian/plugins/obsidian-excalidraw-plugin/` from the HLD vault into this vault's `.obsidian/plugins/`
- [ ] Install **Dataview** the same way (or copy `.obsidian/plugins/dataview/` from the HLD vault)
- [ ] Settings → Appearance → CSS snippets → `dsa-workspace` is enabled
- [ ] Core plugins on: Bases, Templates, Properties view, Bookmarks, Outline, Backlinks, Graph view, Page preview
- [ ] Run the three checks below

## Settings that matter

| Plugin | Setting |
|---|---|
| Excalidraw | *Excalidraw folder* = `Excalidraw`; *Links → Ctrl/Cmd-click to open links* on; *Match Obsidian theme* if you use dark mode; leave *Compress JSON* on (the plugin rewrites files on first save, expected) |
| Dataview | *Enable JavaScript queries* and *Enable inline JavaScript queries* on |
| Templates (core) | folder `Templates`, date format `YYYY-MM-DD` (preset in `.obsidian/templates.json`) |

## Check that everything works

1. **Excalidraw** — open [[Topics MOC]] and click *Topic map*: a canvas with the topics arranged by week; Ctrl/Cmd-click any box to open its note. Then open [[Union-Find]] and expand the first canvas callout: a DSU forest with arrows pointing to parents.
2. **Dataview** — on the [[DSA Dashboard]], expand *Problems by status and difficulty*: a table with bars, not a code block.
3. **Bases** — the *Problems not yet started* table at the top of the dashboard lists 150 rows sorted by week and NeetCode order.

If a drawing opens as text, the Excalidraw plugin is not enabled. If the graph view is crowded, filter it with `-path:Excalidraw`.

## Related
- [[Workspace Guide]] · [[Start Here]]
