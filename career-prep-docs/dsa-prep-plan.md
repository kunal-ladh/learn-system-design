# DSA Prep Plan — SDE2 Mentor Playbook

> **For**: 5 YOE backend engineer (Java) who has done LeetCode in fits and starts.
> **Reality check**: Multiple false starts, no clean breakthrough. The fix is the **system**, not the effort.
> **Honest audit**:
>   - **Solid**: arrays, strings, hashing, linked list
>   - **Lukewarm**: trees, BST, heaps, two pointers, sliding window, greedy, recursion, searching, sorting
>   - **Zero**: DP, Graphs, Tries
> **Goal**: Clear medium-level OAs + DSA rounds at SDE2 product companies in 8 weeks.

---

## Table of Contents

1. [Why You've Dropped Out Before](#1-why-youve-dropped-out-before)
2. [The Anti-Dropout Protocol](#2-the-anti-dropout-protocol)
3. [Honest Skill Audit + Reorder](#3-honest-skill-audit--reorder)
4. [The 8-Week Plan](#4-the-8-week-plan)
5. [Daily Rhythm](#5-daily-rhythm)
6. [Java Template Vault](#6-the-java-template-vault)
7. [Spaced Repetition Protocol](#7-the-spaced-repetition-protocol)
8. [Mistakes Log](#8-mistakes-log)
9. [When to Start Applying](#9-when-to-start-applying)
10. [Resources](#10-resources-pick-one-primary)
11. [Self-Assessment Checkpoints](#11-self-assessment-checkpoints)

---

## 1. Why You've Dropped Out Before

You drop out around weeks 3-4 because you treat DSA as a sprint where you need to "feel ready" before applying. The cycle is:

**Burn 6h/day → get to medium → no external pressure → motivation dies → 2-week gap → restart from "easy" → loop**

The protocol below makes dropping out almost impossible. Read this section twice before moving on.

The mindset shift: **DSA is not about feeling ready. It's about maintaining a streak long enough that ready arrives by accident.**

---

## 2. The Anti-Dropout Protocol

These rules are non-negotiable.

### Rule 1: The 1-Problem Minimum
On bad days, do **one problem**. Even if you peek at the editorial. Just keep the streak alive. **Zero days = relapse risk.** One day off becomes three becomes a month.

### Rule 2: The 48-Hour Rule
Never miss two days in a row. Ever. After one miss, the next day is mandatory — non-negotiable.

### Rule 3: One-List Discipline
Stop mixing NeetCode 150 + Striver + Blind 75 + LeetCode hot 100. Pick **NeetCode 150** as your spine. Reference others only when you need extra problems on a specific topic. Channel-hopping is dropout fuel.

### Rule 4: Public Commit Log
Create a GitHub repo: `dsa-prep`. One folder per topic. Daily commits with the problem name + your solution. The streak itself becomes the motivation.

### Rule 5: Pattern Over Problem Count
Doing 200 random LCs ≠ doing 100 *patterned* problems with spaced review. Solving without re-solving is shallow.

### Rule 6: Start Applying Before You Feel Ready
Apply at end of **Week 4**, not Week 8. You'll bomb the first OAs — that's the price of admission. The pressure of pending interviews kills the dropout urge.

---

## 3. Honest Skill Audit + Reorder

Based on what you said:

| Topic | Status | Action |
|---|---|---|
| Arrays / Strings | Solid | Keep warm via daily mixing |
| Hashing | Solid | Keep warm |
| Linked List | Solid | Keep warm |
| Two Pointers | Lukewarm | Lock down Week 1 |
| Sliding Window | Lukewarm | Lock down Week 1 |
| Searching / Sorting | Lukewarm | Lock down Week 2 |
| Trees / BST | Lukewarm | Lock down Week 2 |
| Heaps | Lukewarm | Lock down Week 2 (high leverage) |
| Recursion / Backtracking | Lukewarm | Embedded throughout |
| Greedy | Lukewarm | Just-in-time during DP phase |
| **Graphs** | Zero | **P0 — Week 3-4** |
| **DP** | Zero | **P0 — Week 5-6** |
| **Trie** | Zero | **P0 — Week 7** |
| Union Find | Cold | Week 4 (pairs with Graph) |
| Segment Tree / Fenwick | Skip | Not needed for SDE2 |

**Hidden priority insight**: Heaps + Sliding Window + Graphs are the 3 highest-leverage topics for SDE2 OAs in India. They show up disproportionately at PhonePe, Flipkart, Razorpay, Swiggy, Amazon.

---

## 4. The 8-Week Plan

Target: ~14 problems/week (2/day weekdays + 4 weekend) + 1 LeetCode weekly contest (Sunday morning).

### Week 1 — Two Pointers + Sliding Window (Lock-Down)

Goal: re-build muscle memory on patterns you "kinda know".

- 4 re-do problems on TP/SW from NeetCode
- 10 new mediums (mix TP and SW)
- Build ONE template each for: fixed-size window, variable window, two-pointer-from-ends, fast/slow pointer

**Key problems**: Longest Substring Without Repeating, Minimum Window Substring, Container With Most Water, 3Sum, Trapping Rain Water (HARD, attempt only)

### Week 2 — Heaps + Trees (Lock-Down)

- 7 heap problems (top K, k-way merge, two-heap median, scheduling)
- 7 tree problems (BFS, DFS, level order, lowest common ancestor, validate BST)
- Build templates: PriorityQueue with comparator, recursive DFS, iterative BFS

**Key problems**: Kth Largest Element, Top K Frequent, Merge K Sorted Lists, Find Median from Data Stream, BST validation, Level Order Traversal, LCA

### Week 3 — Graphs Part 1 (BFS / DFS / Grids)

Your P0 topic begins. Don't be scared — graphs are *easier* than DP once the template clicks.

- BFS template
- DFS template (recursive + iterative)
- Grid traversal (most common interview pattern)
- Adjacency list construction
- 14 problems

**Key problems**: Number of Islands, Clone Graph, Pacific Atlantic, Rotting Oranges, Walls and Gates, Surrounded Regions, Course Schedule (intro to topo), Word Ladder

### Week 4 — Graphs Part 2 (Topo Sort + Shortest Path + Union Find)

- Kahn's algorithm (BFS-based topo)
- DFS-based topo
- BFS shortest path (unweighted)
- Dijkstra with PriorityQueue
- Union-Find template + 4 DSU problems
- 14 problems

**Key problems**: Course Schedule II, Alien Dictionary, Network Delay Time, Cheapest Flights Within K Stops, Number of Connected Components, Redundant Connection, Accounts Merge, Graph Valid Tree

### Week 5 — DP Part 1 (1D)

Goal: stop fearing DP. The trick is the formula: **state + transition + base case**. That's the whole game.

- House Robber I & II
- Coin Change I & II
- Climbing Stairs (and variants)
- Longest Increasing Subsequence (LIS) — both O(n²) and O(n log n)
- Decode Ways
- Word Break
- Maximum Subarray (Kadane's)
- 14 problems

### Week 6 — DP Part 2 (2D + Classic Patterns)

- Unique Paths I & II
- Minimum Path Sum
- Longest Common Subsequence (LCS)
- Edit Distance
- 0/1 Knapsack (template)
- Partition Equal Subset Sum
- Longest Palindromic Substring
- 12 problems

### Week 7 — Trie + Binary Search on Answer + Backtracking Polish

- Trie template (insert, search, startsWith)
- Trie problems: Word Search II, Replace Words, Design Add and Search Words
- BS-on-answer: Koko Eating Bananas, Capacity to Ship Packages, Median of Two Sorted Arrays (hard but worth it)
- Backtracking review: Word Search, Subsets, Permutations, Combination Sum
- 14 problems

### Week 8 — Mocks + Weak-Area Drilling

- Daily 90-min mock OA (2-3 problems, fully timed)
- Re-do failed problems from Weeks 1-7 (spaced repetition kicks in here)
- 1 LeetCode contest
- Solve under interview pressure (camera on, talk through approach out loud)

---

## 5. Daily Rhythm

Slot into your existing 5:00-6:45 AM morning DSA block.

| Time | Activity |
|---|---|
| 5:00-5:10 | Wake up, hydrate, re-read yesterday's mistake notes |
| 5:10-6:00 | Problem 1 (medium, **timed 50 min**) |
| 6:00-6:40 | Problem 2 (medium, **timed 40 min** — should be on same pattern) |
| 6:40-6:50 | Mistakes log: pattern category, edge case missed, complexity |
| 6:50-7:00 | Spaced repetition: re-do 1 problem from D-3, D-7, or D-21 list |

### Saturday (2-3 hours total)
- 90-min OA mock (2-3 problems, NO editorial peeking)
- 60-min review: re-implement wrong ones from scratch
- 1 hard problem attempt (don't worry about solving)

### Sunday (3-4 hours)
- **LeetCode Weekly Contest** at 8:00 AM IST (mandatory — most underused tool)
- 1-hour post-contest review (read editorials for problems you missed)
- Apply to 5-10 jobs (after Week 4)

---

## 6. The Java Template Vault

You **MUST** memorize these cold. Type them from memory every morning for the first 2 weeks until your fingers know them.

### BFS Template (Grid)
```java
Queue<int[]> q = new ArrayDeque<>();
q.offer(new int[]{startR, startC});
boolean[][] visited = new boolean[m][n];
visited[startR][startC] = true;
int[][] dirs = {{-1,0},{1,0},{0,-1},{0,1}};
while (!q.isEmpty()) {
    int[] cur = q.poll();
    for (int[] d : dirs) {
        int nr = cur[0] + d[0], nc = cur[1] + d[1];
        if (nr<0||nr>=m||nc<0||nc>=n||visited[nr][nc]) continue;
        visited[nr][nc] = true;
        q.offer(new int[]{nr, nc});
    }
}
```

### DFS Recursive (Adjacency List)
```java
void dfs(int node, List<List<Integer>> g, boolean[] visited) {
    visited[node] = true;
    for (int nei : g.get(node)) {
        if (!visited[nei]) dfs(nei, g, visited);
    }
}
```

### Priority Queue (Min/Max Heap)
```java
PriorityQueue<int[]> minHeap = new PriorityQueue<>((a,b) -> a[0] - b[0]);
PriorityQueue<int[]> maxHeap = new PriorityQueue<>((a,b) -> b[0] - a[0]);
// For long values: use Long.compare(a, b) to avoid overflow
```

### Union Find
```java
int[] parent, rank;
int find(int x) {
    if (parent[x] != x) parent[x] = find(parent[x]);
    return parent[x];
}
boolean union(int x, int y) {
    int rx = find(x), ry = find(y);
    if (rx == ry) return false;
    if (rank[rx] < rank[ry]) { int t = rx; rx = ry; ry = t; }
    parent[ry] = rx;
    if (rank[rx] == rank[ry]) rank[rx]++;
    return true;
}
```

### Sliding Window (Variable Size)
```java
int l = 0, best = 0;
Map<Character, Integer> count = new HashMap<>();
for (int r = 0; r < s.length(); r++) {
    count.merge(s.charAt(r), 1, Integer::sum);
    while (/* invariant broken */) {
        count.merge(s.charAt(l), -1, Integer::sum);
        if (count.get(s.charAt(l)) == 0) count.remove(s.charAt(l));
        l++;
    }
    best = Math.max(best, r - l + 1);
}
```

### Trie
```java
class TrieNode {
    TrieNode[] children = new TrieNode[26];
    boolean isEnd;
}
void insert(TrieNode root, String w) {
    TrieNode cur = root;
    for (char c : w.toCharArray()) {
        if (cur.children[c-'a'] == null) cur.children[c-'a'] = new TrieNode();
        cur = cur.children[c-'a'];
    }
    cur.isEnd = true;
}
```

### Dijkstra (Shortest Path)
```java
PriorityQueue<int[]> pq = new PriorityQueue<>((a,b) -> a[1] - b[1]);
int[] dist = new int[n];
Arrays.fill(dist, Integer.MAX_VALUE);
dist[src] = 0;
pq.offer(new int[]{src, 0});
while (!pq.isEmpty()) {
    int[] cur = pq.poll();
    int u = cur[0], d = cur[1];
    if (d > dist[u]) continue;
    for (int[] e : graph.get(u)) {
        int v = e[0], w = e[1];
        if (dist[u] + w < dist[v]) {
            dist[v] = dist[u] + w;
            pq.offer(new int[]{v, dist[v]});
        }
    }
}
```

### Topological Sort (Kahn's BFS)
```java
int[] indegree = new int[n];
// ... compute indegrees from edges ...
Queue<Integer> q = new ArrayDeque<>();
for (int i = 0; i < n; i++) if (indegree[i] == 0) q.offer(i);
List<Integer> order = new ArrayList<>();
while (!q.isEmpty()) {
    int u = q.poll();
    order.add(u);
    for (int v : graph.get(u)) {
        if (--indegree[v] == 0) q.offer(v);
    }
}
// if order.size() != n, cycle exists
```

### 1D DP Skeleton
```java
int[] dp = new int[n + 1];
dp[0] = /* base case */;
for (int i = 1; i <= n; i++) {
    dp[i] = /* transition: f(dp[i-1], dp[i-2], ...) */;
}
return dp[n];
```

### 2D DP Skeleton
```java
int[][] dp = new int[m + 1][n + 1];
// init base cases
for (int i = 1; i <= m; i++) {
    for (int j = 1; j <= n; j++) {
        dp[i][j] = /* transition */;
    }
}
return dp[m][n];
```

---

## 7. The Spaced Repetition Protocol

This is the cheat code that prevents skill decay. Every problem you solve gets logged with:
- Date solved
- Difficulty perceived (your scale: easy/medium/hard for you)
- Re-do dates: **+3, +7, +21 days**

If you fail a re-do, **restart the schedule** for that problem.

Use a Google Sheet or Notion table:

| Problem | Pattern | First solve | +3 | +7 | +21 | Status |
|---|---|---|---|---|---|---|
| Number of Islands | Graph BFS | 2026-05-20 | 2026-05-23 ✓ | 2026-05-27 ✓ | 2026-06-10 | Done |
| Coin Change | DP 1D | 2026-05-21 | 2026-05-24 ✗ | RESET | — | Failed |

This is the single highest-ROI habit in DSA prep. Most people skip it. Don't.

---

## 8. Mistakes Log

A single MD file: `dsa-prep/mistakes.md`. Append after every problem. Each entry:

- Problem name + URL
- The bug / what you missed (1 sentence)
- Pattern category
- Java syntax/API gotcha (if any)

Examples:
- "Empty input case missed in sliding window — for-loop didn't enter, returned 0 instead of -1"
- "Used `==` instead of `.equals()` on Integer (autoboxed) — wrong for values > 127"
- "PriorityQueue comparator returned `(a-b)` — works for `int` but **overflowed** when values were near `Integer.MAX_VALUE`. Use `Integer.compare(a,b)`."
- "Forgot `visited[start] = true` BEFORE adding to queue — caused infinite loop in BFS"

**Read this file every Monday morning.** It compresses your mistakes into pattern memory faster than re-solving alone.

---

## 9. When to Start Applying

**End of Week 4. Not later.**

| Week | What to apply to |
|---|---|
| 4-6 | Tier C (Indian product: Razorpay, Cred, PhonePe, Swiggy, Meesho, Groww) — warm-up loops |
| 6-8 | Add Tier B (Amazon, Microsoft, Adobe, Salesforce, LinkedIn) |
| 8+ | Add Tier A (Google, Meta, Uber, Atlassian, Stripe) |

You'll bomb the first 2-3 OAs. That's the price. The pressure kills the dropout urge. The 4th will pass.

---

## 10. Resources (Pick ONE Primary, No Channel-Hopping)

| Type | Pick | Why |
|---|---|---|
| Problem list | **NeetCode 150** | Your spine. Already started. Continue. |
| Pattern videos | **NeetCode YouTube** | Java solutions, clean explanations |
| Backup videos | **Striver YouTube** | Best for graphs + DP in Hindi/English mix |
| Contest | **LeetCode weekly** (Sun 8 AM IST) | Mandatory — real interview pressure |
| Java reference | **CTCI** — only chapters on topics you're stuck on | Don't read cover-to-cover |
| For DP intuition | **Aditya Verma** (DP playlist only) | Recursion → memoization → tabulation framework |

Stop watching anything else. No infinite scrolling of "DSA tips" videos. Watch a pattern once → solve 8 problems on it.

---

## 11. Self-Assessment Checkpoints

### End of Week 2
- [ ] BFS, DFS, Sliding Window, Heap templates typed from memory
- [ ] 25+ problems solved in 2-week block
- [ ] 1 LeetCode weekly contest attempted (no matter the score)
- [ ] Mistakes log has 15+ entries

### End of Week 4
- [ ] Graphs no longer a fear topic
- [ ] Union Find template typed from memory
- [ ] 50+ problems total
- [ ] 1 OA mock cleared at 60%+ (2/3 problems)
- [ ] **Applied to 10+ companies**

### End of Week 6
- [ ] DP no longer a fear topic (1D feels natural)
- [ ] 80+ problems total
- [ ] 3 weekly contests done
- [ ] First interview round scheduled

### End of Week 8
- [ ] 100+ problems
- [ ] 4+ contests done
- [ ] Can solve a fresh medium in 25 min
- [ ] Active interview pipeline (3+ in progress)

---

## One-Page Cheat Sheet

```
ANTI-DROPOUT RULES:
  1. 1-problem minimum on bad days
  2. Never miss 2 days in a row
  3. One list only (NeetCode 150)
  4. Public commit log
  5. Pattern > count
  6. Apply at Week 4

DAILY (5:00-7:00 AM):
  - 2 timed mediums (same pattern)
  - Mistakes log
  - 1 spaced-repetition re-do

WEEKLY:
  - LeetCode contest Sunday 8 AM IST (no excuses)
  - Saturday OA mock
  - Weekly skill check

P0 GAP TOPICS (in order):
  Week 3-4: Graphs (BFS/DFS → Topo → Dijkstra → Union Find)
  Week 5-6: DP (1D → 2D)
  Week 7:   Trie + BS-on-answer

JAVA TEMPLATES (memorize):
  BFS grid, DFS recursive, PQ min/max,
  Union Find, Sliding Window, Trie,
  Dijkstra, Topo Sort, 1D DP, 2D DP

THE GOLDEN RULE:
  Doing 100 problems with re-do > Doing 200 once.
```

---

*Companion docs: `tech-refined.md` (overall plan), `lld-prep-plan.md` (LLD), `hld-prep-plan.md` (HLD), `advice-refined.md` (job search).*
