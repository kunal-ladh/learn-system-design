---
title: Java Pitfalls in Interviews
type: guide
tags: [guide, java]
aliases: [Java gotchas, Java traps, Interview Java mistakes]
summary: "The twenty-three ways correct-looking Java silently returns the wrong answer under interview conditions — Integer cache, int overflow, remove(int) vs remove(Object), ConcurrentModificationException, PriorityQueue iteration order, substring cost, recursion depth, subtraction comparators, negative modulo and more — each as a failing snippet with the one-line fix, grouped by numbers, collections, strings, recursion, sorting and iteration."
related: ["[[Java Collections Cheat Sheet]]", "[[DSA Patterns in Java]]", "[[Mistakes Log]]", "[[Hashing]]", "[[Sorting]]", "[[Recursion]]"]
created: 2026-11-04
updated: 2026-11-04
---

# Java Pitfalls in Interviews

> [!abstract] In one sentence
> A catalogue of Java behaviours that compile, run, pass the sample input and fail the hidden tests — know each one well enough to see it in your own code before the interviewer does.

**Map:** [[Java Refresher MOC]] → [[DSA Dashboard]]

> [!question] Predict before you read
> `Integer a = 127, b = 127; Integer c = 128, d = 128;` — what do `a == b` and `c == d` return, and why does the answer depend on the number? If you cannot explain the 127, read the Numbers section first.

> [!warning] The top ten, in order of how often they cost people an offer
> 1. `(lo + hi) / 2` overflows — write `lo + (hi - lo) / 2`.
> 2. `(a, b) -> a - b` as a comparator overflows on large values — write `Integer.compare(a, b)`.
> 3. `list.remove(2)` on a `List<Integer>` removes the *index* — write `remove(Integer.valueOf(2))`.
> 4. `Integer` compared with `==` works up to 127 and fails after — write `.equals` or unbox to `int`.
> 5. `int x = map.get(key)` throws NPE on a missing key — write `getOrDefault(key, 0)`.
> 6. Removing from a collection inside a for-each throws `ConcurrentModificationException` — use an `Iterator` or `removeIf`.
> 7. Building a string with `+` in a loop is O(n²) — use `StringBuilder`.
> 8. Recursion 10⁵ deep throws `StackOverflowError` — convert to an explicit stack or a loop.
> 9. `-7 % 3` is `-1`, not `2` — write `Math.floorMod(-7, 3)`.
> 10. Iterating a `PriorityQueue` does not give sorted order — `poll()` in a loop.

Every block below is a fragment (`// snippet`): the failing lines are marked `BUG`, the correction `FIX`.

## Numbers

### Integer cache and `==`

`Integer.valueOf` caches −128..127, so autoboxed small values are the *same object* and `==` happens to work. Above 127 each boxing allocates, and `==` compares references.

```java
// snippet
Integer a = 127, b = 127, c = 128, d = 128;
boolean small = a == b;                 // true, cached
boolean big = c == d;                   // BUG: false — two different objects
boolean fixed = c.equals(d);            // FIX: true
int x = c, y = d;                       // FIX: or unbox and compare ints
```

It hides in `map.get(k1) == map.get(k2)` and in `list.get(i) == list.get(j)`: correct on the sample input with small numbers, wrong on the hidden tests.

### `int` overflow

`int` wraps silently at ±2.1 × 10⁹. The three places it bites: the midpoint of two large indices, a product of two `int`s, and a running sum.

```java
// snippet
int mid = (lo + hi) / 2;                // BUG: lo + hi overflows when both are near MAX_VALUE
int mid2 = lo + (hi - lo) / 2;          // FIX: the difference never overflows
int mid3 = (lo + hi) >>> 1;             // FIX: also correct — unsigned shift reads the wrapped sum as positive

int prod = a * b;                       // BUG: 50_000 * 50_000 wraps negative
long prod2 = (long) a * b;              // FIX: cast ONE operand before the multiply, not the result

int sum = 0;                            // BUG when n * max exceeds 2^31: use long
long sum2 = 0;
```

Rule: before you write `*` on two ints, say the maximum product out loud. If it exceeds 2 × 10⁹, the left-hand side is `long`.

### `Math.abs(Integer.MIN_VALUE)` is negative

There is no positive `int` for 2³¹, so `Math.abs` returns `MIN_VALUE` unchanged. The same trap exists for `-x` and for `Integer.MIN_VALUE / -1`.

```java
// snippet
int bad = Math.abs(Integer.MIN_VALUE);  // BUG: still -2147483648
long ok = Math.abs((long) x);           // FIX: widen first
```

### `%` with negative numbers

Java's `%` keeps the sign of the *dividend*, so `-1 % n` is `-1`, never `n - 1`. Circular indexing and hash buckets break on negative inputs.

