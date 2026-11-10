---
title: DSA Patterns in Java
type: guide
tags: [guide, java]
aliases: [Template vault, Java templates, Pattern templates]
summary: "The template vault: one compiling Java 21 block per interview pattern — two pointers, sliding window, prefix sums, monotonic stack and deque, the three binary-search templates, fast/slow pointers, list reversal, BFS and DFS, topological sort, Union-Find, Dijkstra, Prim, Kruskal, Trie, heaps, backtracking, 1-D and 2-D DP, knapsack, LIS, intervals, bit tricks — each with when to use it, its complexity and the NeetCode problems it solves."
related: ["[[Java Collections Cheat Sheet]]", "[[Java Pitfalls in Interviews]]", "[[Pattern Recognition Table]]", "[[Problem-Solving Framework]]", "[[DP Pattern Catalogue]]"]
created: 2026-11-04
updated: 2026-11-04
---

# DSA Patterns in Java

> [!abstract] In one sentence
> Thirty templates you type from memory until your fingers know them — every one compiles on its own, states its complexity, and names the NeetCode problems that are that template plus one twist.

**Map:** [[Java Refresher MOC]] → [[DSA Dashboard]]

> [!question] Predict before you read
> Of the thirty patterns below, which five cover more than half of NeetCode 150? Guess, then count the links as you read. (The answer is at the end of the note.)

> [!info] How to use this note
> The plan in [[8-Week DSA Plan]] asks you to type the templates of the current week from memory every morning for the first two weeks of that topic. Do it in a scratch file, compile it, diff against this note. The recognition cue for each pattern lives in [[Pattern Recognition Table]]; the mechanism and the proof live in the topic note linked under each heading. A block that starts with `// snippet` is a fragment and does not compile by itself; everything else does.

## Arrays and pointers

### Two pointers, opposite ends

When to use: sorted input (or a palindrome check) and a question about a *pair* — the sum tells you which pointer to move. Mechanism in [[Two Pointers]].

```java
import java.util.*;

class Solution {
    int[] twoSumSorted(int[] a, int target) {          // 1-indexed answer, as LC 167 wants
        int lo = 0, hi = a.length - 1;
        while (lo < hi) {
            int sum = a[lo] + a[hi];
            if (sum == target) return new int[]{lo + 1, hi + 1};
            if (sum < target) lo++;                     // need more: only lo can raise the sum
            else hi--;                                  // need less: only hi can lower it
        }
        return new int[]{-1, -1};
    }

    boolean isPalindrome(String s) {
        int l = 0, r = s.length() - 1;
        while (l < r) {
            while (l < r && !Character.isLetterOrDigit(s.charAt(l))) l++;
            while (l < r && !Character.isLetterOrDigit(s.charAt(r))) r--;
            if (Character.toLowerCase(s.charAt(l++)) != Character.toLowerCase(s.charAt(r--))) return false;
        }
        return true;
    }
}
```

**Time** O(n) · **Space** O(1)

Solves: [[LC 125 Valid Palindrome]], [[LC 167 Two Sum II Input Array Is Sorted]], [[LC 15 3Sum]] (sort, fix `i`, two-pointer the rest, skip duplicates), [[LC 11 Container With Most Water]] (move the shorter wall), [[LC 42 Trapping Rain Water]] (move the side with the smaller max).

### Sliding window, fixed size

When to use: "every window of size k", "any substring of length `p.length()`". Keep a count array, add the entering element, remove the leaving one. Mechanism in [[Sliding Window]].

```java
import java.util.*;

class Solution {
    boolean checkInclusion(String p, String s) {       // does s contain a permutation of p?
        int k = p.length();
        if (k > s.length()) return false;
        int[] need = new int[26], have = new int[26];
        for (char c : p.toCharArray()) need[c - 'a']++;
        for (int r = 0; r < s.length(); r++) {
            have[s.charAt(r) - 'a']++;                  // element enters
            if (r >= k) have[s.charAt(r - k) - 'a']--;  // element leaves once the window is full
            if (r >= k - 1 && Arrays.equals(need, have)) return true;   // 26 compares per step
        }
        return false;
    }
}
```

**Time** O(26·n) · **Space** O(1)

Solves: [[LC 567 Permutation in String]], and the deque form below for [[LC 239 Sliding Window Maximum]].

### Sliding window, variable size

When to use: "longest / shortest substring or subarray such that …" with a condition that is monotone under growing the window. Grow with `r`, shrink with `l` while the invariant is broken.

```java
import java.util.*;

class Solution {
    int longestNoRepeat(String s) {                      // longest substring without repeating chars
        Map<Character, Integer> count = new HashMap<>();
        int l = 0, best = 0;
        for (int r = 0; r < s.length(); r++) {
            count.merge(s.charAt(r), 1, Integer::sum);
            while (count.get(s.charAt(r)) > 1) {       // invariant broken: shrink from the left
                count.merge(s.charAt(l), -1, Integer::sum);
                l++;
            }
            best = Math.max(best, r - l + 1);
        }
        return best;
    }

    int minWindowLength(String s, String t) {           // shortest window of s containing all of t
        int[] need = new int[128];
        int missing = t.length();                       // chars of t still not covered by the window
        for (char c : t.toCharArray()) need[c]++;
        int l = 0, best = Integer.MAX_VALUE;
        for (int r = 0; r < s.length(); r++) {
            if (need[s.charAt(r)]-- > 0) missing--;     // only chars still needed reduce missing
            while (missing == 0) {                      // window valid: record, then shrink
                best = Math.min(best, r - l + 1);
                if (++need[s.charAt(l)] > 0) missing++; // left char was essential; window breaks
                l++;
            }
        }
        return best == Integer.MAX_VALUE ? 0 : best;
    }
}
```

**Time** O(n) — each index enters and leaves once · **Space** O(alphabet)

Solves: [[LC 3 Longest Substring Without Repeating Characters]], [[LC 424 Longest Repeating Character Replacement]] (shrink while `len - maxFreq > k`), [[LC 76 Minimum Window Substring]], [[LC 121 Best Time to Buy and Sell Stock]] (the degenerate window: track the running minimum).

### Prefix sums

When to use: many range-sum queries on a static array, or "subarray with sum k" (prefix sum + hash map of earlier prefixes). Mechanism in [[Arrays & Hashing]].

```java
import java.util.*;

class Solution {
    int[] prefix(int[] a) {                              // pre[i] = sum of a[0..i-1], pre[0] = 0
        int[] pre = new int[a.length + 1];
        for (int i = 0; i < a.length; i++) pre[i + 1] = pre[i] + a[i];
        return pre;                                      // sum(l..r) = pre[r+1] - pre[l]
    }

    int[] productExceptSelf(int[] a) {                   // prefix product from the left, suffix from the right
        int n = a.length;
        int[] out = new int[n];
        out[0] = 1;
        for (int i = 1; i < n; i++) out[i] = out[i - 1] * a[i - 1];
        int suffix = 1;
        for (int i = n - 1; i >= 0; i--) { out[i] *= suffix; suffix *= a[i]; }
        return out;
    }
}
```

