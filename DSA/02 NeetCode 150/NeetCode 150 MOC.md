---
title: NeetCode 150 MOC
aliases: [NeetCode 150, Problem Bank, Problems]
type: moc
tags: [moc]
cssclasses: [wide]
summary: "All 150 NeetCode problems as notes, grouped by category in the canonical order with LeetCode number, difficulty and week, plus the live tables of progress."
created: 2026-11-04
updated: 2026-11-04
---

# NeetCode 150 MOC

**Map:** [[DSA Dashboard]] · [[Topics MOC]] · [[8-Week DSA Plan]] · Canvas: [[NeetCode 150 Map.excalidraw|Problem map]]


> [!abstract] One note per problem
> Every note has the same sixteen sections: problem, clarifying questions, brute force, the key insight, the optimal approach with a diagram, a walkthrough, a compiling Java solution, why it works, edge cases, common mistakes, variants, an interview script, related notes, an attempt log and a mastery checklist. Solve first, read second; log every attempt so the [[Spaced Repetition Protocol]] can schedule the revisions.

| Difficulty | Count |
|---|---|

| Easy | 28 |
| Medium | 101 |
| Hard | 21 |

## By category (canonical order)


### Arrays & Hashing — week 1 · pattern note [[Arrays & Hashing]]

| # | Problem | Difficulty | Patterns |
|---|---|---|---|
| 1 | [[LC 217 Contains Duplicate]] | Easy | hash-set |
| 2 | [[LC 242 Valid Anagram]] | Easy | counting, hash-map |
| 3 | [[LC 1 Two Sum]] | Easy | hash-map, complement-lookup |
| 4 | [[LC 49 Group Anagrams]] | Medium | hash-map, canonical-key, counting |
| 5 | [[LC 347 Top K Frequent Elements]] | Medium | hash-map, bucket-sort, heap |
| 6 | [[LC 271 Encode and Decode Strings]] | Medium | length-prefix-encoding, string-parsing |
| 7 | [[LC 238 Product of Array Except Self]] | Medium | prefix-sum, two-pass |
| 8 | [[LC 36 Valid Sudoku]] | Medium | hash-set, matrix, index-mapping |
| 9 | [[LC 128 Longest Consecutive Sequence]] | Medium | hash-set, sequence-start-detection |

### Two Pointers — week 1 · pattern note [[Two Pointers]]

| # | Problem | Difficulty | Patterns |
|---|---|---|---|
| 10 | [[LC 125 Valid Palindrome]] | Easy | two-pointers, opposite-ends |
| 11 | [[LC 167 Two Sum II Input Array Is Sorted]] | Medium | two-pointers, opposite-ends |
| 12 | [[LC 15 3Sum]] | Medium | two-pointers, sort-then-scan, dedupe |
| 13 | [[LC 11 Container With Most Water]] | Medium | two-pointers, opposite-ends, greedy-elimination |
| 14 | [[LC 42 Trapping Rain Water]] | Hard | two-pointers, prefix-max, monotonic-stack |

### Sliding Window — week 1 · pattern note [[Sliding Window]]

| # | Problem | Difficulty | Patterns |
|---|---|---|---|
| 15 | [[LC 121 Best Time to Buy and Sell Stock]] | Easy | sliding-window, running-minimum, kadane |
| 16 | [[LC 3 Longest Substring Without Repeating Characters]] | Medium | sliding-window, last-index, hash-map |
| 17 | [[LC 424 Longest Repeating Character Replacement]] | Medium | sliding-window, max-frequency, never-shrink |
| 18 | [[LC 567 Permutation in String]] | Medium | sliding-window, fixed-window, frequency-count |
| 19 | [[LC 76 Minimum Window Substring]] | Hard | sliding-window, shrink-while-valid, formed-counter, hash-map |
| 20 | [[LC 239 Sliding Window Maximum]] | Hard | sliding-window, monotonic-deque, heap-lazy-deletion |

### Stack — week 1 · pattern note [[Stack & Monotonic Stack]]

