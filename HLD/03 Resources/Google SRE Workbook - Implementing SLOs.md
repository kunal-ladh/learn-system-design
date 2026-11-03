---
title: Google SRE Workbook - Implementing SLOs
type: resource
resource_type: deep-dive
tags: [resource, deep-dive]
url: https://sre.google/workbook/implementing-slos/
author: "Steven Thurgood and David Ferguson, with Alex Hidalgo and Betsy Beyer (Google)"
year: "2018"
status: unread
priority: high
related_concepts: ["[[SLI SLO and SLA]]", "[[Error Budgets]]", "[[Alerting]]"]
related_problems: ["[[Design a Metrics and Monitoring System]]"]
summary: "The step-by-step recipe for choosing SLIs, setting SLO targets from real data, computing error budgets and writing the policy that decides what happens when the budget runs out."
created: 2026-11-03
updated: 2026-11-03
---

# Google SRE Workbook - Implementing SLOs

> [!abstract] Why this resource
> Every 'non-functional requirements' section in the problem bank wants a number, and this chapter is how you pick one you can defend: from measured data, over a rolling window, with an error budget that drives decisions. Read it before you write your first weekly review of a design.

**Link:** [https://sre.google/workbook/implementing-slos/](https://sre.google/workbook/implementing-slos/)  
**Type:** Architecture deep dive · **Author:** Steven Thurgood and David Ferguson, with Alex Hidalgo and Betsy Beyer (Google) · **Year:** 2018

**Map:** [[Resources MOC]] · [[Engineering Dashboard]]

> [!info] Access and verification
> Free chapter of the SRE Workbook. Link fetched and verified 2026-11-03.

## When to read it

Whole chapter (forty minutes) in week 2 of [[Start Here]] with [[SLI SLO and SLA]]; the decision matrix near the end is the part to copy into your own notes.

## What to look for

*Pre-filled from the resource itself so you know what you are looking for. It is not a substitute for reading: the note stays `unread` until you have.*

- With a 99.9% SLO over 3,000,000 requests in four weeks, the error budget is 3,000 errors; a single 1,500-error outage spends half of it.
- Worked example: 3,663,253 requests at 97.123% success yields proposed SLOs of 97% availability, 90% of requests under 450 ms and 99% under 900 ms.
- A four-week rolling window is the general-purpose interval, with weekly summaries for prioritisation and quarterly reports for planning.
- An SLI is good events divided by total events, so SLO target and error budget (100% minus SLO) share one format.
- 100% is the wrong target: dependencies fail and each extra nine costs more for less user benefit; the error-budget policy needs product, development and SRE sign-off.

## Key takeaways

*Write these in your own words after reading. Then set `status: reading` or `reviewed` and bump `updated`.*

- 

## Personal notes


## Related

### Concepts

- [[Alerting]]
- [[Error Budgets]]
- [[Monitoring]]
- [[Prioritizing Reliability Performance Cost and Time to Market]]
- [[Production Incident Analysis]]
- [[SLI SLO and SLA]]

### Design problems

- [[Design a Metrics and Monitoring System]]
