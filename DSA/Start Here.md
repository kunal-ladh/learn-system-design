---
title: Start Here
aliases: [Beginner Guide, Reading Guide, How to read this vault]
type: guide
category: Guides
tags: [guide]
summary: "How to use this vault for SDE-2 DSA prep: what a topic note and a problem note are, how to read one, the learning loop the properties track, the eight-week order, and what to skip."
created: 2026-11-04
updated: 2026-11-04
---

# Start here — a reading guide for the DSA vault

> [!abstract] The short version
> Twenty-nine topic notes to *read*, one hundred and fifty problem notes to *solve*, three Java refresher notes to keep open, and five plan notes that tell you what to do each day. The order is [[8-Week DSA Plan]]; the daily list is on the [[DSA Dashboard]]; the schedule that makes it stick is [[Spaced Repetition Protocol]].

**Map:** [[DSA Dashboard]] · [[Topics MOC]] · [[NeetCode 150 MOC]] · [[Workspace Guide]]

## What is in the vault

| Folder | What | How many |
|---|---|---|
| `01 Topics` | reading-style deep dives, one per topic, from zero | 29 (6 marked `scared`: Tries, Graphs, Advanced Graphs, Union-Find, DP, 2-D DP — plus the DP catalogue and the two stretch structures, Segment and Fenwick trees) |
| `02 NeetCode 150` | one note per NeetCode problem, Java solution included, grouped by category | 150 |
| `03 Java Refresher` | the collections cheat sheet, the pattern templates, the pitfalls | 3 |
| `04 Plans` | the 8-week plan, spaced repetition, mistakes log, interview-day protocol, pattern table | 5 |
| `05 Reviews` | your weekly review notes (from the template) | grows weekly |
| `Excalidraw` | editable canvases: topic and problem maps, every diagram, every data-structure trace | 455 |

## How to read a topic note (30–40 minutes)

1. Answer the **Predict before you read** callout in your head (or on paper) before reading further. This is the step that makes the rest stick.
2. Read **Why this exists** and **The idea from zero** slowly; the pictures (Excalidraw canvases, Mermaid diagrams) are the explanation, not decoration — open the canvas and trace it with the mouse.
3. Read **The mechanism** and **Java template**; type the template once yourself without looking.
4. Skim **The canonical shapes**, then do the **Worked example** by hand.
5. Read **Pitfalls** and **Spot the bug**; find the bug before opening the fix.
6. Write three sentences under **Personal understanding** without scrolling up; tick what you honestly can in the mastery checklist; set `status: reading` and `confidence: 2`.

The six scared topics get this treatment twice: once as above, and once more two days later as a 20-minute "explain it without the note" session before the first problem.

## How to solve a problem note (25 minutes + 10)

1. Read only **Problem** and **Clarifying questions**. Start a 25-minute timer. Solve in your editor.
2. If solved: compare with **Java solution** and **Edge cases**; read **Interview script** once aloud.
3. If stuck at 25 minutes: read **The key insight** only, retry for 10 minutes; then the full solution.
4. Either way: fill the **Attempt log** row, set `status`, `attempts`, `last_solved`, `time_minutes`, `confidence`, `next_review` ([[Spaced Repetition Protocol]]), and add a row to the [[Mistakes Log]] if it failed.

## The learning loop the properties track

| Stage | What it looks like | Property |
|---|---|---|
| Discover | know the term and its category | `status: not-started` |
| Learn | topic read, three sentences written | `status: reading`, `confidence: 2` |
| Apply | solved a problem of the shape without hints | problem `status: solved`, topic `status: understood`, `confidence: 3` |
| Retain | solved again at +3 and +7 days | `confidence: 4` |
| Master | solved at +21 days, can explain the variant | `status: mastered`, `confidence: 5` |
| Revisit | failed a review or a mock | `status: needs-revision`, lower `confidence` honestly |

## The eight-week order

[[8-Week DSA Plan]] is the full table. In one line: W1 arrays/hashing, two pointers, sliding window, stack → W2 binary search, sorting, strings, linked lists, heaps → W3 trees, tries, backtracking → W4 graphs, advanced graphs, union-find → W5 1-D DP, greedy → W6 2-D DP, intervals → W7 math, bits, segment/Fenwick trees → W8 mocks and revision. Six weeks only? Do W1, W2, W3, W4, W5 and W8, and read the 2-D DP note in W5.

## What to skip for now

- [[Segment Trees]] and [[Fenwick Trees]] until week 7; they are rarely asked at SDE-2 — read them because you said you never have, not because the loop needs them.
- Hard problems beyond one per week.
- Any variant marked "mention, do not implement" in a note.

## A daily routine that fits around a job

Morning 20 minutes of topic reading, lunch 25 minutes of one timed problem, evening 30 minutes of a second problem or the revision queue, Sunday 90 minutes for the weekly review and a mock. The [[8-Week DSA Plan]] has the full routine and the rules that keep it alive.

## Related
- [[Workspace Guide]] — how the dashboard, Bases and templates work
- [[Plugins to Install]] — Excalidraw and Dataview, ten minutes once
- [[Java Refresher MOC]] — read before week 1
