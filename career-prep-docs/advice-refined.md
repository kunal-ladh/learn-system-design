# SDE2 Job Search Playbook (Refined)

A 5-step action plan to convert your work history into SDE2 offers.

---

## Step 1: LinkedIn + Resume First (This Weekend)

Update both in parallel — applications start before you "feel ready".

### LinkedIn

- Use `02_linkedin_content.txt` as source material
- Pick the **headline**, **about section**, and **experience bullets** that feel natural — don't copy-paste everything
- Lead with what you actually built, not what you helped with

### Resume

- Use `03_resume_content.txt` as source material
- Keep it to **1-2 pages max**
- Lead with your strongest stories:
  - Agent Onboarding (sole developer, 544 commits)
  - Utkarsh Platform (5 microservices from scratch)
- Tailor bullets to each role (Kafka role → lead with Kafka work; AWS role → lead with AWS work)

### Rule

Start applying the moment these are live. You will prep in parallel.

---

## Step 2: Pick Your 5 Core Stories

From the 10 STAR stories in `01_complete_contributions_detail.txt`, pick 5 you can deliver in **3-5 minutes each without notes**.

### Recommended 5

| Story | What it signals |
|---|---|
| Agent Onboarding (544 commits, sole dev) | Ownership end-to-end |
| Utkarsh Platform (5 microservices from scratch) | System architecture |
| Spring Boot 2→3 Migration | Handling complexity + risk |
| Kafka Consumer Rebalancing Fix | Production debugging |
| BGV System / AOF Pipeline | Feature delivery end-to-end |

### Practice rule

Speak each story **out loud** — to yourself or a friend. The first 2 times will feel clunky. By the 3rd it flows.

---

## Step 3: HR / HM Rounds

### HR opener

"Tell me about your current role" — do NOT list 14 repos.

> "I've been at Niyo for 3+ years building banking backends. I started with a risk engine, then built an entire neobanking platform from scratch across 5 microservices, and now I own multiple production services — including one I built solo with 544 commits."

Then let them ask follow-ups.

### HM deep dive

Expect: *"What was the hardest technical problem you solved?"*

Pull from:
- Kafka rebalancing fix
- Spring Boot 2→3 migration
- Facets Cloud migration

Refresh specifics from the analysis files (`agent_onboarding_analysis.txt`, `bharat_vas_analysis.txt`, etc.) the night before each interview.

---

## Step 4: Technical / System Design Rounds

You have **real production system design experience** — use it.

| If they ask | Pull from |
|---|---|
| Design a payment / banking system | Your Bharat + Utkarsh work |
| Design an event-driven system | Kafka across 10+ services |
| Design a notification system | Template-based SMS, 15+ event types |
| Design an onboarding flow | Customer onboarding + BGV |

Real experience beats textbook answers. Interviewers can tell the difference.

### SDE-2 signals to emphasize

- You made **architecture decisions** (multi-channel controllers, dual-auth, worker patterns) — not just wrote code
- You **owned services end-to-end** — not features within someone else's service
- You handled **production incidents** (Kafka rebalancing, NPEs, migration rollbacks)
- You improved things **proactively** (security hardening, Sonar fixes, Facets migration)

---

## Step 5: DSA Prep (Parallel Track)

The one thing these docs won't help with.

- Stick to one list (NeetCode 150 recommended) for 8-10 weeks
- 1-2 mediums per day, timed
- Most SDE-2 loops still gate on DSA, no shortcuts

See `tech-refined.md` for the full DSA + LLD + HLD weekly schedule.

---

## Confidence Calibration

Don't undersell yourself. Walk into interviews knowing this:

- **4,100+** commits across **14** repos
- **7+** services built from scratch
- **Sole developer** on a 544-commit service
- Multi-stack: Java + TypeScript
- Production migrations, debugging, ownership

That is **not SDE-1 output**.

---

## Quick Reference: Files in This Folder

| File | Use it for |
|---|---|
| `tech-refined.md` | Study schedule, book list, LLD patterns, DSA plan |
| `advice-refined.md` | This file — job search action plan |
| `01_complete_contributions_detail.txt` | STAR stories reference bible |
| `02_linkedin_content.txt` | LinkedIn headline/about/experience drafts |
| `03_resume_content.txt` | Resume bullets and summaries |
| `agent_onboarding_analysis.txt` | Deep dive: agent onboarding work |
| `bharat_vas_analysis.txt` | Deep dive: Bharat VAS work |
| `inventory_manager_analysis.txt` | Deep dive: Inventory Manager |
| `ledger_and_lending_analysis.txt` | Deep dive: Ledger + Lending |
| `nsdl_services_analysis.txt` | Deep dive: NSDL services |
| `Grokking-System-Design/` | HLD reference PDFs + canonical problems |