**Time** O(n) build, O(1) per query · **Space** O(n)

Solves: [[LC 238 Product of Array Except Self]]; the prefix-plus-hash-map form ("subarray sum equals k": store each prefix sum in a map, look up `prefix - k`) is not in the 150 but is the most common follow-up to [[LC 1 Two Sum]].

### Monotonic stack

When to use: "next greater / smaller element", "how many days until warmer", histogram areas. The stack holds indices whose answer is still unknown, kept monotone so each push pops everything it answers. Mechanism in [[Stack & Monotonic Stack]].

```java
import java.util.*;

class Solution {
    int[] dailyTemperatures(int[] t) {                   // days until a warmer day
        int[] ans = new int[t.length];
        Deque<Integer> st = new ArrayDeque<>();          // indices with strictly decreasing temps
        for (int i = 0; i < t.length; i++) {
            while (!st.isEmpty() && t[st.peek()] < t[i]) {
                int j = st.pop();
                ans[j] = i - j;                          // i is j's next greater
            }
            st.push(i);
        }
        return ans;                                      // unanswered indices stay 0
    }

    int largestRectangle(int[] h) {                      // histogram
        int best = 0;
        Deque<Integer> st = new ArrayDeque<>();          // indices with increasing heights
        for (int i = 0; i <= h.length; i++) {
            int cur = i == h.length ? 0 : h[i];          // sentinel 0 flushes the stack at the end
            while (!st.isEmpty() && h[st.peek()] >= cur) {
                int height = h[st.pop()];
                int left = st.isEmpty() ? -1 : st.peek(); // previous smaller index
                best = Math.max(best, height * (i - left - 1));
            }
            st.push(i);
        }
        return best;
    }
}
```

**Time** O(n) — every index is pushed and popped once · **Space** O(n)

Solves: [[LC 739 Daily Temperatures]], [[LC 84 Largest Rectangle in Histogram]], [[LC 853 Car Fleet]] (sort by position, stack of arrival times), [[LC 42 Trapping Rain Water]] (stack form), [[LC 155 Min Stack]] (a stack of running minima).

### Monotonic deque

When to use: max or min of every sliding window in O(n). Front holds the index of the current max; expire from the front, evict dominated candidates from the back.

```java
import java.util.*;

class Solution {
    int[] maxSlidingWindow(int[] a, int k) {
        int n = a.length;
        int[] out = new int[n - k + 1];
        Deque<Integer> dq = new ArrayDeque<>();          // indices, values decreasing front to back
        for (int r = 0; r < n; r++) {
            while (!dq.isEmpty() && a[dq.peekLast()] <= a[r]) dq.pollLast();   // smaller ones can never win
            dq.offerLast(r);
            if (dq.peekFirst() <= r - k) dq.pollFirst(); // front fell out of the window
            if (r >= k - 1) out[r - k + 1] = a[dq.peekFirst()];
        }
        return out;
    }
}
```

**Time** O(n) · **Space** O(k)

Solves: [[LC 239 Sliding Window Maximum]].

## Binary search — three templates

Mechanism and invariants in [[Binary Search]]. The three shapes differ only in what the loop invariant promises.

### Exact match

When to use: sorted array, "find x". `lo <= hi`, return on hit.

```java
import java.util.*;

class Solution {
    int search(int[] a, int target) {
        int lo = 0, hi = a.length - 1;
        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2;                // never (lo + hi) / 2: overflow
            if (a[mid] == target) return mid;
            if (a[mid] < target) lo = mid + 1; else hi = mid - 1;
        }
        return -1;                                       // lo is the insertion point here
    }

    boolean searchMatrix(int[][] m, int target) {        // treat the matrix as one sorted array
        int rows = m.length, cols = m[0].length, lo = 0, hi = rows * cols - 1;
        while (lo <= hi) {
            int mid = lo + (hi - lo) / 2, v = m[mid / cols][mid % cols];
            if (v == target) return true;
            if (v < target) lo = mid + 1; else hi = mid - 1;
        }
        return false;
    }
}
```

**Time** O(log n) · **Space** O(1)

Solves: [[LC 704 Binary Search]], [[LC 74 Search a 2D Matrix]].

### First true (lower bound)

When to use: a predicate that is `false…false true…true` over the index range; you want the first `true`. Rotated-array problems are this with a cleverly chosen predicate. `lo < hi`, `hi = mid` on true, `lo = mid + 1` on false; never skip `mid` on the side that might hold the answer.

```java
import java.util.*;

class Solution {
    int firstTrue(int[] a, int x) {                      // first index with a[i] >= x; a.length if none
        int lo = 0, hi = a.length;                       // hi is one past the end: "none" is a valid answer
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (a[mid] >= x) hi = mid;                   // mid could be the answer: keep it
            else lo = mid + 1;                           // mid is false: answer is strictly right
        }
        return lo;
    }

    int findMinRotated(int[] a) {                        // predicate: a[mid] <= a[last]
        int lo = 0, hi = a.length - 1;
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (a[mid] <= a[hi]) hi = mid;               // min is at mid or to its left
            else lo = mid + 1;                           // rotation point is to the right
        }
        return a[lo];
    }
}
```

**Time** O(log n) · **Space** O(1)

Solves: [[LC 153 Find Minimum in Rotated Sorted Array]], [[LC 33 Search in Rotated Sorted Array]] (find the pivot, then exact-match in the right half), [[LC 981 Time Based Key-Value Store]] (last timestamp ≤ query — the mirror "last true"), [[LC 4 Median of Two Sorted Arrays]] (binary search the partition size).

### On the answer

When to use: "minimum k such that it is feasible", feasibility is monotone in k and checkable in O(n). Binary search the *answer space*, not the array.

```java
import java.util.*;

class Solution {
    int minEatingSpeed(int[] piles, int h) {             // smallest speed that finishes in h hours
        int lo = 1, hi = Arrays.stream(piles).max().getAsInt();
        while (lo < hi) {
            int mid = lo + (hi - lo) / 2;
            if (canFinish(piles, h, mid)) hi = mid; else lo = mid + 1;
        }
        return lo;
    }

    private boolean canFinish(int[] piles, int h, int k) {
        long hours = 0;                                  // long: sum of ceilings can exceed int
        for (int p : piles) hours += (p + k - 1) / k;    // integer ceiling
        return hours <= h;
    }
}
```

**Time** O(n log(max)) · **Space** O(1)

Solves: [[LC 875 Koko Eating Bananas]], [[LC 778 Swim in Rising Water]] (feasibility = BFS with a threshold; Dijkstra variant also works), and the "capacity to ship packages" family outside the 150.

## Linked lists

Each block defines its own `ListNode`; in an interview the class is given, so skip that line.

### Fast and slow pointers

When to use: cycle detection, middle of the list, k-th from the end, and any "array as a linked list" problem (values as next-pointers). Mechanism in [[Linked Lists]].

