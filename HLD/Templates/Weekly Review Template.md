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

## What I can now explain independently

- 

## What I designed or implemented

- 

## What surprised me

- 

## What remains unclear

- 

## Which assumptions I need to validate

- [ ] 

## What I want to build next

- 

## Three priorities for the coming week

1. 
2. 
3. 

---

## Automatic progress (Dataview)

> [!info]- Concepts reviewed this period
> ```dataview
> TABLE category, status, confidence, last_reviewed
> FROM "01 HLD & System Design"
> WHERE type = "concept" AND last_reviewed >= this.period_start AND last_reviewed <= this.period_end
> SORT last_reviewed DESC
> ```

> [!info]- Design problems worked this period
> ```dataview
> TABLE status, confidence, last_worked
> FROM "02 System Design Problems"
> WHERE type = "problem" AND last_worked >= this.period_start AND last_worked <= this.period_end
> SORT last_worked DESC
> ```

> [!info]- Projects updated this period
> ```dataview
> TABLE status, current_milestone, next_task
> FROM "04 Projects"
> WHERE type = "project" AND updated >= this.period_start AND updated <= this.period_end
> SORT updated DESC
> ```

> [!info]- Decisions recorded this period
> ```dataview
> TABLE project, status
> FROM "04 Projects/Decisions"
> WHERE type = "decision" AND created >= this.period_start AND created <= this.period_end
> ```

> [!info]- Mock interviews this period
> ```dataview
> TABLE problem, format, overall, score_tradeoffs, score_communication
> FROM "06 Interview Prep/Mocks"
> WHERE type = "mock" AND date >= this.period_start AND date <= this.period_end
> SORT date DESC
> ```
