---
title: Topics MOC
aliases: [Topics, DSA Topics, Topic Index]
type: moc
tags: [moc]
cssclasses: [wide]
summary: "Every topic deep dive in the vault in reading order by week, with the six scared topics marked, one-line summaries, and the live table of their status."
created: 2026-11-04
updated: 2026-11-04
---

# Topics MOC

**Map:** [[DSA Dashboard]] · [[NeetCode 150 MOC]] · [[Java Refresher MOC]] · [[Plans MOC]] · Canvas: [[DSA Topic Map.excalidraw|Topic map]]


> [!abstract] How to read a topic note
> Each note is written to be *read*, start to finish, in one sitting of 30–40 minutes: the idea from zero, the mechanism with pictures, the Java template, the canonical problem shapes, the pitfalls, and an interview script. The six notes marked **scared** assume no prior exposure and are the longest. Read the week's topics before the week's problems ([[8-Week DSA Plan]]).

## Reading order by week


### Week 1

- [[Complexity Analysis]] — Big-O is the habit of counting how work grows with input size while ignoring constants — it lets you read the constraints line of a problem, pick the target complexity before writing code, price every Java library call you lean on, and defend the amortised claims (ArrayList growth, path compression) that interviewers probe.
- [[Recursion]] — Recursion solves a problem by calling itself on a strictly smaller input and combining the answer — the whole discipline is to write the contract, the base case and the recursive case, trust the smaller call, and know what the call stack costs you (O(depth) memory, no tail-call optimisation in Java, a crash somewhere past ten thousand frames).
- [[Problem-Solving Framework]] — A fixed six-stage protocol for a 45-minute coding round — understand, examples, brute force, optimise, code, test — with minute budgets, the questions to ask at each stage, a cue-to-pattern table for the optimise step, and the moves that get you unstuck, so that nerves never decide what you do next.
- [[Arrays & Hashing]] — The week-1 toolbox for arrays: count with an index or a map, precompute prefix sums so range questions become subtraction, use the array itself as a hash table when O(1) space is demanded, encode with length prefixes, bucket by frequency to beat the sort, and walk only from sequence starts — six tricks that turn O(n²) scans into one or two linear passes.
- [[Hashing]] — A hash table turns a key into a bucket index so that lookups cost O(1) on average — Java's HashMap does it with hashCode spread into a power-of-two array, chaining that treeifies at 8, and doubling at load factor 0.75 — and knowing the machinery tells you how to design keys (immutable, with hashCode and equals agreeing), when lookups degrade, and when hashing is the trick that makes a problem linear.
- [[Two Pointers]] — Two pointers replaces the O(n²) enumeration of pairs with an O(n) walk in which each step provably discards a whole row or column of candidates — from opposite ends on sorted or monotone data, in the same direction to rewrite an array in place, or at different speeds to find cycles and middles of linked lists.
- [[Sliding Window]] — A sliding window maintains a contiguous range [l, r] and a small summary of what is inside it (counts, a sum, a max), growing r one step at a time and shrinking l only when a monotone validity condition breaks, so that every subarray or substring question that asks for the longest, shortest or best contiguous piece costs O(n) instead of O(n²).
- [[Stack & Monotonic Stack]] — A stack answers anything with nested or last-in-first-out structure — matching brackets, evaluating postfix, tracking the minimum — and a monotonic stack, which pops every element the newcomer dominates before pushing it, settles each element exactly once and so answers next-greater, next-smaller, daily-temperatures and largest-rectangle questions in O(n).

### Week 2

- [[Binary Search]] — Binary search halves a range on every step wherever a predicate flips exactly once from false to true — on a sorted array, on a rotated one, on a flattened matrix, or on the space of candidate answers — and the three invariant-based templates (exact, first-true, last-true) with mid = lo + (hi − lo) / 2 remove the off-by-one and infinite-loop bugs that make it the most mis-implemented O(log n) algorithm.
- [[Searching]] — A routing note for the question 'where is x, does x exist, what is the first x such that' — linear scan when nothing is structured, binary search when a predicate is monotone, ternary when a function is unimodal, exponential when the array is unbounded, the staircase walk when a matrix is sorted both ways, BFS or DFS when the space is a graph of states, backtracking when it is a tree of decisions, and TreeMap floor and ceiling when the data keeps changing.
- [[Sorting]] — Sorting is the n log n preprocessing step that turns pair problems into two-pointer walks, interval problems into sweeps and greedy problems into one pass — know merge sort (and its inversion count), quicksort and quickselect, heap sort, counting and bucket sort, what Arrays.sort actually runs (dual-pivot quicksort for primitives, stable TimSort for objects), and the comparator mistakes that compile but return wrong order.
- [[Strings]] — Strings in Java are immutable arrays of UTF-16 units, so every interview string problem is really an array problem with three habits attached: build output in a StringBuilder, treat characters as small integers (c - 'a'), and recognise the four shapes — palindrome (two pointers from the ends or the centre), anagram (a count key), substring search (sliding window or KMP) and encoding (length prefixes) — before reaching for anything cleverer.
- [[Linked Lists]] — A linked list trades O(1) indexing for O(1) splicing — every interview problem on it is pointer surgery built from five moves (dummy head, in-place reversal, fast/slow with a gap, merge, cut-and-rejoin) and the discipline of saving a pointer before you overwrite it.
- [[Heaps & Priority Queues]] — A binary heap is a complete binary tree stored in a plain array where every parent beats its children, which makes the extreme element O(1) to read and O(log n) to remove or insert — the structure behind every 'k largest', 'running median', 'merge k streams' and 'schedule by priority' problem, and behind Java's PriorityQueue, whose API has three traps worth knowing by name.

### Week 3