```java
import java.util.*;

class ListNode { int val; ListNode next; ListNode(int v) { val = v; } }

class Solution {
    boolean hasCycle(ListNode head) {
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) {      // fast is the one that can hit null
            slow = slow.next;
            fast = fast.next.next;
            if (slow == fast) return true;
        }
        return false;
    }

    ListNode middle(ListNode head) {                     // second middle for even length
        ListNode slow = head, fast = head;
        while (fast != null && fast.next != null) { slow = slow.next; fast = fast.next.next; }
        return slow;
    }

    ListNode removeNthFromEnd(ListNode head, int n) {
        ListNode dummy = new ListNode(0);                // dummy head: removing the first node is no special case
        dummy.next = head;
        ListNode lead = dummy, trail = dummy;
        for (int i = 0; i <= n; i++) lead = lead.next;   // gap of n+1 so trail stops before the target
        while (lead != null) { lead = lead.next; trail = trail.next; }
        trail.next = trail.next.next;
        return dummy.next;
    }

    int findDuplicate(int[] nums) {                      // Floyd on the implicit list i -> nums[i]
        int slow = nums[0], fast = nums[0];
        do { slow = nums[slow]; fast = nums[nums[fast]]; } while (slow != fast);
        slow = nums[0];                                  // phase 2: both at speed 1 meet at the cycle entry
        while (slow != fast) { slow = nums[slow]; fast = nums[fast]; }
        return slow;
    }
}
```

**Time** O(n) · **Space** O(1)

Solves: [[LC 141 Linked List Cycle]], [[LC 287 Find the Duplicate Number]], [[LC 19 Remove Nth Node From End of List]], [[LC 143 Reorder List]] (middle, reverse the second half, merge), [[LC 202 Happy Number]] (cycle on the digit-square sequence).

### In-place reversal

When to use: reverse a list, reverse in groups of k, reverse the second half. Three pointers, one line per link.

```java
import java.util.*;

class ListNode { int val; ListNode next; ListNode(int v) { val = v; } }

class Solution {
    ListNode reverse(ListNode head) {
        ListNode prev = null, cur = head;
        while (cur != null) {
            ListNode next = cur.next;                    // save before you overwrite
            cur.next = prev;
            prev = cur;
            cur = next;
        }
        return prev;                                     // prev is the new head
    }

    ListNode reverseKGroup(ListNode head, int k) {
        ListNode dummy = new ListNode(0);
        dummy.next = head;
        ListNode groupPrev = dummy;
        while (true) {
            ListNode kth = groupPrev;
            for (int i = 0; i < k && kth != null; i++) kth = kth.next;
            if (kth == null) break;                      // fewer than k left: leave them
            ListNode groupNext = kth.next, prev = groupNext, cur = groupPrev.next;
            while (cur != groupNext) {                   // reverse exactly k nodes onto groupNext
                ListNode next = cur.next; cur.next = prev; prev = cur; cur = next;
            }
            ListNode first = groupPrev.next;             // old first becomes the group's last
            groupPrev.next = kth;
            groupPrev = first;
        }
        return dummy.next;
    }
}
```

**Time** O(n) · **Space** O(1)

Solves: [[LC 206 Reverse Linked List]], [[LC 25 Reverse Nodes in k-Group]], [[LC 143 Reorder List]], [[LC 2 Add Two Numbers]] (no reversal needed — digits are already least-significant first; the template is the dummy head + carry loop), [[LC 21 Merge Two Sorted Lists]] and [[LC 23 Merge k Sorted Lists]] (dummy head + heap).

## Graphs

Mechanism in [[Graphs]]; the weighted algorithms in [[Advanced Graphs]].

### Matrix directions

```java
// snippet
int[][] DIRS = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};          // 4-neighbour
int[][] DIRS8 = {{1,0},{-1,0},{0,1},{0,-1},{1,1},{1,-1},{-1,1},{-1,-1}};
int[] dr = {0, 1, 0, -1}, dc = {1, 0, -1, 0};            // spiral order: right, down, left, up
for (int[] d : DIRS) {
    int nr = r + d[0], nc = c + d[1];
    if (nr < 0 || nr >= rows || nc < 0 || nc >= cols) continue;   // bounds first, then the cell test
}
```

Used by every grid problem: [[LC 200 Number of Islands]], [[LC 54 Spiral Matrix]] (turn when the next cell is out of bounds or visited), [[LC 48 Rotate Image]] (transpose, then reverse each row), [[LC 73 Set Matrix Zeroes]] (first row/column as the marker).

### BFS on a grid

When to use: shortest path in an unweighted grid, "minimum time for something to spread" (multi-source: seed the queue with every source at distance 0).

```java
import java.util.*;

class Solution {
    int orangesRotting(int[][] g) {                      // multi-source BFS; minutes until all rotten
        int rows = g.length, cols = g[0].length, fresh = 0, minutes = 0;
        Deque<int[]> q = new ArrayDeque<>();
        for (int r = 0; r < rows; r++)
            for (int c = 0; c < cols; c++) {
                if (g[r][c] == 2) q.offer(new int[]{r, c});
                else if (g[r][c] == 1) fresh++;
            }
        int[][] dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
        while (!q.isEmpty() && fresh > 0) {
            int size = q.size();                         // one layer = one minute
            for (int i = 0; i < size; i++) {
                int[] cur = q.poll();
                for (int[] d : dirs) {
                    int nr = cur[0] + d[0], nc = cur[1] + d[1];
                    if (nr < 0 || nr >= rows || nc < 0 || nc >= cols || g[nr][nc] != 1) continue;
                    g[nr][nc] = 2;                       // mark when enqueued, not when dequeued
                    fresh--;
                    q.offer(new int[]{nr, nc});
                }
            }
            minutes++;
        }
        return fresh == 0 ? minutes : -1;
    }
}
```

**Time** O(rows · cols) · **Space** O(rows · cols)

Solves: [[LC 994 Rotting Oranges]], [[LC 286 Walls and Gates]] (seed with all gates), [[LC 200 Number of Islands]] (BFS form), [[LC 778 Swim in Rising Water]] (BFS as the feasibility check inside binary search on the answer).

### BFS on an adjacency list

When to use: shortest path by edge count, level-by-level processing, "fewest transformations".

```java
import java.util.*;

class Solution {
    int[] bfsDistances(int n, List<List<Integer>> adj, int src) {
        int[] dist = new int[n];
        Arrays.fill(dist, -1);
        dist[src] = 0;
        Deque<Integer> q = new ArrayDeque<>();
        q.offer(src);
        while (!q.isEmpty()) {
            int u = q.poll();
            for (int v : adj.get(u)) {
                if (dist[v] != -1) continue;             // visited test doubles as the distance array
                dist[v] = dist[u] + 1;
                q.offer(v);
            }
        }
        return dist;
    }

    List<List<Integer>> buildAdj(int n, int[][] edges, boolean directed) {
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
        for (int[] e : edges) {
            adj.get(e[0]).add(e[1]);
            if (!directed) adj.get(e[1]).add(e[0]);
        }
        return adj;
    }
}
```

**Time** O(V + E) · **Space** O(V)