| # | Problem | Difficulty | Patterns |
|---|---|---|---|
| 21 | [[LC 20 Valid Parentheses]] | Easy | stack, bracket-matching, lifo |
| 22 | [[LC 155 Min Stack]] | Medium | stack, auxiliary-stack, design, running-minimum |
| 23 | [[LC 150 Evaluate Reverse Polish Notation]] | Medium | stack, postfix-evaluation, operand-order |
| 24 | [[LC 22 Generate Parentheses]] | Medium | backtracking, pruning, catalan, stack-depth |
| 25 | [[LC 739 Daily Temperatures]] | Medium | monotonic-stack, next-greater-element, indices-on-stack |
| 26 | [[LC 853 Car Fleet]] | Medium | monotonic-stack, sorting, arrival-time, greedy |
| 27 | [[LC 84 Largest Rectangle in Histogram]] | Hard | monotonic-stack, previous-smaller, next-smaller, sentinel |

### Binary Search — week 2 · pattern note [[Binary Search]]

| # | Problem | Difficulty | Patterns |
|---|---|---|---|
| 28 | [[LC 704 Binary Search]] | Easy | binary-search, closed-interval, loop-invariant |
| 29 | [[LC 74 Search a 2D Matrix]] | Medium | binary-search, matrix, flattened-index, virtual-array |
| 30 | [[LC 875 Koko Eating Bananas]] | Medium | binary-search-on-answer, monotonic-predicate, lower-bound, ceiling-division |
| 31 | [[LC 33 Search in Rotated Sorted Array]] | Medium | binary-search, rotated-array, sorted-half |
| 32 | [[LC 153 Find Minimum in Rotated Sorted Array]] | Medium | binary-search, rotated-array, first-true |
| 33 | [[LC 981 Time Based Key-Value Store]] | Medium | binary-search, last-true, design, hash-map |
| 34 | [[LC 4 Median of Two Sorted Arrays]] | Hard | binary-search, partition, two-sorted-arrays, kth-element |

### Linked List — week 2 · pattern note [[Linked Lists]]

| # | Problem | Difficulty | Patterns |
|---|---|---|---|
| 35 | [[LC 206 Reverse Linked List]] | Easy | in-place-reversal, recursion |
| 36 | [[LC 21 Merge Two Sorted Lists]] | Easy | merge, dummy-head, two-pointers |
| 37 | [[LC 141 Linked List Cycle]] | Easy | fast-slow-pointers |
| 38 | [[LC 143 Reorder List]] | Medium | fast-slow-pointers, in-place-reversal, merge |
| 39 | [[LC 19 Remove Nth Node From End of List]] | Medium | two-pointers, dummy-head |
| 40 | [[LC 138 Copy List with Random Pointer]] | Medium | hash-map, interleaving |
| 41 | [[LC 2 Add Two Numbers]] | Medium | digit-arithmetic, dummy-head, carry |
| 42 | [[LC 287 Find the Duplicate Number]] | Medium | fast-slow-pointers, binary-search-on-answer |
| 43 | [[LC 146 LRU Cache]] | Medium | hash-map, doubly-linked-list, design |
| 44 | [[LC 23 Merge k Sorted Lists]] | Hard | heap, divide-and-conquer, merge |
| 45 | [[LC 25 Reverse Nodes in k-Group]] | Hard | in-place-reversal, dummy-head |

### Trees — week 3 · pattern note [[Trees]]

