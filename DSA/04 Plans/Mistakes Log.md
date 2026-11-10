---
title: Mistakes Log
type: guide
category: Plans
tags: [guide, plan]
aliases: [Error Log, Bug Log]
summary: "One row per mistake, tagged with one of seven categories, so the weekly review can see which kind of error keeps recurring; includes the category definitions, three worked example rows and the questions to ask when a category repeats."
created: 2026-11-04
updated: 2026-11-04
---

# Mistakes Log

> [!abstract] In one sentence
> Most interview failures are not missing knowledge but a repeated kind of slip — an off-by-one, a wrong data structure, an unread constraint — and the only way to see the repetition is to write every slip down in the same table with the same seven labels.

**Map:** [[Plans MOC]] → [[DSA Dashboard]]

> [!tip] How to use
> Add a row the moment an attempt fails or a review goes wrong — before reading the solution, while the mistake is still yours. Keep rows short. On Sunday, count the categories.

## Categories

| Category | What it means | Typical fix |
|---|---|---|
| `off-by-one` | wrong loop bound, `<` vs `<=`, index vs length, `mid` rounding | write the invariant as a comment before the loop; test with sizes 0, 1, 2 |
| `wrong-structure` | used a list where a heap/map/deque was needed; sorted when a count array would do | say the operation you need ("min in O(log n)", "membership in O(1)") before choosing |
| `missed-edge-case` | empty input, single element, duplicates, negatives, all-equal, overflow-sized values | list five edge cases out loud before coding (see [[Interview Day Protocol]]) |
| `overflow` | `int` sum/product, `mid = (lo + hi) / 2`, `Math.abs(MIN_VALUE)`, subtraction comparators | `long` accumulators, `lo + (hi - lo) / 2`, `Integer.compare` |
| `tle` | right idea, wrong complexity (nested loops where a window/prefix sum would do, recursion without memo) | read the constraints, derive the target complexity from `n` ([[Complexity Analysis]]) |
| `misread` | solved a different problem: wrong direction, inclusive vs exclusive, "subarray" vs "subsequence" | restate the problem and one example in your own words before thinking |
| `api-misuse` | Java API slip: `remove(int)` vs `remove(Object)`, `PriorityQueue` order, `String ==`, `%` on negatives | [[Java Pitfalls in Interviews]], [[Java Collections Cheat Sheet]] |

## The log

| Date | Problem | Category | What happened | The fix I will apply next time |
|---|---|---|---|---|
| 2026-11-04 | [[LC 153 Find Minimum in Rotated Sorted Array]] | off-by-one | compared `nums[mid]` with `nums[lo]`, failed on an unrotated array | compare with `nums[hi]`; test the unrotated case first |
| 2026-11-04 | [[LC 215 Kth Largest Element in an Array]] | wrong-structure | sorted the whole array, O(n log n) when a size-k heap gives O(n log k) | ask "do I need full order?" before sorting |
| 2026-11-04 | [[LC 7 Reverse Integer]] | overflow | checked the bound after `rev = rev * 10 + d` | check `rev > MAX / 10` before multiplying |
| | | | | |

## Sunday questions

1. Which category appears most this week? That is next week's morning reading (e.g. three `overflow` rows → reread [[Java Pitfalls in Interviews]]).
2. Which problems appear twice? Move them to the top of the revision queue by setting `status: needs-revision`.
3. Did any `misread` happen under time pressure? Then the first two minutes of the [[Interview Day Protocol]] are being skipped — rehearse the restatement step.
4. Any `tle` on a Medium? Reread [[Complexity Analysis]]'s constraints table.

## Related

- [[Spaced Repetition Protocol]] — what a failed attempt does to the schedule
- [[8-Week DSA Plan]] — the weekly review slot where this log is read
- [[Java Pitfalls in Interviews]] — the `api-misuse` and `overflow` catalogue