Solves: [[LC 127 Word Ladder]] (neighbours = wildcard patterns `h*t`), [[LC 133 Clone Graph]] (BFS with a `Map<Node, Node>` of copies), [[LC 102 Binary Tree Level Order Traversal]] and [[LC 199 Binary Tree Right Side View]] (the `size` loop on a tree).

### DFS, recursive and iterative

When to use: components, flood fill, reachability, anything where the *order* of visiting does not matter. Recursive for clarity; iterative when depth can exceed ~10⁴ frames (a 1000×1000 grid of all land).

```java
import java.util.*;

class Solution {
    int numIslands(char[][] g) {
        int count = 0;
        for (int r = 0; r < g.length; r++)
            for (int c = 0; c < g[0].length; c++)
                if (g[r][c] == '1') { count++; sink(g, r, c); }
        return count;
    }

    private void sink(char[][] g, int r, int c) {        // recursive flood fill, marks in place
        if (r < 0 || r >= g.length || c < 0 || c >= g[0].length || g[r][c] != '1') return;
        g[r][c] = '0';
        sink(g, r + 1, c); sink(g, r - 1, c); sink(g, r, c + 1); sink(g, r, c - 1);
    }

    int maxAreaIterative(int[][] g) {                    // explicit stack; no recursion-depth risk
        int rows = g.length, cols = g[0].length, best = 0;
        boolean[][] seen = new boolean[rows][cols];
        int[][] dirs = {{1, 0}, {-1, 0}, {0, 1}, {0, -1}};
        for (int r = 0; r < rows; r++)
            for (int c = 0; c < cols; c++) {
                if (g[r][c] == 0 || seen[r][c]) continue;
                int area = 0;
                Deque<int[]> st = new ArrayDeque<>();
                st.push(new int[]{r, c});
                seen[r][c] = true;                       // mark on push, exactly like BFS
                while (!st.isEmpty()) {
                    int[] cur = st.pop();
                    area++;
                    for (int[] d : dirs) {
                        int nr = cur[0] + d[0], nc = cur[1] + d[1];
                        if (nr < 0 || nr >= rows || nc < 0 || nc >= cols || g[nr][nc] == 0 || seen[nr][nc]) continue;
                        seen[nr][nc] = true;
                        st.push(new int[]{nr, nc});
                    }
                }
                best = Math.max(best, area);
            }
        return best;
    }
}
```

**Time** O(rows · cols) · **Space** O(rows · cols) worst-case stack

Solves: [[LC 200 Number of Islands]], [[LC 695 Max Area of Island]], [[LC 417 Pacific Atlantic Water Flow]] (DFS *from the oceans inward*), [[LC 130 Surrounded Regions]] (DFS from the border, then flip), [[LC 133 Clone Graph]].

### Topological sort — Kahn and DFS

When to use: prerequisites, build order, "is this schedule possible" (cycle detection in a directed graph), alien dictionary. Kahn gives the order and detects cycles by counting; DFS with three colours detects cycles and yields the reverse post-order.

```java
import java.util.*;

class Solution {
    int[] kahn(int n, int[][] prereq) {                  // prereq[i] = {course, before}; order or empty on cycle
        List<List<Integer>> adj = new ArrayList<>();
        int[] indeg = new int[n];
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
        for (int[] p : prereq) { adj.get(p[1]).add(p[0]); indeg[p[0]]++; }
        Deque<Integer> q = new ArrayDeque<>();
        for (int i = 0; i < n; i++) if (indeg[i] == 0) q.offer(i);
        int[] order = new int[n];
        int k = 0;
        while (!q.isEmpty()) {
            int u = q.poll();
            order[k++] = u;
            for (int v : adj.get(u)) if (--indeg[v] == 0) q.offer(v);
        }
        return k == n ? order : new int[0];              // k < n means a cycle swallowed some nodes
    }

    private static final int WHITE = 0, GRAY = 1, BLACK = 2;

    boolean canFinishDfs(int n, int[][] prereq) {
        List<List<Integer>> adj = new ArrayList<>();
        for (int i = 0; i < n; i++) adj.add(new ArrayList<>());
        for (int[] p : prereq) adj.get(p[1]).add(p[0]);
        int[] colour = new int[n];
        for (int i = 0; i < n; i++) if (colour[i] == WHITE && hasCycle(i, adj, colour)) return false;
        return true;
    }

    private boolean hasCycle(int u, List<List<Integer>> adj, int[] colour) {
        colour[u] = GRAY;                                // on the current path
        for (int v : adj.get(u)) {
            if (colour[v] == GRAY) return true;          // back edge: cycle
            if (colour[v] == WHITE && hasCycle(v, adj, colour)) return true;
        }
        colour[u] = BLACK;                               // finished; push to a stack here for the order
        return false;
    }
}
```

**Time** O(V + E) · **Space** O(V + E)

Solves: [[LC 207 Course Schedule]], [[LC 210 Course Schedule II]], [[LC 269 Alien Dictionary]] (build edges from adjacent words, then Kahn).

### Union-Find

When to use: connectivity as edges arrive, redundant edge, "is it a tree", Kruskal. Full derivation in [[Union-Find]].

```java
import java.util.*;

class DSU {
    private final int[] parent, size;
    int components;

    DSU(int n) {
        parent = new int[n]; size = new int[n]; components = n;
        for (int i = 0; i < n; i++) { parent[i] = i; size[i] = 1; }
    }

    int find(int x) {
        while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; }   // path halving
        return x;
    }

    boolean union(int a, int b) {                        // false if already connected (= this edge closes a cycle)
        int ra = find(a), rb = find(b);
        if (ra == rb) return false;
        if (size[ra] < size[rb]) { int t = ra; ra = rb; rb = t; }
        parent[rb] = ra;
        size[ra] += size[rb];
        components--;
        return true;
    }
}

class Solution {
    int[] findRedundantConnection(int[][] edges) {
        DSU dsu = new DSU(edges.length + 1);             // nodes are 1-indexed
        for (int[] e : edges) if (!dsu.union(e[0], e[1])) return e;
        return new int[0];
    }
}
```

**Time** O(α(n)) per operation, effectively O(1) · **Space** O(n)

Solves: [[LC 684 Redundant Connection]], [[LC 323 Number of Connected Components in an Undirected Graph]] (`components` after all unions), [[LC 261 Graph Valid Tree]] (`n - 1` edges and no failed union), [[LC 1584 Min Cost to Connect All Points]] (Kruskal).

### Dijkstra

When to use: shortest path with non-negative weights. No decrease-key in `PriorityQueue`: push duplicates, skip stale entries on poll.

```java
import java.util.*;

class Solution {
    int[] dijkstra(int n, List<List<int[]>> adj, int src) {   // adj.get(u) = list of {v, w}
        int[] dist = new int[n];
        Arrays.fill(dist, Integer.MAX_VALUE);
        dist[src] = 0;
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> Integer.compare(a[1], b[1]));  // {node, dist}
        pq.offer(new int[]{src, 0});
        while (!pq.isEmpty()) {
            int[] cur = pq.poll();
            int u = cur[0], d = cur[1];
            if (d > dist[u]) continue;                   // stale entry: a shorter path was already settled
            for (int[] e : adj.get(u)) {
                int v = e[0], nd = d + e[1];
                if (nd < dist[v]) { dist[v] = nd; pq.offer(new int[]{v, nd}); }
            }
        }
        return dist;                                     // MAX_VALUE = unreachable
    }
}
```