| # | Problem | Difficulty | Patterns |
|---|---|---|---|
| 46 | [[LC 226 Invert Binary Tree]] | Easy | dfs, recursion |
| 47 | [[LC 104 Maximum Depth of Binary Tree]] | Easy | dfs, bfs, recursion |
| 48 | [[LC 543 Diameter of Binary Tree]] | Easy | dfs, recursion |
| 49 | [[LC 110 Balanced Binary Tree]] | Easy | dfs, recursion |
| 50 | [[LC 100 Same Tree]] | Easy | dfs, recursion |
| 51 | [[LC 572 Subtree of Another Tree]] | Easy | dfs, recursion |
| 52 | [[LC 235 Lowest Common Ancestor of a Binary Search Tree]] | Medium | bst, dfs |
| 53 | [[LC 102 Binary Tree Level Order Traversal]] | Medium | bfs |
| 54 | [[LC 199 Binary Tree Right Side View]] | Medium | bfs, dfs |
| 55 | [[LC 1448 Count Good Nodes in Binary Tree]] | Medium | dfs, recursion |
| 56 | [[LC 98 Validate Binary Search Tree]] | Medium | dfs, bst, recursion |
| 57 | [[LC 230 Kth Smallest Element in a BST]] | Medium | bst, dfs, inorder |
| 58 | [[LC 105 Construct Binary Tree from Preorder and Inorder Traversal]] | Medium | dfs, recursion, hash-map |
| 59 | [[LC 124 Binary Tree Maximum Path Sum]] | Hard | dfs, recursion |
| 60 | [[LC 297 Serialize and Deserialize Binary Tree]] | Hard | dfs, bfs, recursion |

### Tries — week 3 · pattern note [[Tries]]

| # | Problem | Difficulty | Patterns |
|---|---|---|---|
| 61 | [[LC 208 Implement Trie Prefix Tree]] | Medium | trie, design |
| 62 | [[LC 211 Design Add and Search Words Data Structure]] | Medium | trie, dfs, design |
| 63 | [[LC 212 Word Search II]] | Hard | trie, dfs, backtracking |

### Heap / Priority Queue — week 2 · pattern note [[Heaps & Priority Queues]]

| # | Problem | Difficulty | Patterns |
|---|---|---|---|
| 64 | [[LC 703 Kth Largest Element in a Stream]] | Easy | heap, top-k, design |
| 65 | [[LC 1046 Last Stone Weight]] | Easy | heap, simulation |
| 66 | [[LC 973 K Closest Points to Origin]] | Medium | heap, top-k, quickselect |
| 67 | [[LC 215 Kth Largest Element in an Array]] | Medium | heap, top-k, quickselect |
| 68 | [[LC 621 Task Scheduler]] | Medium | heap, greedy, simulation |
| 69 | [[LC 355 Design Twitter]] | Medium | heap, k-way-merge, design, hash-map |
| 70 | [[LC 295 Find Median from Data Stream]] | Hard | heap, two-heaps, design, running-median |

### Backtracking — week 3 · pattern note [[Backtracking]]

| # | Problem | Difficulty | Patterns |
|---|---|---|---|
| 71 | [[LC 78 Subsets]] | Medium | backtracking |
| 72 | [[LC 39 Combination Sum]] | Medium | backtracking |
| 73 | [[LC 46 Permutations]] | Medium | backtracking |
| 74 | [[LC 90 Subsets II]] | Medium | backtracking |
| 75 | [[LC 40 Combination Sum II]] | Medium | backtracking |
| 76 | [[LC 79 Word Search]] | Medium | backtracking, dfs, matrix |
| 77 | [[LC 131 Palindrome Partitioning]] | Medium | backtracking, memoisation |
| 78 | [[LC 17 Letter Combinations of a Phone Number]] | Medium | backtracking |
| 79 | [[LC 51 N-Queens]] | Hard | backtracking |

### Graphs — week 4 · pattern note [[Graphs]]

| # | Problem | Difficulty | Patterns |
|---|---|---|---|
| 80 | [[LC 200 Number of Islands]] | Medium | bfs, dfs, matrix |
| 81 | [[LC 695 Max Area of Island]] | Medium | dfs, matrix |
| 82 | [[LC 133 Clone Graph]] | Medium | dfs, bfs, hash-map |
| 83 | [[LC 286 Walls and Gates]] | Medium | bfs, matrix |
| 84 | [[LC 994 Rotting Oranges]] | Medium | bfs, matrix |
| 85 | [[LC 417 Pacific Atlantic Water Flow]] | Medium | dfs, bfs, matrix |
| 86 | [[LC 130 Surrounded Regions]] | Medium | bfs, dfs, matrix |
| 87 | [[LC 207 Course Schedule]] | Medium | topological-sort, bfs, dfs |
| 88 | [[LC 210 Course Schedule II]] | Medium | topological-sort, bfs, dfs |
| 89 | [[LC 684 Redundant Connection]] | Medium | union-find, dfs |
| 90 | [[LC 323 Number of Connected Components in an Undirected Graph]] | Medium | union-find, dfs, bfs |
| 91 | [[LC 261 Graph Valid Tree]] | Medium | union-find, dfs, bfs |
| 92 | [[LC 127 Word Ladder]] | Hard | bfs, hash-map |