```java
// snippet
int prev = (i - 1) % n;                 // BUG: -1 when i == 0 -> ArrayIndexOutOfBounds
int prev2 = Math.floorMod(i - 1, n);    // FIX: always in [0, n)
int prev3 = (i - 1 + n) % n;            // FIX: the manual form, fine when i - 1 >= -n
int q = -7 / 2;                         // -3: integer division truncates toward zero
int q2 = Math.floorDiv(-7, 2);          // -4 if you need "round down"
```

### `>>>` vs `>>`

`>>` is arithmetic: it copies the sign bit, so a negative number shifted right stays negative forever and a `while (n != 0) n >>= 1` loop never terminates. `>>>` is logical: it fills with zeros.

```java
// snippet
int count = 0;
while (n != 0) { count += n & 1; n >>= 1; }     // BUG: infinite loop for negative n
while (n != 0) { count += n & 1; n >>>= 1; }    // FIX: 32 iterations at most
```

### Boxed `Long` and `Integer` are never equal

`Long.valueOf(1).equals(Integer.valueOf(1))` is `false` — `equals` checks the class first. It bites when a `Map<Long, …>` is looked up with an `int` key: the key autoboxes to `Integer` and never matches.

```java
// snippet
Map<Long, Integer> m = new HashMap<>();
m.put(1L, 42);
Integer v = m.get(1);                   // BUG: null — the key boxed to Integer, not Long
Integer v2 = m.get(1L);                 // FIX: match the key type exactly
```

### `Integer` null unboxing

Any expression that needs an `int` from a `null` `Integer` throws `NullPointerException`: `int x = map.get(k)`, `map.get(k) + 1`, `map.get(k) == 3`, `pq.poll()` on an empty heap assigned to an `int`.

```java
// snippet
int count = freq.get(c) + 1;            // BUG: NPE on the first occurrence
int count2 = freq.getOrDefault(c, 0) + 1;   // FIX
freq.merge(c, 1, Integer::sum);         // FIX: one line, no read
```

## Collections

### `Arrays.asList` on an `int[]`

Varargs of `T...` cannot take primitives, so an `int[]` becomes a *single element* of type `int[]`. You get a `List<int[]>` of size 1.

```java
// snippet
int[] a = {3, 1, 2};
List<int[]> wrong = Arrays.asList(a);                           // BUG: one element, the array itself
List<Integer> ok = Arrays.stream(a).boxed().collect(Collectors.toList());   // FIX
List<Integer> ok2 = Arrays.asList(3, 1, 2);                     // FIX: literals box fine (fixed-size list)
```

### `List.remove(int)` vs `remove(Object)`

Overload resolution picks `remove(int index)` for an `int` argument — no boxing needed, so it wins.

```java
// snippet
List<Integer> list = new ArrayList<>(List.of(5, 2, 9));
list.remove(2);                         // BUG: removes index 2 -> [5, 2]
list.remove(Integer.valueOf(2));        // FIX: removes the value 2 -> [5, 9]
```

### `HashMap` and `HashSet` have no order

Iteration order depends on hash codes and capacity, and it changes after a resize. Never rely on it for "first inserted" or "sorted".

```java
// snippet
Map<String, Integer> m = new HashMap<>();
for (String k : m.keySet()) { }         // BUG if the problem wants insertion or sorted order
Map<String, Integer> ordered = new LinkedHashMap<>();   // FIX: insertion order
Map<String, Integer> sorted = new TreeMap<>();          // FIX: key order
```

### `PriorityQueue` iteration is heap order

`toString`, `iterator`, `stream` and the enhanced for-loop walk the backing array, which is a heap, not a sorted list. Only `poll()` gives sorted output.

```java
// snippet
PriorityQueue<Integer> pq = new PriorityQueue<>(List.of(5, 1, 4, 2));
for (int x : pq) { }                    // BUG: 1, 2, 4, 5 is NOT guaranteed — you see 1, 2, 4, 5 or 1, 5, 4, 2 ...
while (!pq.isEmpty()) { int x = pq.poll(); }   // FIX: ascending, destructively
```

### Mutable keys in a `HashMap`

A key's hash is computed on insertion. Mutate the key afterwards and the entry sits in the wrong bucket: `get` returns `null`, `remove` fails, and the map leaks. Arrays are the usual culprit — and `int[]` keys never match anyway because arrays use identity `equals`.

```java
// snippet
Map<int[], Integer> bad = new HashMap<>();
bad.put(new int[]{1, 2}, 1);
Integer v = bad.get(new int[]{1, 2});   // BUG: null — different array object
record Pos(int r, int c) {}
Map<Pos, Integer> ok = new HashMap<>();
ok.put(new Pos(1, 2), 1);
Integer v2 = ok.get(new Pos(1, 2));     // FIX: records have value equals/hashCode
Map<Integer, Integer> ok2 = new HashMap<>();   // FIX: or flatten r * cols + c
```