**Time** O((V + E) log V) · **Space** O(V + E)

Solves: [[LC 743 Network Delay Time]] (answer = max of `dist`, −1 if any unreachable), [[LC 778 Swim in Rising Water]] (minimise the *max* edge: relax with `Math.max(d, grid[nr][nc])`), [[LC 787 Cheapest Flights Within K Stops]] (Dijkstra breaks on the stop limit — use Bellman-Ford with k+1 rounds on a *copy* of `dist`).

### Prim

When to use: MST on a dense graph given as points (every pair is an edge); grows one tree from a start node. Lazy version with a heap of candidate edges.

```java
import java.util.*;

class Solution {
    int minCostConnectPoints(int[][] p) {               // Manhattan distances, complete graph
        int n = p.length, cost = 0, taken = 0;
        boolean[] inTree = new boolean[n];
        PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> Integer.compare(a[0], b[0]));  // {weight, node}
        pq.offer(new int[]{0, 0});
        while (taken < n) {
            int[] cur = pq.poll();
            int w = cur[0], u = cur[1];
            if (inTree[u]) continue;                     // lazy deletion of stale candidates
            inTree[u] = true;
            cost += w;
            taken++;
            for (int v = 0; v < n; v++)
                if (!inTree[v]) pq.offer(new int[]{Math.abs(p[u][0] - p[v][0]) + Math.abs(p[u][1] - p[v][1]), v});
        }
        return cost;
    }
}
```

**Time** O(n² log n) on a complete graph, O(E log V) in general · **Space** O(E)

Solves: [[LC 1584 Min Cost to Connect All Points]].

### Kruskal

When to use: MST from an edge list; sort edges by weight, union the ones that connect new components. Pairs naturally with the DSU above.

```java
import java.util.*;

class Solution {
    int kruskal(int n, int[][] edges) {                  // edges[i] = {u, v, w}; -1 if not connected
        Arrays.sort(edges, (a, b) -> Integer.compare(a[2], b[2]));
        int[] parent = new int[n];
        for (int i = 0; i < n; i++) parent[i] = i;
        int cost = 0, used = 0;
        for (int[] e : edges) {
            int ra = find(parent, e[0]), rb = find(parent, e[1]);
            if (ra == rb) continue;                      // would close a cycle
            parent[ra] = rb;
            cost += e[2];
            if (++used == n - 1) break;                  // a spanning tree has exactly n-1 edges
        }
        return used == n - 1 ? cost : -1;
    }

    private int find(int[] parent, int x) {
        while (parent[x] != x) { parent[x] = parent[parent[x]]; x = parent[x]; }
        return x;
    }
}
```

**Time** O(E log E) for the sort · **Space** O(V)

Solves: [[LC 1584 Min Cost to Connect All Points]] (build all n(n−1)/2 edges first).

## Trees and tries

Tree traversals (recursive DFS, BFS with the `size` loop) are covered by the DFS/BFS templates above and in [[Trees]]; the one tree-specific template worth memorising is the trie.

### Trie

When to use: prefix queries, "does any word start with", word search on a board with a dictionary (prune by prefix), wildcard search. Mechanism in [[Tries]].

```java
import java.util.*;

class Trie {
    private static class Node {
        Node[] next = new Node[26];                      // lower-case letters only; use a HashMap for wider alphabets
        boolean end;
        String word;                                     // set at the terminal node: lets Word Search II emit words
    }

    private final Node root = new Node();

    void insert(String w) {
        Node cur = root;
        for (char c : w.toCharArray()) {
            int i = c - 'a';
            if (cur.next[i] == null) cur.next[i] = new Node();
            cur = cur.next[i];
        }
        cur.end = true;
        cur.word = w;
    }

    boolean search(String w) { Node n = walk(w); return n != null && n.end; }

    boolean startsWith(String p) { return walk(p) != null; }

    private Node walk(String s) {
        Node cur = root;
        for (char c : s.toCharArray()) {
            cur = cur.next[c - 'a'];
            if (cur == null) return null;
        }
        return cur;
    }

    boolean searchWithDots(String w) { return dfs(root, w, 0); }  // '.' matches any single letter

    private boolean dfs(Node n, String w, int i) {
        if (n == null) return false;
        if (i == w.length()) return n.end;
        char c = w.charAt(i);
        if (c != '.') return dfs(n.next[c - 'a'], w, i + 1);
        for (Node child : n.next) if (child != null && dfs(child, w, i + 1)) return true;
        return false;
    }
}
```

**Time** O(L) per insert / search for a word of length L; wildcard search O(26^dots · L) · **Space** O(total characters · 26 references)

Solves: [[LC 208 Implement Trie Prefix Tree]], [[LC 211 Design Add and Search Words Data Structure]], [[LC 212 Word Search II]] (DFS the board while walking the trie; remove found words to prune).

## Heaps

Mechanism in [[Heaps & Priority Queues]].

### Top-k with a size-k heap

When to use: "k largest / most frequent / closest" from n items. Keep a **min**-heap of size k for the k largest; the root is the k-th largest. For a stream, keep the heap alive between calls.

```java
import java.util.*;

class Solution {
    int findKthLargest(int[] nums, int k) {
        PriorityQueue<Integer> pq = new PriorityQueue<>();   // min-heap holding the k largest seen so far
        for (int x : nums) {
            pq.offer(x);
            if (pq.size() > k) pq.poll();                // evict the smallest: it cannot be in the top k
        }
        return pq.peek();
    }

    int[] topKFrequent(int[] nums, int k) {
        Map<Integer, Integer> freq = new HashMap<>();
        for (int x : nums) freq.merge(x, 1, Integer::sum);
        PriorityQueue<Map.Entry<Integer, Integer>> pq =
                new PriorityQueue<>((a, b) -> Integer.compare(a.getValue(), b.getValue()));
        for (Map.Entry<Integer, Integer> e : freq.entrySet()) {
            pq.offer(e);
            if (pq.size() > k) pq.poll();
        }
        int[] out = new int[k];
        for (int i = k - 1; i >= 0; i--) out[i] = pq.poll().getKey();   // poll order is ascending
        return out;
    }
}
```

**Time** O(n log k) · **Space** O(k)

Solves: [[LC 215 Kth Largest Element in an Array]], [[LC 703 Kth Largest Element in a Stream]], [[LC 347 Top K Frequent Elements]] (bucket sort is the O(n) alternative), [[LC 973 K Closest Points to Origin]] (max-heap on squared distance), [[LC 1046 Last Stone Weight]] (max-heap simulation), [[LC 621 Task Scheduler]] (max-heap of counts + cooldown queue), [[LC 355 Design Twitter]] (k-way merge of feeds), [[LC 23 Merge k Sorted Lists]] (heap of list heads).

