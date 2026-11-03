---
title: instructions
aliases: [Excalidraw instructions, Diagram instructions, Canvas conventions]
type: guide
tags: [guide, excalidraw, instructions]
summary: "Standing instructions for making and maintaining the Excalidraw canvases in this vault — the excalidraw-diagram skill's method, the vault's own conventions on top of it, how the 244 existing drawings were produced, and how to add or regenerate one. For Kunal and for any Claude session (Cowork or Claude Code) working in this repo."
created: 2026-11-03
updated: 2026-11-03
---

# instructions

> [!abstract] What this file is
> Everything a future session needs to make a diagram in this vault look and behave like the ones already here. Section 1 is the skill's method (condensed from `.claude/skills/excalidraw-diagram/SKILL.md`), section 2 is what this vault adds on top, section 3 is how the existing drawings were built and how to add or regenerate one, section 4 is the quality bar. Read the skill's full `SKILL.md`, `references/color-palette.md` and `references/element-templates.md` before drawing anything by hand.

**Related:** [[Plugins to Install]] · [[Workspace Guide]] · [[HLD Vault Status]] · [[HLD Category Map.excalidraw]]

## 0. The skill

The repo uses Cole Medin's **excalidraw-diagram** skill (`https://github.com/coleam00/excalidraw-diagram-skill`). Install it at the repo root so Claude Code picks it up:

```bash
cd ~/code-repository/learn-system-design
git clone https://github.com/coleam00/excalidraw-diagram-skill.git
mkdir -p .claude/skills
cp -r excalidraw-diagram-skill .claude/skills/excalidraw-diagram
cd .claude/skills/excalidraw-diagram/references && uv sync && uv run playwright install chromium
```

The last line sets up the renderer (`uv run python render_excalidraw.py <file.excalidraw>` → PNG next to the file). Its HTML template imports `@excalidraw/excalidraw` from esm.sh; if that host is blocked, bundle the package from npm with esbuild and serve it locally instead (that is what the Cowork session did).

## 1. The method (from the skill)

**Diagrams argue, they do not display.** The structure alone, with the text removed, should still say what the diagram is about (isomorphism test), and a reader should learn something concrete from it (education test).

1. **Assess depth first.** Conceptual diagrams (mental models, "where ideas sit") can use abstract shapes. Technical diagrams (a real system, protocol, API) must show what things actually look like — real payloads, real event and endpoint names, real numbers — as *evidence artifacts*: dark boxes with monospace text for code and JSON, timelines for sequences, nested rectangles for UI.
2. **Research before drawing anything technical.** Look up the actual formats and names; no "Service A → Service B".
3. **Map each concept to a pattern** and vary them: fan-out for one-to-many, convergence for many-to-one, tree (lines + free text) for hierarchy, timeline (line + dots + labels) for sequences, cycle for loops, assembly line for transformations, side-by-side for comparisons, gap/break for phases. No uniform card grids.
4. **Multi-zoom.** A summary flow (level 1), labelled rooms that group related parts (level 2), and concrete detail inside the rooms (level 3).
5. **Containers only when they carry meaning.** Titles, annotations and details are free-floating text; rectangles for things arrows connect to, diamonds for decisions, ellipses for start/end, small dots as markers. Aim for fewer than 30 % of text elements inside containers.
6. **Modern aesthetics.** `roughness: 0`, `opacity: 100`, `strokeWidth` 2 for shapes and main arrows, 1 for structure lines, `fontFamily: 3` (Cascadia), `fontSize` 16 in boxes.
7. **Build large diagrams section by section**, with descriptive element ids, and fix cross-section bindings as you go.
8. **Render → view → fix, in a loop, until you would show it without caveats.** Check: no clipped or overlapping text, arrows that land on the right element without crossing others, even spacing, balanced composition, hero elements dominant.

Colours come only from `references/color-palette.md`. The ones this vault uses:

| Meaning | Fill | Stroke | Used for |
|---|---|---|---|
| Start / entry | `#fed7aa` | `#c2410c` | the first concept to read; clients and users in architecture diagrams |
| Concept (neutral) | `#93c5fd` | `#1e3a5f` | ordinary nodes |
| Hub / data store | `#3b82f6` / `#60a5fa` | `#1e3a5f` | the concept everything hangs off (white text); cylinders in architecture diagrams |
| Decision / trade-off | `#fef3c7` | `#b45309` | "X vs Y" concepts, diamonds |
| Failure mode | `#fee2e2` | `#dc2626` | stampedes, retry storms, dead letters |
| Danger | `#fecaca` | `#b91c1c` | split-brain |
| Destination | `#a7f3d0` | `#047857` | where a chain lands |
| Evidence artifact | `#1e293b` bg | `#22c55e` text | code, JSON, arithmetic |
| Title / section title | `#1e40af` | — | 28 px title, 20 px room titles |
| Subtitle / reading order | `#3b82f6` | — | 14 px |
| Body / hints / details | `#64748b` | — | 14 px |

## 2. What this vault adds

**Where drawings live.** `Excalidraw/` at the vault root, in the Obsidian Excalidraw plugin's markdown format (`X.excalidraw.md`, frontmatter `excalidraw-plugin: parsed`, uncompressed JSON under `## Drawing`; the plugin may compress it on first save — that is fine).

| Folder | Contents | Count |
|---|---|---|
| `Excalidraw/Maps/` | 11 category concept maps + `HLD Category Map` — hand-designed | 12 |
| `Excalidraw/Concepts/` | one canvas per concept-note diagram, plus the Knowledge Graph chains | 172 |
| `Excalidraw/Problems/` | one per problem note + Problem Bank MOC | 29 |
| `Excalidraw/Projects/` | one per project note + Projects MOC | 17 |
| `Excalidraw/Guides/` | dashboard, Start Here, interview notes, Resources and Reviews MOCs | 14 |
| `Excalidraw/Redraws/` | your own redraw-from-memory attempts (create when needed) | — |

**Naming.** A drawing is named after its note: `Cache-Aside.excalidraw.md`. A note with several diagrams gets ` — <heading>` appended, and a repeated heading gets ` (2)`. Never create a drawing whose basename equals a note's basename without the `.excalidraw` part — wikilinks resolve by basename.

**Linking boxes to notes.** Any box whose text is exactly a note title carries `"link": "[[Note Title]]"` on the shape element, and the link is repeated under `## Element Links` in the markdown so Obsidian's backlinks and graph see it. Keep box labels identical to note titles; wrap long titles across lines inside the box rather than shortening them. Arrow labels are text bound to the arrow (`containerId` = the arrow's id), which Excalidraw draws with a background so the line does not strike through the words.

**Embedding.** Every drawing is embedded in the note it belongs to, directly under the Mermaid block it corresponds to:

```markdown
> [!example]- Excalidraw version — the same diagram as an editable canvas
> ![[Cache-Aside.excalidraw]]
> [[Cache-Aside.excalidraw|Open in Excalidraw]] — needs the Excalidraw plugin (see [[Plugins to Install]]).
```

Folded (`-`) in notes, because the Mermaid diagram above it is already visible; unfolded (`+`) on the eleven category MOCs and the HLD MOC, where the canvas is a richer map than the Mermaid one. The Mermaid block is always the reference; the canvas is the editable, annotatable copy.

**The category maps** (`Excalidraw/Maps/`) follow a fixed anatomy, so a new category would too: a 28 px title and one-line subtitle stating the argument; a reading-order timeline (line + dots + the room names); dashed rooms with the title and a two-line detail *above* each frame; inside each room the concepts laid out in the pattern that fits them (a hub with lanes, a snake for a six-step chain, a convergence for three signals into one, side-by-side for a pair); one evidence box per map (the cache-aside loop, the quorum arithmetic, an `OrderPlaced` event, a JWT shape, an inverted index, SLO burn-rate maths…); a legend of the colours actually used; and the hint line about Ctrl/Cmd-click. Cross-room arrows are straight when nothing is in the way and otherwise routed orthogonally around boxes and text.

**Generated canvases** (everything outside `Maps/`) keep the note's Mermaid layout — node positions, cluster frames, edge paths and labels come from Mermaid's own render — and only restyle it: sources become orange, cylinders become the darker blue with a drum top, diamonds amber, subgraphs dashed rooms with the title above, `~~~` links disappear, and the note's `summary` becomes the subtitle. They are honest conversions, not redesigned arguments; redesign one by hand when it matters (see §3).

**Graph and bookmarks.** `.obsidian/graph.json` greys `path:Excalidraw`; filter the graph with `-path:Excalidraw` to hide drawings. The HLD map is bookmarked under *Learn*.

## 3. Making and regenerating drawings