### `List.of`, `Map.of`, `Set.of` are immutable

They throw `UnsupportedOperationException` on `add`, `remove`, `set`, `put`, and they reject `null`. `Arrays.asList` is the half-way case: `set` works, `add`/`remove` throw.

```java
// snippet
List<Integer> path = List.of(1, 2);
path.add(3);                            // BUG: UnsupportedOperationException
List<Integer> path2 = new ArrayList<>(List.of(1, 2));   // FIX: copy into a mutable list
```

### 2-D array initialisation

`new int[m][n]` allocates `m` rows of `n` zeros — fine. The trap is filling with a non-zero value (`Arrays.fill` is one-dimensional) and creating rows by assignment so that every row is the *same* array.

```java
// snippet
int[][] dp = new int[m][n];
Arrays.fill(dp, -1);                    // BUG: does not compile — fill wants int[], you passed int[][]
for (int[] row : dp) Arrays.fill(row, -1);   // FIX

int[] shared = new int[n];
int[][] grid = new int[m][];
for (int i = 0; i < m; i++) grid[i] = shared;   // BUG: every row is the same array
for (int i = 0; i < m; i++) grid[i] = new int[n];   // FIX
```

### `clone()` of an array is shallow

`int[].clone()` copies the ints — safe. `int[][].clone()` copies the row *references* — both copies share rows, and "snapshot the board before backtracking" silently corrupts the original.

```java
// snippet
int[][] copy = grid.clone();            // BUG: copy[i] == grid[i] for every i
int[][] copy2 = new int[grid.length][];
for (int i = 0; i < grid.length; i++) copy2[i] = grid[i].clone();   // FIX: clone each row
```

## Strings

### `String` compared with `==`

Literals are interned, so `"abc" == "abc"` is `true` and hides the bug. Any computed string — `substring`, `new String`, `StringBuilder.toString`, user input — is a fresh object.

```java
// snippet
String s = sb.toString();
if (s == "abc") { }                     // BUG: false even when the characters match
if (s.equals("abc")) { }                // FIX
if ("abc".equals(s)) { }                // FIX: also null-safe
```

### `substring` is O(n) per call

Since Java 7u6 `substring` copies the characters. Calling it inside an O(n) loop makes the algorithm O(n²), and `set.contains(s.substring(i, j))` in a double loop is O(n³).

```java
// snippet
for (int i = 0; i < n; i++)
    if (dict.contains(s.substring(i, i + k))) { }      // O(n * k) copies: fine for one k, bad in a nested loop
boolean eq = s.regionMatches(i, t, 0, k);              // FIX: compare in place, no allocation
boolean pre = s.startsWith(t, i);                      // FIX: prefix check at an offset
```

For palindrome checks use indices, not `substring(...).equals(reversed)`.

### `char` arithmetic promotes to `int`

`char + int` is an `int`. Assigning it back to a `char` needs a cast, and appending it to a `StringBuilder` appends the *number*.

```java
// snippet
char c = 'a';
char next = c + 1;                      // BUG: does not compile — int cannot narrow to char
char next2 = (char) (c + 1);            // FIX
sb.append(c + 1);                       // BUG: appends "98"
sb.append((char) (c + 1));              // FIX: appends "b"
int idx = c - 'a';                      // fine: this is the int you wanted
```

### String concatenation in a loop

Each `+=` copies the whole string so far: n appends cost O(n²). The JIT does not rescue a loop.

```java
// snippet
String out = "";
for (String p : parts) out += p;        // BUG: O(n^2) — 10^5 parts is seconds, not milliseconds
StringBuilder sb = new StringBuilder();
for (String p : parts) sb.append(p);    // FIX: O(n)
String out2 = String.join("", parts);   // FIX: for a plain join
```

## Recursion

### Recursion depth