### Two heaps

When to use: running median, or any "split the stream into a lower half and an upper half" question. Max-heap for the low half, min-heap for the high half, sizes differ by at most one.

```java
import java.util.*;

class MedianFinder {
    private final PriorityQueue<Integer> low = new PriorityQueue<>(Collections.reverseOrder());  // max-heap
    private final PriorityQueue<Integer> high = new PriorityQueue<>();                           // min-heap

    void addNum(int x) {
        low.offer(x);                                    // always push to low, then rebalance
        high.offer(low.poll());                          // the largest of low moves up: keeps low <= high
        if (high.size() > low.size()) low.offer(high.poll());   // keep low the bigger half (or equal)
    }

    double findMedian() {
        if (low.size() > high.size()) return low.peek();
        return (low.peek() + high.peek()) / 2.0;
    }
}
```

**Time** O(log n) add, O(1) median · **Space** O(n)

Solves: [[LC 295 Find Median from Data Stream]]. For a *sliding-window* median you need removals, which means two `TreeMap<Integer, Integer>` counts instead of heaps.

## Backtracking

When to use: enumerate all subsets / permutations / combinations, or search for one valid assignment with pruning. Choose, explore, unchoose. Mechanism in [[Backtracking]].

```java
import java.util.*;

class Solution {
    List<List<Integer>> subsets(int[] a) {               // 2^n subsets; sort first + skip dups for Subsets II
        List<List<Integer>> out = new ArrayList<>();
        backtrackSubsets(a, 0, new ArrayList<>(), out);
        return out;
    }

    private void backtrackSubsets(int[] a, int start, List<Integer> path, List<List<Integer>> out) {
        out.add(new ArrayList<>(path));                  // every node of the recursion tree is a subset
        for (int i = start; i < a.length; i++) {
            if (i > start && a[i] == a[i - 1]) continue; // dedupe (requires sorted input); harmless otherwise
            path.add(a[i]);                              // choose
            backtrackSubsets(a, i + 1, path, out);       // explore (i + 1: each element at most once)
            path.remove(path.size() - 1);                // unchoose
        }
    }

    List<List<Integer>> permutations(int[] a) {          // n! permutations
        List<List<Integer>> out = new ArrayList<>();
        backtrackPerms(a, new boolean[a.length], new ArrayList<>(), out);
        return out;
    }

    private void backtrackPerms(int[] a, boolean[] used, List<Integer> path, List<List<Integer>> out) {
        if (path.size() == a.length) { out.add(new ArrayList<>(path)); return; }
        for (int i = 0; i < a.length; i++) {
            if (used[i]) continue;
            used[i] = true; path.add(a[i]);
            backtrackPerms(a, used, path, out);
            path.remove(path.size() - 1); used[i] = false;
        }
    }

    List<List<Integer>> combinationSum(int[] cands, int target) {   // reuse allowed: recurse with i, not i + 1
        List<List<Integer>> out = new ArrayList<>();
        Arrays.sort(cands);
        backtrackCombos(cands, 0, target, new ArrayList<>(), out);
        return out;
    }

    private void backtrackCombos(int[] c, int start, int remain, List<Integer> path, List<List<Integer>> out) {
        if (remain == 0) { out.add(new ArrayList<>(path)); return; }
        for (int i = start; i < c.length && c[i] <= remain; i++) {   // sorted input makes this prune valid
            path.add(c[i]);
            backtrackCombos(c, i, remain - c[i], path, out);         // i again: unlimited reuse
            path.remove(path.size() - 1);
        }
    }
}
```

**Time** O(n · 2ⁿ) subsets, O(n · n!) permutations, exponential with pruning for combination sum · **Space** O(n) recursion depth plus the output

Solves: [[LC 78 Subsets]], [[LC 90 Subsets II]], [[LC 46 Permutations]], [[LC 39 Combination Sum]], [[LC 40 Combination Sum II]] (`i + 1` and the dedupe line), [[LC 17 Letter Combinations of a Phone Number]], [[LC 79 Word Search]] (grid DFS with mark/unmark), [[LC 131 Palindrome Partitioning]], [[LC 51 N-Queens]] (sets for columns and both diagonals).

## Dynamic programming

Mechanism and the five 1-D shapes in [[Dynamic Programming]]; grids, strings and knapsacks in [[2-D Dynamic Programming]]; the recognition map in [[DP Pattern Catalogue]].

### 1-D DP: memo → table → rolling variables

When to use: the answer for `i` depends on a fixed number of smaller indices. Write the recursion, memoise it, then flip it into a loop; finally keep only the last few values.

```java
import java.util.*;

class Solution {
    private Integer[] memo;

    int robMemo(int[] h) {                               // step 1: recursion + memo (top-down)
        memo = new Integer[h.length];
        return best(h, 0);
    }

    private int best(int[] h, int i) {                   // max loot from house i onwards
        if (i >= h.length) return 0;
        if (memo[i] != null) return memo[i];
        return memo[i] = Math.max(h[i] + best(h, i + 2), best(h, i + 1));   // rob i, or skip it
    }

    int robTable(int[] h) {                              // step 2: bottom-up table
        int n = h.length;
        int[] dp = new int[n + 2];                       // dp[i] = best from i onwards; dp[n] = dp[n+1] = 0
        for (int i = n - 1; i >= 0; i--) dp[i] = Math.max(h[i] + dp[i + 2], dp[i + 1]);
        return dp[0];
    }

    int robRolling(int[] h) {                            // step 3: O(1) space, only dp[i+1] and dp[i+2] matter
        int next = 0, nextNext = 0;
        for (int i = h.length - 1; i >= 0; i--) {
            int cur = Math.max(h[i] + nextNext, next);
            nextNext = next;
            next = cur;
        }
        return next;
    }

    int coinChange(int[] coins, int amount) {            // min coins; the "unbounded, order irrelevant" 1-D shape
        int INF = amount + 1;                            // not MAX_VALUE: we add 1 to it
        int[] dp = new int[amount + 1];
        Arrays.fill(dp, INF);
        dp[0] = 0;
        for (int a = 1; a <= amount; a++)
            for (int c : coins)
                if (c <= a) dp[a] = Math.min(dp[a], dp[a - c] + 1);
        return dp[amount] == INF ? -1 : dp[amount];
    }
}
```

**Time** O(n) for the robber, O(amount · coins) for coin change · **Space** O(n) → O(1)

Solves: [[LC 70 Climbing Stairs]], [[LC 746 Min Cost Climbing Stairs]], [[LC 198 House Robber]], [[LC 213 House Robber II]] (run twice: without the first house, without the last), [[LC 322 Coin Change]], [[LC 91 Decode Ways]], [[LC 139 Word Break]] (`dp[i]` = prefix of length i is breakable), [[LC 152 Maximum Product Subarray]] (track max *and* min), [[LC 53 Maximum Subarray]] (Kadane is this with one variable), [[LC 5 Longest Palindromic Substring]] and [[LC 647 Palindromic Substrings]] (expand around centre is simpler than DP here).