### Advanced Graphs — week 4 · pattern note [[Advanced Graphs]]

| # | Problem | Difficulty | Patterns |
|---|---|---|---|
| 93 | [[LC 332 Reconstruct Itinerary]] | Hard | eulerian-path, dfs, heap |
| 94 | [[LC 1584 Min Cost to Connect All Points]] | Medium | mst, prim, kruskal, union-find |
| 95 | [[LC 743 Network Delay Time]] | Medium | dijkstra, heap, bfs |
| 96 | [[LC 778 Swim in Rising Water]] | Hard | binary-search-on-answer, bfs, dijkstra, union-find |
| 97 | [[LC 269 Alien Dictionary]] | Hard | topological-sort, bfs, kahn |
| 98 | [[LC 787 Cheapest Flights Within K Stops]] | Medium | bellman-ford, dijkstra, tabulation |

### 1-D DP — week 5 · pattern note [[Dynamic Programming]]

| # | Problem | Difficulty | Patterns |
|---|---|---|---|
| 99 | [[LC 70 Climbing Stairs]] | Easy | memoisation, tabulation, linear-dp |
| 100 | [[LC 746 Min Cost Climbing Stairs]] | Easy | memoisation, tabulation, linear-dp |
| 101 | [[LC 198 House Robber]] | Medium | memoisation, tabulation, take-or-skip |
| 102 | [[LC 213 House Robber II]] | Medium | memoisation, tabulation, take-or-skip |
| 103 | [[LC 5 Longest Palindromic Substring]] | Medium | expand-around-centre, tabulation, two-pointers |
| 104 | [[LC 647 Palindromic Substrings]] | Medium | expand-around-centre, tabulation |
| 105 | [[LC 91 Decode Ways]] | Medium | linear-step, memoisation, tabulation |
| 106 | [[LC 322 Coin Change]] | Medium | unbounded-knapsack, memoisation, tabulation |
| 107 | [[LC 152 Maximum Product Subarray]] | Medium | running-best, kadane, memoisation, tabulation |
| 108 | [[LC 139 Word Break]] | Medium | segmentation, memoisation, tabulation |
| 109 | [[LC 300 Longest Increasing Subsequence]] | Medium | lis, memoisation, tabulation, binary-search |
| 110 | [[LC 416 Partition Equal Subset Sum]] | Medium | knapsack, subset-sum, memoisation, tabulation |

### 2-D DP — week 6 · pattern note [[2-D Dynamic Programming]]

| # | Problem | Difficulty | Patterns |
|---|---|---|---|
| 111 | [[LC 62 Unique Paths]] | Medium | grid-paths, memoisation, tabulation |
| 112 | [[LC 1143 Longest Common Subsequence]] | Medium | two-sequence, memoisation, tabulation |
| 113 | [[LC 309 Best Time to Buy and Sell Stock with Cooldown]] | Medium | state-machine, memoisation, tabulation |
| 114 | [[LC 518 Coin Change II]] | Medium | unbounded-knapsack, memoisation, tabulation |
| 115 | [[LC 494 Target Sum]] | Medium | knapsack, subset-sum, memoisation, tabulation |
| 116 | [[LC 97 Interleaving String]] | Medium | tabulation, memoisation |
| 117 | [[LC 329 Longest Increasing Path in a Matrix]] | Hard | memoisation, dfs, matrix, topological-sort |
| 118 | [[LC 115 Distinct Subsequences]] | Hard | 2d-dp, string-dp, counting-dp, tabulation |
| 119 | [[LC 72 Edit Distance]] | Medium | 2d-dp, string-dp, tabulation, memoisation |
| 120 | [[LC 312 Burst Balloons]] | Hard | 2d-dp, interval-dp, last-decision, memoisation |
| 121 | [[LC 10 Regular Expression Matching]] | Hard | 2d-dp, string-dp, boolean-dp, memoisation |

