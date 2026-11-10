---
title: Pattern Recognition Table
type: guide
category: Plans
tags: [guide, plan, patterns]
aliases: [Cue Table, Which Pattern]
summary: "Forty rows from the words in a problem statement to the pattern they signal, the topic note that teaches it and the two problems to try first — the table to scan in the first five minutes of a round when nothing comes to mind."
created: 2026-11-04
updated: 2026-11-04
---

# Pattern Recognition Table

> [!abstract] In one sentence
> Most problems announce their pattern in the first two lines — "sorted", "k largest", "longest substring", "prerequisites", "number of ways" — and the fastest way to get unstuck is to match those words against a table you have read enough times that the match happens on its own.

**Map:** [[Plans MOC]] → [[DSA Dashboard]]

> [!tip] How to use
> Read the *cue* column once a week until you can cover the other columns and recite them. In a round, scan it mentally during minutes 5–12 of the [[Interview Day Protocol]]. When a review fails because you did not *recognise* the pattern (as opposed to failing to code it), add the cue that should have triggered to the [[Mistakes Log]] as `misread`.

| Statement cue | Pattern | Topic note | First problem | Second problem |
|---|---|---|---|---|
| "contains duplicate", "count of each", "frequency" | hash map / count array | [[Arrays & Hashing]] | [[LC 217 Contains Duplicate]] | [[LC 347 Top K Frequent Elements]] |
| "anagram", "same letters" | count key (int[26]) | [[Hashing]] | [[LC 242 Valid Anagram]] | [[LC 49 Group Anagrams]] |
| "product / sum of all except self" | prefix and suffix arrays | [[Arrays & Hashing]] | [[LC 238 Product of Array Except Self]] | [[LC 128 Longest Consecutive Sequence]] |
| sorted array + "pair / triple summing to" | two pointers from the ends | [[Two Pointers]] | [[LC 167 Two Sum II Input Array Is Sorted]] | [[LC 15 3Sum]] |
| "palindrome" | two pointers / expand around centre | [[Strings]] | [[LC 125 Valid Palindrome]] | [[LC 5 Longest Palindromic Substring]] |
| "most water", "trapped" | two pointers moving the smaller side | [[Two Pointers]] | [[LC 11 Container With Most Water]] | [[LC 42 Trapping Rain Water]] |
| "longest substring with / without" | variable sliding window | [[Sliding Window]] | [[LC 3 Longest Substring Without Repeating Characters]] | [[LC 424 Longest Repeating Character Replacement]] |
| "minimum window containing" | window with have/need counts | [[Sliding Window]] | [[LC 76 Minimum Window Substring]] | [[LC 567 Permutation in String]] |
| "maximum of each window of size k" | monotonic deque | [[Sliding Window]] | [[LC 239 Sliding Window Maximum]] | [[LC 739 Daily Temperatures]] |
| "next greater", "days until warmer", "largest rectangle" | monotonic stack | [[Stack & Monotonic Stack]] | [[LC 739 Daily Temperatures]] | [[LC 84 Largest Rectangle in Histogram]] |
| "valid parentheses", "evaluate expression" | stack | [[Stack & Monotonic Stack]] | [[LC 20 Valid Parentheses]] | [[LC 150 Evaluate Reverse Polish Notation]] |
| sorted / rotated + "find", "O(log n)" | binary search on index | [[Binary Search]] | [[LC 704 Binary Search]] | [[LC 33 Search in Rotated Sorted Array]] |
| "minimum speed / capacity such that" | binary search on the answer | [[Binary Search]] | [[LC 875 Koko Eating Bananas]] | [[LC 778 Swim in Rising Water]] |
| "median of two sorted" | partition binary search | [[Binary Search]] | [[LC 4 Median of Two Sorted Arrays]] | [[LC 295 Find Median from Data Stream]] |
| "reverse / reorder / remove nth from end" on a list | dummy head, fast/slow, in-place reversal | [[Linked Lists]] | [[LC 206 Reverse Linked List]] | [[LC 19 Remove Nth Node From End of List]] |
| "cycle", "duplicate number in 1..n" | Floyd's fast/slow pointers | [[Linked Lists]] | [[LC 141 Linked List Cycle]] | [[LC 287 Find the Duplicate Number]] |
| "LRU", "get and put in O(1)" | hash map + doubly linked list | [[Linked Lists]] | [[LC 146 LRU Cache]] | [[LC 981 Time Based Key-Value Store]] |
| "depth / diameter / balanced / same tree" | post-order recursion returning a value | [[Trees]] | [[LC 104 Maximum Depth of Binary Tree]] | [[LC 543 Diameter of Binary Tree]] |
| "level order", "right side view" | BFS by levels | [[Trees]] | [[LC 102 Binary Tree Level Order Traversal]] | [[LC 199 Binary Tree Right Side View]] |
| "validate BST", "kth smallest in BST" | bounds-passing / inorder traversal | [[Trees]] | [[LC 98 Validate Binary Search Tree]] | [[LC 230 Kth Smallest Element in a BST]] |
| "prefix", "starts with", "wildcard search", "many words in a grid" | trie | [[Tries]] | [[LC 208 Implement Trie Prefix Tree]] | [[LC 212 Word Search II]] |
| "k largest / closest / most frequent" | size-k heap | [[Heaps & Priority Queues]] | [[LC 215 Kth Largest Element in an Array]] | [[LC 973 K Closest Points to Origin]] |
| "running median" | two heaps | [[Heaps & Priority Queues]] | [[LC 295 Find Median from Data Stream]] | [[LC 703 Kth Largest Element in a Stream]] |
| "all subsets / permutations / combinations" | backtracking (choose, explore, unchoose) | [[Backtracking]] | [[LC 78 Subsets]] | [[LC 46 Permutations]] |
| "number of islands", "flood", "rotting" | grid BFS / DFS, multi-source BFS | [[Graphs]] | [[LC 200 Number of Islands]] | [[LC 994 Rotting Oranges]] |
| "prerequisites", "order of courses", "cycle in directed graph" | topological sort (Kahn) | [[Graphs]] | [[LC 207 Course Schedule]] | [[LC 210 Course Schedule II]] |
| "connected components", "redundant edge", "valid tree" | union-find | [[Union-Find]] | [[LC 323 Number of Connected Components in an Undirected Graph]] | [[LC 684 Redundant Connection]] |
| "shortest path with weights", "network delay" | Dijkstra | [[Advanced Graphs]] | [[LC 743 Network Delay Time]] | [[LC 787 Cheapest Flights Within K Stops]] |
| "connect all points at minimum cost" | MST (Prim / Kruskal) | [[Advanced Graphs]] | [[LC 1584 Min Cost to Connect All Points]] | [[LC 684 Redundant Connection]] |
| "number of ways", "minimum cost to reach", "can we partition" | 1-D DP (memo → table) | [[Dynamic Programming]] | [[LC 70 Climbing Stairs]] | [[LC 322 Coin Change]] |
| "longest increasing subsequence" | LIS DP / patience sorting | [[Dynamic Programming]] | [[LC 300 Longest Increasing Subsequence]] | [[LC 329 Longest Increasing Path in a Matrix]] |
| two strings + "edit / common / match" | 2-D string DP | [[2-D Dynamic Programming]] | [[LC 1143 Longest Common Subsequence]] | [[LC 72 Edit Distance]] |
| "coins / items with repetition or without" | knapsack DP | [[2-D Dynamic Programming]] | [[LC 518 Coin Change II]] | [[LC 416 Partition Equal Subset Sum]] |
| "burst / merge / remove in an interval" | interval DP (last operation) | [[2-D Dynamic Programming]] | [[LC 312 Burst Balloons]] | [[LC 10 Regular Expression Matching]] |
| "can jump / fewest jumps / gas station" | greedy with an exchange argument | [[Greedy]] | [[LC 55 Jump Game]] | [[LC 134 Gas Station]] |
| "merge / insert / non-overlapping intervals" | sort by start or end | [[Intervals]] | [[LC 56 Merge Intervals]] | [[LC 435 Non-overlapping Intervals]] |
| "minimum rooms", "maximum concurrent" | sweep line / heap of ends | [[Intervals]] | [[LC 253 Meeting Rooms II]] | [[LC 1851 Minimum Interval to Include Each Query]] |
| "rotate / spiral / set zeroes" on a matrix | index arithmetic in place | [[Math & Geometry]] | [[LC 48 Rotate Image]] | [[LC 73 Set Matrix Zeroes]] |
| "single number", "missing number", "without +" | XOR / bit tricks | [[Bit Manipulation]] | [[LC 136 Single Number]] | [[LC 371 Sum of Two Integers]] |
| "range sum with updates" | Fenwick / segment tree | [[Fenwick Trees]] | [[LC 1851 Minimum Interval to Include Each Query]] | [[LC 239 Sliding Window Maximum]] |

## When two patterns both fit

- Sorted input and a target: binary search ([[Binary Search]]) if you need one position; two pointers ([[Two Pointers]]) if you need a pair or a range.
- Counting with a window versus a prefix sum: a window when the property is monotone under shrinking ("at most k distinct"); prefix sums when it is not ("exactly k", "sum equals target" → prefix sums plus a hash map).
- Greedy versus DP: try the exchange argument ([[Greedy]]); if you cannot state why the local choice is safe, it is DP ([[DP Pattern Catalogue]]).
- Union-find versus DFS for components: both work; union-find when edges arrive over time or when you also need cycle detection on the fly ([[Union-Find]]).
- Heap versus sort for "top k": a heap when `k ≪ n` or the data streams ([[Heaps & Priority Queues]]); sort when you need everything ordered anyway ([[Sorting]]).

## Related

- [[Problem-Solving Framework]] — the protocol the cues plug into
- [[DP Pattern Catalogue]] — the finer-grained table for DP alone
- [[8-Week DSA Plan]] · [[Spaced Repetition Protocol]] · [[Mistakes Log]] · [[Interview Day Protocol]]