The default thread stack holds roughly 10⁴ simple frames (JVM-dependent; LeetCode's is more generous than most laptops). A DFS on a 10⁵-node path graph, a linked-list recursion of 10⁵ nodes, or a flood fill on a 1000 × 1000 all-land grid overflows.

```java
// snippet
void dfs(int u) { seen[u] = true; for (int v : adj.get(u)) if (!seen[v]) dfs(v); }   // BUG at depth 10^5
void dfsIterative(int s) {                                          // FIX: explicit stack
    Deque<Integer> st = new ArrayDeque<>();
    st.push(s); seen[s] = true;
    while (!st.isEmpty()) {
        int u = st.pop();
        for (int v : adj.get(u)) if (!seen[v]) { seen[v] = true; st.push(v); }
    }
}
```

Say it out loud in the interview — "depth can reach n here, so I would write this iteratively in production" — and keep the recursive version if n ≤ 10⁴. The memoised-DP cousin of this trap: a `memo` of type `int[]` initialised to 0 cannot tell "not computed" from "computed as 0"; use `Integer[]` (null = unknown) or fill with −1.

## Sorting

### Sorting primitives descending

`Arrays.sort(int[], Comparator)` does not exist — comparators need objects. Three honest options, none of which is `Collections.reverseOrder()` on an `int[]`.

```java
// snippet
Arrays.sort(nums, Collections.reverseOrder());          // BUG: does not compile for int[]
Arrays.sort(nums); /* then read from nums.length - 1 down */   // FIX 1: iterate backwards
Integer[] boxed = Arrays.stream(nums).boxed().toArray(Integer[]::new);
Arrays.sort(boxed, Collections.reverseOrder());         // FIX 2: box (slower, allocates)
Arrays.sort(nums); for (int i = 0, j = nums.length - 1; i < j; i++, j--) { int t = nums[i]; nums[i] = nums[j]; nums[j] = t; }   // FIX 3: reverse in place
```

### Comparator by subtraction overflows

`a - b` is a valid comparator only when the difference fits in an `int`. With values near ±10⁹ the subtraction wraps, the sign flips, and `TimSort` throws `IllegalArgumentException: Comparison method violates its general contract` — or worse, returns a wrong order quietly.

```java
// snippet
PriorityQueue<int[]> pq = new PriorityQueue<>((a, b) -> a[1] - b[1]);      // BUG when |a[1] - b[1]| > 2^31
PriorityQueue<int[]> ok = new PriorityQueue<>((a, b) -> Integer.compare(a[1], b[1]));   // FIX
Comparator<int[]> ok2 = Comparator.comparingInt(a -> a[1]);                 // FIX: also readable
```

The companion trap is a comparator that is not a total order (`(a, b) -> a < b ? -1 : 1` with no `0`), which also violates the contract.

## Iteration

### `ConcurrentModificationException`

Structurally modifying a collection while a for-each loop is iterating it throws on the *next* `next()` call — sometimes a few iterations later, which makes it look random.

```java
// snippet
for (Integer k : map.keySet()) if (map.get(k) == 0) map.remove(k);   // BUG: CME
map.entrySet().removeIf(e -> e.getValue() == 0);                      // FIX: removeIf
Iterator<Integer> it = list.iterator();
while (it.hasNext()) if (it.next() < 0) it.remove();                  // FIX: iterator's own remove
for (Integer k : new ArrayList<>(map.keySet())) if (map.get(k) == 0) map.remove(k);   // FIX: iterate a copy
```

Adding to a list while iterating it by index (`for (int i = 0; i < list.size(); i++)`) does *not* throw but changes `size()` under you; decide up front whether you want the new elements visited.

### For-each cannot write back

`for (int x : a) x = 0;` changes the copy. Use an index loop to write, and remember that `for (int[] row : grid)` *can* mutate the row's contents because the row is a reference.

```java
// snippet
for (int x : nums) x *= 2;                              // BUG: nums unchanged
for (int i = 0; i < nums.length; i++) nums[i] *= 2;     // FIX
for (int[] row : grid) Arrays.fill(row, -1);            // fine: the row object is shared
```

> [!question] Spot the bug
> `Map<Character, Integer> count = new HashMap<>(); for (char c : s.toCharArray()) count.put(c, count.get(c) + 1);` — it compiles. What happens on the first character, and what is the one-line fix? (NPE from unboxing `null`; `count.merge(c, 1, Integer::sum)`.)

## Related

### Siblings in this folder
- [[Java Collections Cheat Sheet]] — the gotcha column, in table form
- [[DSA Patterns in Java]] — the templates written so that none of these can happen

### Topic notes where the trap lives
- [[Hashing]] — why mutable keys and identity `equals` break a `HashMap`
- [[Sorting]] — comparator contracts, TimSort vs dual-pivot
- [[Recursion]] — stack frames and converting to iteration
- [[Binary Search]] — the midpoint overflow
- [[Bit Manipulation]] — `>>>` vs `>>`, two's complement
- [[Math & Geometry]] — overflow, `floorMod`, fast power

### Where to record the ones you hit
- [[Mistakes Log]] — category "Java API gotcha"

## Personal understanding

> [!question] Which of these have I actually hit?
> Tick the ones you have personally written in the last year. Those are the ones to re-read before every mock.

## Open questions
- [ ] What is the actual recursion depth limit on my laptop's JVM with default `-Xss`, measured?