### Greedy — week 5 · pattern note [[Greedy]]

| # | Problem | Difficulty | Patterns |
|---|---|---|---|
| 122 | [[LC 53 Maximum Subarray]] | Medium | greedy, kadane, prefix-sum |
| 123 | [[LC 55 Jump Game]] | Medium | greedy, frontier |
| 124 | [[LC 45 Jump Game II]] | Medium | greedy, bfs, frontier |
| 125 | [[LC 134 Gas Station]] | Medium | greedy, prefix-sum |
| 126 | [[LC 846 Hand of Straights]] | Medium | greedy, hash-map, sorting |
| 127 | [[LC 1899 Merge Triplets to Form Target Triplet]] | Medium | greedy, filtering |
| 128 | [[LC 763 Partition Labels]] | Medium | greedy, hash-map, frontier |
| 129 | [[LC 678 Valid Parenthesis String]] | Medium | greedy, range-tracking, stack |

### Intervals — week 6 · pattern note [[Intervals]]

| # | Problem | Difficulty | Patterns |
|---|---|---|---|
| 130 | [[LC 57 Insert Interval]] | Medium | intervals, merge, linear-scan |
| 131 | [[LC 56 Merge Intervals]] | Medium | intervals, sorting, merge |
| 132 | [[LC 435 Non-overlapping Intervals]] | Medium | intervals, greedy, sort-by-end, interval-scheduling |
| 133 | [[LC 252 Meeting Rooms]] | Easy | intervals, sort-by-start, overlap-check |
| 134 | [[LC 253 Meeting Rooms II]] | Medium | intervals, heap, sweep-line, two-pointers |
| 135 | [[LC 1851 Minimum Interval to Include Each Query]] | Hard | intervals, heap, offline-queries, sweep-line |

### Math & Geometry — week 7 · pattern note [[Math & Geometry]]

| # | Problem | Difficulty | Patterns |
|---|---|---|---|
| 136 | [[LC 48 Rotate Image]] | Medium | matrix |
| 137 | [[LC 54 Spiral Matrix]] | Medium | matrix |
| 138 | [[LC 73 Set Matrix Zeroes]] | Medium | matrix |
| 139 | [[LC 202 Happy Number]] | Easy | fast-slow-pointers, hash-map |
| 140 | [[LC 66 Plus One]] | Easy |  |
| 141 | [[LC 50 Pow(x, n)]] | Medium | bit-tricks |
| 142 | [[LC 43 Multiply Strings]] | Medium | math, digit-arithmetic, grid, carry |
| 143 | [[LC 2013 Detect Squares]] | Medium | math, geometry, hash-map, counting |

### Bit Manipulation — week 7 · pattern note [[Bit Manipulation]]

| # | Problem | Difficulty | Patterns |
|---|---|---|---|
| 144 | [[LC 136 Single Number]] | Easy | bit-tricks, xor |
| 145 | [[LC 191 Number of 1 Bits]] | Easy | bit-tricks, popcount, lowest-set-bit |
| 146 | [[LC 338 Counting Bits]] | Easy | bit-tricks, dp, popcount |
| 147 | [[LC 190 Reverse Bits]] | Easy | bit-tricks, unsigned-shift, masks |
| 148 | [[LC 268 Missing Number]] | Easy | bit-tricks, xor, math, gauss-sum |
| 149 | [[LC 371 Sum of Two Integers]] | Medium | bit-tricks, xor, carry, two's-complement |
| 150 | [[LC 7 Reverse Integer]] | Medium | math, digit-arithmetic, overflow-check |

## Live progress

![[Problems.base#All]]

## Related
- [[Topics MOC]] · [[Pattern Recognition Table]] · [[8-Week DSA Plan]] · [[DSA Dashboard]]
