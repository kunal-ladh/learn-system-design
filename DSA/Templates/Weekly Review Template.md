---
title: {{date:GGGG-[W]WW}} Review
type: review
tags: [review]
week: {{date:GGGG-[W]WW}}
period_start:
period_end: {{date:YYYY-MM-DD}}
created: {{date:YYYY-MM-DD}}
---

# Week {{date:GGGG-[W]WW}} review

> [!tip] Fill `period_start` (the Monday) and `period_end` (today) in the properties above — the automatic sections below use them.

## What I learned

-

## What I can now do without the note

-

## What surprised me

-

## What remains unclear

-

## Mock this week

| Date | Problem | Understanding | Approach | Code | Testing | Communication |
|---|---|---|---|---|---|---|
| | | /5 | /5 | /5 | /5 | /5 |

## Three priorities for the coming week

1.
2.
3.

---

## Automatic progress (Dataview)

> [!info]- Problems solved this period
> ```dataview
> TABLE category, difficulty, confidence, time_minutes, last_solved
> FROM "02 NeetCode 150"
> WHERE type = "problem" AND last_solved >= this.period_start AND last_solved <= this.period_end
> SORT last_solved DESC
> ```

> [!info]- Topics reviewed this period
> ```dataview
> TABLE category, status, confidence, last_reviewed
> FROM "01 Topics"
> WHERE type = "topic" AND last_reviewed >= this.period_start AND last_reviewed <= this.period_end
> SORT last_reviewed DESC
> ```

> [!info]- Still due
> ```dataview
> TABLE status, confidence, next_review
> FROM "01 Topics" OR "02 NeetCode 150"
> WHERE next_review <= this.period_end AND status != "mastered"
> SORT next_review ASC
> ```