- [[Trees]] — Every binary-tree problem is one recursive function whose shape you choose deliberately — what it returns to its parent, what it records globally, and what it passes down — plus BFS by frozen levels; BSTs add the inorder-is-sorted invariant, and the hard problems (diameter, max path sum, construction, serialisation) all reduce to 'return one side up, record both sides here'.
- [[Tries]] **scared** — A trie stores a set of strings as a tree in which each edge is one character, so every shared prefix is stored once and insert, lookup and prefix checks cost O(length of the word) no matter how many words are stored; it beats a HashSet exactly when the question is about prefixes, wildcards or many words searched at once.
- [[Backtracking]] — Backtracking is depth-first search over the tree of partial decisions — choose, explore, unchoose — and every subsets / permutations / combinations / grid-path / constraint-placement problem is the same template with a different rule for which children a node has and when to stop early; it is the right tool when the answer is 'all solutions' or the search space is small enough to enumerate (n ≤ ~20), and the wrong one when only a count or optimum is wanted and states repeat (that is DP).

### Week 4

- [[Graphs]] **scared** — A graph is nodes plus edges, stored as an adjacency list or implied by a grid; almost every interview graph problem is one of two traversals (BFS for fewest steps and layers, DFS for reachability and structure) plus a bookkeeping twist — a visited set, a parent, three colours, or in-degrees — and the twist is what the problem is really testing.
- [[Advanced Graphs]] **scared** — Once edges carry weights, BFS stops being the shortest-path algorithm and a small family takes over — Dijkstra (non-negative weights, a priority queue instead of a queue), Bellman-Ford (negative weights or a bound on the number of edges), Prim and Kruskal (cheapest way to connect everything), Hierholzer (use every edge once) — and the interview skill is picking the right one from the statement in under a minute.
- [[Union-Find]] **scared** — Union-Find (disjoint set union) keeps track of which elements belong to the same group using only a parent array — find follows parents to the group's representative, union links two representatives — and with path compression plus union by rank both operations cost effectively constant amortised time, which makes it the tool for connectivity questions where edges arrive one at a time.

### Week 5

- [[Dynamic Programming]] **scared** — Dynamic programming is recursion with a notebook: define what f(i) means, write f(i) in terms of smaller i, answer the smallest i directly, and compute in an order that never asks for an unanswered value — memoisation does it top-down with a cache, tabulation bottom-up with a loop — and every 1-D problem is one of a handful of shapes once you can name the state.
- [[Greedy]] — A greedy algorithm commits to the locally best choice and never revisits it, which is correct only when an exchange argument shows any optimal solution can be rewritten to agree with that choice at no cost — the eight NeetCode greedy problems are eight such arguments (frontier, reset, smallest-first, last-index, min/max open count), and coin change is the counter-example that sends you to DP.

### Week 6

- [[2-D Dynamic Programming]] **scared** — 2-D DP is the same four questions as 1-D DP with a state of two integers — a position in each of two strings, a row and a column, an item and a capacity, the two ends of an interval, or an index and a mode — and the five table shapes that follow (grid paths, two-sequence alignment, knapsack, interval, state machine) cover every NeetCode 2-D problem; the work is choosing the shape and then filling the table in an order where every cell reads finished cells.
- [[DP Pattern Catalogue]] **scared** — One page that maps every DP problem in NeetCode 150 to one of twelve patterns — the cue in the statement, the state in one sentence, the transition in one line — plus a decision tree for picking the pattern in the first minute; read it before a DP problem, and after every DP problem to file what you just solved.
- [[Intervals]] — Every interval problem is 'sort by one endpoint, then sweep once carrying the smallest state that answers the question' — a running end for merging, the earliest finish for scheduling, a min-heap of end times for rooms, a heap of sizes for queries — and the only real decision is which endpoint to sort by, which the exchange argument settles.

### Week 7

- [[Math & Geometry]] — A grab-bag category with three real skills — index arithmetic on matrices (rotate by transpose-then-reverse, spiral by four shrinking boundaries, zero rows and columns using the first row and column as flags), digit-and-carry loops on numbers held as arrays or strings (plus one, multiply strings, happy-number cycles), and binary exponentiation with the overflow discipline every int problem needs — eight NeetCode problems that need no algorithmic idea beyond careful bookkeeping.
- [[Bit Manipulation]] — An int is 32 switches: the six operators read and flip them in O(1), XOR cancels pairs so the odd one out and the missing number fall out of a single pass, n & (n - 1) and n & -n isolate the lowest set bit, a mask of n bits enumerates every subset, and two's complement explains every negative-number surprise — seven NeetCode problems plus a toolkit that leaks into DP, Fenwick trees, backtracking and hashing.
- [[Segment Trees]] **scared** — A segment tree stores an array as a binary tree of halves where every node holds the merged value (sum, min, max, gcd) of its own range, so any range query is assembled from O(log n) stored nodes and a point update refreshes only the O(log n) ancestors of one leaf — the tool when an array keeps changing while range questions keep coming, and lazy propagation extends it to range updates.
- [[Fenwick Trees]] **scared** — A Fenwick tree (binary indexed tree) is an array of n + 1 numbers where index i stores the sum of the lowbit(i) elements ending at i, so a prefix sum is read by repeatedly stripping the lowest set bit and a point update by repeatedly adding it — O(log n) for both in two loops of three lines, the lightweight sibling of the segment tree for sums, counts and anything else with an inverse.

## Live status

![[Topics.base#All]]

### The scared six (plus the two stretch structures)

![[Topics.base#Scared six]]

## Related
- [[NeetCode 150 MOC]] — the problems each topic links to
- [[Pattern Recognition Table]] — statement cue → topic
- [[DSA Dashboard]]