**Adding a diagram to a new note.** Write the Mermaid block first (it renders without any plugin and is what the validator checks). Then either convert it with the pipeline below or draw the canvas by hand in Excalidraw following §1–2, save it in the matching `Excalidraw/` folder with the note's name, and add the folded callout under the Mermaid block.

**Hand-crafting a canvas (the skill's own workflow).** Use it for anything that should *teach*: a core interview problem, a redesigned category map, a sequence you keep getting wrong. Plan the rooms and patterns first, build the JSON one section at a time with readable ids (`gateway_rect`, `arrow_gateway_to_redis`), bind arrows on both ends, then render and fix until it passes §4. Save as `.excalidraw.md` in the plugin format (copy the frontmatter and section skeleton from any file in `Excalidraw/`), with `## Text Elements` listing every text element as `text ^id` and `## Element Links` listing every `id: [[Note]]`. Ids are 8 characters.

**The conversion pipeline (how the 232 generated canvases were made).** Three scripts, run from a scratch folder, with `mmdc` (Mermaid CLI), Python 3 and the local renderer:

1. *Convert* — for each Mermaid block in the vault: render it with `mmdc -i block.mmd -o block.svg`; walk the SVG with cumulative `translate()` offsets (subgraphs with their own `direction` are nested `<g class="root">` groups); read node centres, shapes (`rect`, `circle`, `polygon` → diamond/hexagon/subroutine, `outer-path` → cylinder or stadium), label text (`<p>`, `<br>` → newline), cluster rectangles, edge `data-points` (base64 JSON, the dagre polyline) and edge labels; skip edges whose class contains `invisible`. Scale the layout by at least 1.3 — more when monospace labels are wider than Mermaid's — so boxes keep their spacing; draw rooms, shapes, bound text, arrows through the dagre points (`roundness: {"type": 2}`, interior points inside either box dropped so there are no hooks) and arrow-bound labels; colour by role; link boxes whose label is a note title. Sequence diagrams get their own layout (participants, dashed lifelines, horizontal arrows, note boxes), state diagrams are rewritten as `flowchart LR` first, and the quadrant chart is drawn as a 2×2 grid with dots. The gantt chart has no canvas.
2. *Embed* — insert the callouts (idempotent: a note that already contains `.excalidraw]]` is left alone); on the twelve MOCs embed the hand-designed map instead of the converted one.
3. *Validate* — JSON parses, every `containerId` and binding points at an existing element, every link target is a note, no two text-bearing boxes overlap; then render a sample to PNG and look.

To regenerate one drawing, delete it and run the converter for that note; to change the look of all of them, change the palette or sizing once and rerun, then commit only the files whose content changed. The scripts lived in the Cowork session's scratchpad; ask Claude to reproduce them from this description, or keep a copy under `.scripts/excalidraw/` in the repo (not inside the vault's indexed folders).

**Rendering outside Obsidian.** `render_excalidraw.py` renders a plain `.excalidraw` JSON; for a plugin-format file, extract the JSON between the `## Drawing` fences first. Render at device scale 2 and read the PNG — never judge a drawing from its JSON.

## 4. Quality bar before a drawing is committed

- Rooms, patterns and the reading order match what was planned; the eye has one path through the page.
- Every relationship has an arrow; arrows land on the right element, leave from the side that faces the target, and do not cross boxes or text (route them, do not let them pass through).
- No clipped or overlapping text; similar things evenly spaced; no empty voids beside crowded corners.
- Colours only from the table in §1, with a legend of the ones used; evidence artifacts present on technical diagrams and readable at export size.
- Box labels equal note titles, links present on both the element and under `## Element Links`, `## Text Elements` complete, ids unique and 8 characters.
- The note embeds the drawing under its Mermaid block with the standard callout, and `Plugins to Install` still describes how to see it.
- Rendered to PNG and viewed at least once after the last edit.

## 5. Where to go from here

- **Core-problem canvases** — one comprehensive, hand-crafted canvas for each of the ten core interview problems: summary flow on top, rooms for client / edge / services / data, real request and event payloads as evidence boxes.
- **Redraw deck** — blank versions of the eleven category maps (rooms and titles kept, boxes and arrows removed) in `Excalidraw/Redraws/`, one per week, compared against the original afterwards.
- **Plugin first** — none of the canvases are visible until Excalidraw is installed and enabled in this vault; the three checks in [[Plugins to Install]] confirm it.