### 2-D DP: grids and two strings

When to use: two indices define the state — a cell in a grid, or a prefix of each of two strings. Allocate `(m + 1) × (n + 1)` so that row 0 and column 0 are the empty-prefix base cases.

```java
import java.util.*;

class Solution {
    int uniquePaths(int m, int n) {                      // grid: dp[r][c] = ways to reach (r, c)
        int[][] dp = new int[m][n];
        for (int r = 0; r < m; r++)
            for (int c = 0; c < n; c++)
                dp[r][c] = (r == 0 || c == 0) ? 1 : dp[r - 1][c] + dp[r][c - 1];
        return dp[m - 1][n - 1];
    }

    int lcs(String a, String b) {                        // two strings: dp[i][j] = LCS of a[0..i) and b[0..j)
        int m = a.length(), n = b.length();
        int[][] dp = new int[m + 1][n + 1];
        for (int i = 1; i <= m; i++)
            for (int j = 1; j <= n; j++)
                dp[i][j] = a.charAt(i - 1) == b.charAt(j - 1)
                        ? dp[i - 1][j - 1] + 1                           // match: extend the diagonal
                        : Math.max(dp[i - 1][j], dp[i][j - 1]);         // skip one char of either
        return dp[m][n];
    }

    int editDistance(String a, String b) {               // same table, three transitions
        int m = a.length(), n = b.length();
        int[][] dp = new int[m + 1][n + 1];
        for (int i = 0; i <= m; i++) dp[i][0] = i;       // delete everything
        for (int j = 0; j <= n; j++) dp[0][j] = j;       // insert everything
        for (int i = 1; i <= m; i++)
            for (int j = 1; j <= n; j++)
                dp[i][j] = a.charAt(i - 1) == b.charAt(j - 1)
                        ? dp[i - 1][j - 1]
                        : 1 + Math.min(dp[i - 1][j - 1], Math.min(dp[i - 1][j], dp[i][j - 1]));  // replace, delete, insert
        return dp[m][n];
    }
}
```

**Time** O(m · n) · **Space** O(m · n), O(n) with a rolling row

Solves: [[LC 62 Unique Paths]], [[LC 1143 Longest Common Subsequence]], [[LC 72 Edit Distance]], [[LC 97 Interleaving String]], [[LC 115 Distinct Subsequences]], [[LC 10 Regular Expression Matching]], [[LC 329 Longest Increasing Path in a Matrix]] (memoised DFS on the grid — the order comes from the recursion, not from loops), [[LC 312 Burst Balloons]] (interval DP: `dp[l][r]` over the *last* balloon burst), [[LC 309 Best Time to Buy and Sell Stock with Cooldown]] (state machine: index × holding).

### Knapsack, 0/1 and unbounded

When to use: choose items with weights to hit a capacity exactly or maximise value; "can a subset sum to target" is 0/1 knapsack with boolean values. The *only* difference between 0/1 and unbounded is the direction of the inner loop.

```java
import java.util.*;

class Solution {
    boolean canPartition(int[] nums) {                   // 0/1: each item once -> iterate capacity DOWNWARDS
        int sum = 0;
        for (int x : nums) sum += x;
        if (sum % 2 != 0) return false;
        int target = sum / 2;
        boolean[] dp = new boolean[target + 1];          // dp[s] = some subset sums to s
        dp[0] = true;
        for (int x : nums)
            for (int s = target; s >= x; s--)            // downwards so dp[s - x] is still "without x"
                dp[s] = dp[s] || dp[s - x];
        return dp[target];
    }

    int change(int amount, int[] coins) {                // unbounded: reuse allowed -> iterate capacity UPWARDS
        int[] dp = new int[amount + 1];                  // dp[a] = number of combinations making a
        dp[0] = 1;
        for (int c : coins)                              // coins outer: counts combinations, not permutations
            for (int a = c; a <= amount; a++)
                dp[a] += dp[a - c];
        return dp[amount];
    }
}
```

**Time** O(n · capacity) · **Space** O(capacity)

Solves: [[LC 416 Partition Equal Subset Sum]], [[LC 494 Target Sum]] (count subsets with sum `(total + target) / 2`), [[LC 518 Coin Change II]], [[LC 322 Coin Change]] (unbounded, min instead of count).

### Longest increasing subsequence

When to use: "longest chain where each element is bigger than the previous", envelopes, box stacking. Know both: the O(n²) table that interviewers expect first, and the patience-sorting O(n log n) that they ask for as the follow-up.

```java
import java.util.*;

class Solution {
    int lisQuadratic(int[] a) {
        int n = a.length, best = 0;
        int[] dp = new int[n];                           // dp[i] = LIS ending exactly at i
        for (int i = 0; i < n; i++) {
            dp[i] = 1;
            for (int j = 0; j < i; j++) if (a[j] < a[i]) dp[i] = Math.max(dp[i], dp[j] + 1);
            best = Math.max(best, dp[i]);
        }
        return best;
    }

    int lisPatience(int[] a) {
        int[] tails = new int[a.length];                 // tails[k] = smallest tail of an increasing subsequence of length k+1
        int size = 0;
        for (int x : a) {
            int lo = 0, hi = size;                       // first-true binary search: first tail >= x
            while (lo < hi) {
                int mid = lo + (hi - lo) / 2;
                if (tails[mid] >= x) hi = mid; else lo = mid + 1;
            }
            tails[lo] = x;                               // replace (keeps tails small) or append
            if (lo == size) size++;
        }
        return size;                                     // tails is NOT the subsequence itself, only its length
    }
}
```

**Time** O(n²) / O(n log n) · **Space** O(n)

Solves: [[LC 300 Longest Increasing Subsequence]].

## Intervals and greedy

Mechanism in [[Intervals]] and [[Greedy]].

### Interval scheduling

When to use: overlapping intervals — merge them, count the rooms, drop the fewest to make them disjoint. Sort by start to merge; sort by **end** to pick the maximum number of non-overlapping ones.

```java
import java.util.*;

class Solution {
    int[][] merge(int[][] iv) {
        Arrays.sort(iv, (a, b) -> Integer.compare(a[0], b[0]));   // by start
        List<int[]> out = new ArrayList<>();
        for (int[] cur : iv) {
            if (out.isEmpty() || out.get(out.size() - 1)[1] < cur[0]) out.add(cur);   // no overlap: new interval
            else out.get(out.size() - 1)[1] = Math.max(out.get(out.size() - 1)[1], cur[1]);   // extend
        }
        return out.toArray(new int[0][]);
    }

    int eraseOverlapIntervals(int[][] iv) {              // min removals = n - max non-overlapping
        Arrays.sort(iv, (a, b) -> Integer.compare(a[1], b[1]));   // by END: greedy keeps the earliest finisher
        int keep = 0, lastEnd = Integer.MIN_VALUE;
        for (int[] cur : iv)
            if (cur[0] >= lastEnd) { keep++; lastEnd = cur[1]; }
        return iv.length - keep;
    }

    int minMeetingRooms(int[][] iv) {                    // rooms = max overlap; min-heap of end times
        Arrays.sort(iv, (a, b) -> Integer.compare(a[0], b[0]));
        PriorityQueue<Integer> ends = new PriorityQueue<>();
        for (int[] cur : iv) {
            if (!ends.isEmpty() && ends.peek() <= cur[0]) ends.poll();   // a room freed up: reuse it
            ends.offer(cur[1]);
        }
        return ends.size();
    }
}
```

