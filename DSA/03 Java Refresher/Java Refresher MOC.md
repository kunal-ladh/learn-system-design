---
title: Java Refresher MOC
aliases: [Java Refresher, Java for DSA]
type: moc
tags: [moc]
cssclasses: [wide]
summary: "The three Java refresher notes — the collections cheat sheet of every method you forget, the pattern template vault, and the interview pitfalls — with when to read each."
created: 2026-11-04
updated: 2026-11-04
---

# Java Refresher MOC

**Map:** [[DSA Dashboard]] · [[Topics MOC]] · [[NeetCode 150 MOC]]


> [!abstract] Read these before week 1
> You said you forget `Map`/`Set` methods — `get`, `put`, `getOrDefault`, `computeIfAbsent` — and that is exactly what the cheat sheet is for. Keep it open in a split pane while solving; after two weeks you will stop needing it.

- [[Java Collections Cheat Sheet]] — Every Java 21 collection, string and utility method you reach for in a coding interview — Map, Set, List, ArrayDeque, PriorityQueue, String, StringBuilder, Character, Arrays, Collections, Math, Integer — one table per class with signature, return value, the gotcha and a one-line example, plus the 25 one-liners you keep forgetting and a decision chart for which structure to pick.
- [[DSA Patterns in Java]] — The template vault: one compiling Java 21 block per interview pattern — two pointers, sliding window, prefix sums, monotonic stack and deque, the three binary-search templates, fast/slow pointers, list reversal, BFS and DFS, topological sort, Union-Find, Dijkstra, Prim, Kruskal, Trie, heaps, backtracking, 1-D and 2-D DP, knapsack, LIS, intervals, bit tricks — each with when to use it, its complexity and the NeetCode problems it solves.
- [[Java Pitfalls in Interviews]] — The twenty-three ways correct-looking Java silently returns the wrong answer under interview conditions — Integer cache, int overflow, remove(int) vs remove(Object), ConcurrentModificationException, PriorityQueue iteration order, substring cost, recursion depth, subtraction comparators, negative modulo and more — each as a failing snippet with the one-line fix, grouped by numbers, collections, strings, recursion, sorting and iteration.

## When to open which

| Situation | Open |
|---|---|
| "What was the method to get a default value from a map?" | [[Java Collections Cheat Sheet]] |
| "How do I write BFS / Dijkstra / union-find quickly?" | [[DSA Patterns in Java]] |
| "My code is right but the answer is wrong" | [[Java Pitfalls in Interviews]] |
| Before a mock | all three, 15 minutes |

## Related
- [[Topics MOC]] · [[Plans MOC]] · [[DSA Dashboard]]