**Time** O(n log n) · **Space** O(n)

Solves: [[LC 56 Merge Intervals]], [[LC 57 Insert Interval]] (three passes: before, overlapping, after), [[LC 435 Non-overlapping Intervals]], [[LC 252 Meeting Rooms]], [[LC 253 Meeting Rooms II]], [[LC 1851 Minimum Interval to Include Each Query]] (sort queries, sweep intervals into a heap keyed by length).

## Bits

Mechanism in [[Bit Manipulation]].

### Bit tricks

```java
// snippet
boolean isSet = (mask >> i & 1) == 1;      // test bit i
mask |= 1 << i;                            // set bit i
mask &= ~(1 << i);                         // clear bit i
mask ^= 1 << i;                            // toggle bit i
int low = n & -n;                          // lowest set bit (Fenwick step)
n &= n - 1;                                // clear the lowest set bit (loop this to count bits)
boolean pow2 = n > 0 && (n & (n - 1)) == 0;
int x = a ^ b;                             // XOR: a ^ a == 0, a ^ 0 == a; the "single number" trick
int sum = a ^ b, carry = (a & b) << 1;     // add without +: loop until carry == 0
int unsignedShift = n >>> 1;               // fills with 0; >> keeps the sign bit
for (int sub = mask; sub > 0; sub = (sub - 1) & mask) { }   // enumerate submasks of mask
```

Solves: [[LC 136 Single Number]], [[LC 191 Number of 1 Bits]], [[LC 338 Counting Bits]] (`dp[i] = dp[i >> 1] + (i & 1)`), [[LC 190 Reverse Bits]] (32 iterations of shift-and-or, with `>>>`), [[LC 268 Missing Number]] (XOR all indices and values), [[LC 371 Sum of Two Integers]], [[LC 7 Reverse Integer]] (overflow check before the multiply, not after).

## Memoisation keys

When to use: top-down DP whose state is a pair or a triple and the ranges are too sparse or too large for an array. Pick the key by cost: `long` packing is fastest, `record` is clearest, `String` is the fallback.

```java
import java.util.*;

class Solution {
    private final Map<Long, Integer> memoLong = new HashMap<>();
    private final Map<State, Integer> memoRecord = new HashMap<>();
    private final Map<String, Boolean> memoString = new HashMap<>();

    private record State(int i, int sum) {}             // value equality + hashCode for free

    int countWays(int[] nums, int i, int remaining) {    // Target Sum, memo on (i, remaining) with a long key
        if (i == nums.length) return remaining == 0 ? 1 : 0;
        long key = ((long) i << 32) | (remaining & 0xffffffffL);   // mask keeps a negative remaining from smearing into i
        Integer cached = memoLong.get(key);
        if (cached != null) return cached;
        int ways = countWays(nums, i + 1, remaining - nums[i]) + countWays(nums, i + 1, remaining + nums[i]);
        memoLong.put(key, ways);
        return ways;
    }

    int countWaysRecord(int[] nums, int i, int remaining) {
        if (i == nums.length) return remaining == 0 ? 1 : 0;
        State key = new State(i, remaining);
        Integer cached = memoRecord.get(key);
        if (cached != null) return cached;
        int ways = countWaysRecord(nums, i + 1, remaining - nums[i]) + countWaysRecord(nums, i + 1, remaining + nums[i]);
        memoRecord.put(key, ways);
        return ways;
    }

    boolean isMatch(String s, String p, int i, int j) {  // regex matching memo with a String key
        String key = i + "," + j;
        Boolean cached = memoString.get(key);
        if (cached != null) return cached;
        boolean ans;
        if (j == p.length()) ans = i == s.length();
        else {
            boolean first = i < s.length() && (p.charAt(j) == '.' || p.charAt(j) == s.charAt(i));
            if (j + 1 < p.length() && p.charAt(j + 1) == '*')
                ans = isMatch(s, p, i, j + 2) || (first && isMatch(s, p, i + 1, j));   // zero copies, or one more
            else ans = first && isMatch(s, p, i + 1, j + 1);
        }
        memoString.put(key, ans);
        return ans;
    }
}
```

**Time** O(number of distinct states · work per state) · **Space** O(number of states)

Solves: [[LC 494 Target Sum]], [[LC 10 Regular Expression Matching]], [[LC 139 Word Break]], [[LC 329 Longest Increasing Path in a Matrix]] (the key is a flattened cell index — an `int[][]` memo is better), [[LC 2013 Detect Squares]] (a `Map<Integer, Integer>` keyed by `x * 1001 + y` for point counts).

## Which five cover half the list

Counting the links above: DFS/BFS (grid, graph, and every tree traversal), the variable sliding window, the three binary-search templates, backtracking, and the 1-D/2-D DP pair together touch roughly 70 of the 150. Learn those five families first; everything else is a week each.

> [!question] Spot the bug
> In the 0/1 knapsack `canPartition`, change the inner loop to `for (int s = x; s <= target; s++)`. It still compiles and still returns `true` for `[1, 5, 11, 5]`. For which input is it now wrong? (Answer: `[3, 1]` — sum 4, target 2, no subset sums to 2, but the upward loop lets the `1` be reused: `dp[1]` becomes true, then `dp[2] = dp[1]` is true. Upwards is the unbounded knapsack.)

## Related

### Siblings in this folder
- [[Java Collections Cheat Sheet]] — the API each template uses
- [[Java Pitfalls in Interviews]] — the ways these templates break when typed from memory

### Where each pattern is explained
- [[Two Pointers]] · [[Sliding Window]] · [[Arrays & Hashing]] · [[Stack & Monotonic Stack]]
- [[Binary Search]] · [[Linked Lists]] · [[Heaps & Priority Queues]]
- [[Graphs]] · [[Advanced Graphs]] · [[Union-Find]] · [[Tries]] · [[Trees]] · [[Backtracking]]
- [[Dynamic Programming]] · [[2-D Dynamic Programming]] · [[DP Pattern Catalogue]] · [[Intervals]] · [[Greedy]] · [[Bit Manipulation]] · [[Math & Geometry]]

### Plans that schedule these templates
- [[8-Week DSA Plan]] — which templates to type from memory in which week
- [[Pattern Recognition Table]] — statement cue → template

## Personal understanding

> [!question] Type it from memory
> Each morning of weeks 1–2 of a topic: close this note, type the week's templates into a scratch file, compile, diff. Record below which lines you got wrong; a template counts as known when it compiles twice in a row from memory.

## Open questions
- [ ] Which template did I last get wrong from memory, and which single line was the mistake?
