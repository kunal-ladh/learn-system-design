# LinkedIn Profile Playbook (Refined) — Kunal Ladhani

> **Companion docs:**
> - `03_resume_content.txt` — canonical resume bullets (single source of truth for stats)
> - `advice-refined.md` — job search execution (applications, referrals, HR/HM rounds)
> - `tech-refined.md` — overall SDE2 prep plan
>
> **Goal of this doc**: pick headline + About + experience bullets you can paste into LinkedIn in under 30 minutes, with tactical guidance LinkedIn-specific (pinned skills, post cadence, recommendation requests).

---

## Table of Contents

1. [Quick-Apply Checklist](#1-quick-apply-checklist)
2. [Headlines](#2-headlines-4-options)
3. [About / Summary](#3-about--summary-3-versions)
4. [Experience Section](#4-experience-section)
5. [Skills Section](#5-skills-section)
6. [Featured / Posts](#6-featured--posts)
7. [Recommendations](#7-recommendations)
8. [LinkedIn Hygiene](#8-linkedin-hygiene)

---

## 1. Quick-Apply Checklist

### 5-Minute Version (do today)
1. Update **Headline** — pick one from Section 2
2. Pin **Top 3 Skills** — Java, Spring Boot, Apache Kafka (these are the SDE2-backend search keywords)
3. Replace **About** section — pick one version from Section 3

### 30-Minute Version (do this weekend)
- Everything above
- Rewrite **Experience** section using Section 4 blocks
- Add all skills from Section 5 (LinkedIn allows 50)
- Send 3 recommendation requests using the DM template in Section 7
- Turn on "Open to Work" (visible to recruiters only, not your network)

---

## 2. Headlines (4 Options)

LinkedIn's headline is the #1 ranking factor for recruiter search. Pack it with keywords + impact.

### Option A — Impact-First (recommended for SDE2 search)
```
Backend Engineer | Built 7 microservices for 3 banking partners | Java · Spring Boot · Kafka · AWS
```
Pick this if you want recruiters to see scope immediately. Highest signal.

### Option B — Recruiter-Keyword-Optimized
```
SDE2 Backend Engineer | Java · Spring Boot · Kafka · AWS · Microservices · Event-Driven · Fintech
```
Pick this if you want maximum search-hit volume. Less personality, more keywords.

### Option C — Company-First (current default)
```
Software Engineer at Niyo Solutions | Building Neobanking Platforms | Java · TypeScript · Spring Boot · Kafka · AWS
```
Pick this if you want to lead with brand. Safe but unremarkable.

### Option D — Short
```
Backend Engineer | Fintech | Java · TypeScript · Microservices
```
Pick this for a clean look. Loses recruiter-search advantage.

**Recommendation**: **Option A**. It signals SDE2-level scope without saying "looking for a job", which preserves dignity while signaling intent.

---

## 3. About / Summary (3 Versions)

### Version A — Long (Recruiter-Friendly, ~200 words)

Use this if you want recruiters to fully understand scope without clicking into Experience.

```
Backend engineer with 3+ years building production banking systems at Niyo Solutions.
I've designed, built, and shipped microservices that power neobanking products across
3 partner banks (Utkarsh SFB, Yes Bank, NSDL Payments Bank) — handling customer
onboarding, agent onboarding, KYC/BGV, fund transfers, card management, lending
offers, and notifications.

My work spans the full backend lifecycle — designing APIs and data models, building
Kafka event pipelines, setting up CI/CD with security scanning (SonarQube, Trivy,
Snyk), and deploying to Kubernetes/Facets Cloud. I work across Java (Spring Boot)
and TypeScript (NestJS/Express) stacks and switch between them based on product needs.

What I've shipped:
→ Built 7+ microservices from scratch across 14 repos (4,100+ commits)
→ Owned an entire agent onboarding service as the sole developer (544 commits) —
  login, BGV with face match, risk assessment, business approvals
→ Architected 5 NestJS microservices for the Utkarsh neobanking platform
→ Led Spring Boot 2→3 migrations and Facets Cloud migrations across 5+ services
→ Built event-driven flows with Kafka (MSK), SQS, EventBridge, and SSE
→ Created AWS Lambda functions with SAM, EventBridge scheduling, and SnapStart
→ Set up CI/CD with SonarQube, Snyk, Trivy, OSV, and ECR scanning

Currently looking for SDE-2 backend roles where I can own system design decisions
and continue scaling reliability-critical fintech systems.
```

### Version B — Medium (Peer-Friendly, ~100 words)

Use this for a balanced, scannable read.

```
Backend engineer building neobanking platforms at Niyo Solutions. I ship microservices
in Java (Spring Boot) and TypeScript (NestJS) that handle customer onboarding, agent
onboarding, fund transfers, card management, lending, and real-time notifications for
banking products.

3+ years, 4,100+ commits across 14 repos. Built 7+ services from scratch, led Spring
Boot 2→3 and Facets Cloud migrations, and own multiple production banking backends.
Stack: Kafka (MSK), AWS (Lambda, S3, SQS, EventBridge, ECR, K8s), MongoDB, MySQL,
CI/CD with security scanning.

Looking for SDE-2 roles with system-design ownership.
```

### Version C — Short (Punchy, ~50 words)

Use this if you want recruiters to click into Experience to learn more.

```
Backend engineer · 3+ yrs · Java/Spring Boot + TypeScript/NestJS · Built 7+ banking
microservices for 3 partner banks at Niyo. 4,100+ commits across 14 repos. Strong in
Kafka, AWS, microservices, event-driven systems. Looking for SDE-2.
```

**Recommendation**: **Version A** while actively job-searching. The longer About reduces friction for recruiters and converts more InMails into conversations.

---

## 4. Experience Section

LinkedIn lets you add **one role with multiple project blocks**. Use this structure: each project = 1-line context + 3-5 impact bullets.

### Header

```
Title:    Software Development Engineer
Company:  Niyo Solutions Inc.
Duration: December 2022 – Present
Location: Bengaluru, India
```

### Intro paragraph (under the header)

```
Building backend systems for Niyo's neobanking products — Bharat (Yes Bank partnership)
and Utkarsh (Utkarsh Small Finance Bank). Working across Java/Spring Boot and
TypeScript/NestJS microservices with MongoDB, Kafka, and AWS.
```

### Project Blocks (ordered most-impressive first)

---

**Bharat Agent Onboarding (Java/Spring Boot) — Oct 2024 to Present**

Solo-built greenfield microservice for agent onboarding across the Bharat product line.

- Built the entire agent onboarding microservice from scratch as the sole developer (544 commits) — login/OTP, agent invite, BGV with Digio face match, risk assessment, business approvals, Kafka activity tracking
- Designed multi-channel API architecture (`/ag`, `/app`, `/wb`) enabling independent evolution for agent-facing, mobile app, and web backoffice clients
- Built end-to-end Background Verification (BGV) system — Digio face match API, dynamic HTML-to-PDF report generation, PAN masking for PII compliance, RBAC-based access
- Led Facets Cloud migration (60+ commits) — Datadog APM, AWS Secrets Manager, custom Spring Security auth

---

**Utkarsh Banking Platform (TypeScript/NestJS) — Jun 2023 to Jul 2024**

Architected 5 microservices for a complete neobanking product launch with Utkarsh SFB.

- Architected and built 5 microservices from scratch — Customer Service, BFF, Transaction Service, Card Service, Bank Connector
- Designed full customer onboarding flow — registration, eKYC OTP, PAN validation, Video KYC, bank registration, account opening
- Built fund transfer system supporting NEFT/IMPS/IFT with daily limits (₹5L cap), OTP authorization, duplicate prevention, beneficiary cooldown
- Implemented AOF PDF generation pipeline — HTML templating, PDF converter, SFTP upload to bank, Kafka async processing, S3 backup
- Built real-time event streaming using Kafka MSK + Server-Sent Events (SSE) for fund transfer and beneficiary status updates

---

**Bharat Customer Service & VAS (Java + TypeScript) — Oct 2023 to Present**

Own two production banking backends with 443+ commits on VAS alone over 2.5 years.

- Own customer service backend for Bharat — onboarding, KYC, notifications, levy management, cross-sell APIs
- Led Spring Boot 3 migration — upgraded 20+ dependencies (Kafka, MongoDB, JPA/Hibernate, Datadog), zero downtime
- Migrated Kafka infrastructure from Confluent to AWS MSK with ZSTD compression across API and worker modules
- Built template-based SMS notification system with 15+ templates covering account lifecycle events
- Led AWS SDK v2→v3 migration on VAS and fixed critical Kafka consumer rebalancing in production

---

**NSDL Payments Bank Services (Java + AWS Lambda) — Jun 2024 to Present**

Built 4 microservices and led Lambda-to-K8s migration for NSDL PB integration.

- Built 4 microservices — Customer Service (K8s), Customer Service (Lambda), NSDL Lambda, Cron Lambda — from scratch with SAM templates and EventBridge scheduling
- Created S2S callback API with IP-whitelisted gateway, SQS integration, and Lambda SnapStart for cold-start optimization
- Led Lambda-to-K8s migration via Facets Cloud — consolidated runtime, reduced cold-start latency

---

**Ledger Service (Java/Spring Boot) — Sep 2024 to Present**

Led Spring Boot 2→3 + Java 8→17 migration with zero downtime.

- Led Spring Boot 2→3 migration (60+ commits, Java 8→17, javax→jakarta) with zero downtime in production
- Built bulk transaction sync via Kafka with rate limiting and ZSTD compression

---

**Inventory Manager (Java/Spring Boot) — Jul 2024 to Present**

Built Partner Agent Kit module and custom dual-auth security layer.

- Built Partner Agent Kit Request module from scratch
- Designed custom dual-auth system (19 files, 1,400+ lines) supporting JWT-Visionary tokens and S2S API calls
- Built Asset Transfer V2 APIs with S3 presigned URLs and Excel report generation

---

**OA Lending Offers Service (Java/Spring Boot) — Nov 2024 to Jun 2025**

Built greenfield personal loan offers service end-to-end.

- Built greenfield personal loan offer service — CSV ingestion from ABFL, EMI calculation engine, consent capture with SSE, Kafka integration

---

**Risk Engine (Java/Spring Boot) — Jan 2023 to Apr 2023**

First project at Niyo — built configurable risk assessment engine from scratch.

- Built configurable risk assessment engine — multi-module Maven project with API, Worker, Commons modules
- Implemented 10+ risk validators (Address, CKYC, CRIF, DeviceId, Email, Income, IP, Occupation, PIN) with parameterized unit tests
- Integrated Sign3 third-party API for social/digital footprint verification
- Built maker-checker approval workflow for risk rule changes — preferences require approval before activation

---

## 5. Skills Section

LinkedIn allows up to 50 skills. Add them grouped — recruiters scan by category.

### Top 3 to Pin (recruiter-search optimized)

LinkedIn lets you pin 3 skills at the top. For SDE2 backend search visibility:

| # | Skill | Why |
|---|---|---|
| 1 | **Java** | The #1 keyword for SDE2 backend searches in India |
| 2 | **Spring Boot** | Filter recruiters use after "Java" |
| 3 | **Apache Kafka** | Differentiator — far fewer candidates have it |

### Full Skills List (by category)

| Category | Skills |
|---|---|
| **Languages** | Java, TypeScript, JavaScript, SQL |
| **Backend Frameworks** | Spring Boot, Spring Security, NestJS, Express.js, Hibernate, JPA |
| **Data** | MongoDB, MySQL, Liquibase, Redis (basic) |
| **Messaging / Streaming** | Apache Kafka, AWS MSK, AWS SQS, AWS EventBridge, RabbitMQ |
| **Cloud & Infra** | AWS (S3, KMS, ECR, EKS, ASM, SQS, Lambda, SAM, API Gateway), Docker, Kubernetes, Facets Cloud |
| **DevOps & CI/CD** | Git, Bitbucket Pipelines, SonarQube, Snyk, Trivy, OSV Scanner |
| **Monitoring** | Datadog, OpenTelemetry, Graylog |
| **Testing** | JUnit 5, Jest, Mockito, HyperTest |
| **APIs & Protocols** | REST, OpenAPI/Swagger, gRPC (basic), SSE, SFTP, Retrofit |
| **Architecture Patterns** | Microservices, Event-Driven Architecture, BFF Pattern, Worker Pattern, Serverless |

### Domain Skills (add separately — recruiters in fintech search by these)

Fintech, Neobanking, Payment Systems, KYC / eKYC / BGV, Risk Management, Agent Onboarding, Lending, Card Management

---

## 6. Featured / Posts

LinkedIn rewards posts. While job-searching, **1 high-quality post per week + 5 thoughtful comments/week** beats daily noise.

### 4 Post Hooks Tied to Your Real Work

#### Post 1: Kafka Consumer Rebalancing War Story
**Hook**: *"Our Kafka consumers rebalanced on every deployment. Here's what we found and how we fixed it."*
**Why this works**: Real production debugging story. Specific. Senior engineers will engage.
**Anchor it in**: Bharat VAS consumer fix.

#### Post 2: Spring Boot 2→3 Migration — What Nobody Talks About
**Hook**: *"Migrating Spring Boot 2 to 3 isn't just `javax → jakarta`. Here are the 5 unexpected issues we hit."*
**Why this works**: Migrations are universal pain. Practical content travels.
**Anchor it in**: Ledger Service + Bharat Customer Service migrations.

#### Post 3: Building a Banking Microservice Solo — 544 Commits Later
**Hook**: *"I built our entire agent onboarding service from scratch as the only engineer. Here's the architecture and what I'd do differently."*
**Why this works**: Ownership signal. Architecture content. Numbers in the headline.
**Anchor it in**: Bharat Agent Onboarding.

#### Post 4: From Confluent to MSK — A ZSTD-Compressed Story
**Hook**: *"We moved 10+ services from Confluent Cloud to AWS MSK. The cost savings were obvious. The gotchas weren't."*
**Why this works**: Specific migration. Practical lessons. Cost angle = hiring manager catnip.
**Anchor it in**: Bharat Customer Service Kafka migration.

### Cadence Guide

| Activity | Frequency |
|---|---|
| Original posts | 1/week (Mon or Tue, 9 AM IST hits the algo best) |
| Comments on others' posts | 5/week (thoughtful, not "great post!") |
| Article shares with takeaway | 1-2/week |
| DMs to network | 3-5/week (not spam — real conversations) |

---

## 7. Recommendations

LinkedIn recommendations on your profile = social proof. Aim for **3-5 strong ones** during your job search.

### Who to Ask + What to Highlight

| Who | What to ask them to highlight |
|---|---|
| Your tech lead / EM | Architecture decisions, ownership, end-to-end delivery |
| Senior engineer on Utkarsh | Building 5 microservices from scratch, NestJS work |
| Senior engineer on Bharat | Migration leadership, Kafka work, Spring Boot 3 |
| Product manager you've worked with | Delivery reliability, cross-functional collaboration |
| Peer engineer | Code quality, debugging, helping unblock teammates |

### DM Template (paste-ready)

```
Hi [name],

I'm in the middle of a job search for SDE2 roles and a few interviewers have asked
about recommendations. Would you be open to writing a short one for my LinkedIn?

If it helps, here are 2-3 things you could mention (feel free to use your own
framing):
- [specific thing 1 — e.g., "owning the agent onboarding service end-to-end"]
- [specific thing 2 — e.g., "the Spring Boot 3 migration we did together"]
- [specific thing 3 — e.g., "how I unblocked X for your team"]

Happy to return the favor — I'd be glad to write one for you too.

Thanks!
Kunal
```

**Tip**: Customize the 2-3 bullets per recipient. Generic asks get generic recommendations.

---

## 8. LinkedIn Hygiene

Small things that compound.

### Profile Polish
- **Profile photo**: clean headshot, neutral background, smiling. Phone camera is fine if lighting is good.
- **Banner image**: a clean tech-themed banner (city skyline, code abstract, or your company's product). Avoid stock templates.
- **Contact info**: add personal email + GitHub profile in the contact section
- **Custom URL**: claim `linkedin.com/in/kunalladhani` (or closest available) — looks professional in resumes

### Settings
- Turn ON: "Open to Work" (visible to **recruiters only**, not your network — choose this option)
- Turn ON: "Career interests" — select India, Remote, plus any specific countries
- Add: minimum compensation expectation (recruiters filter on this)

### What NOT to Do
- Don't add "looking for a job" to your headline — looks desperate, hurts negotiating leverage
- Don't change your profile picture every week — disrupts recognition
- Don't post motivational quotes / generic career posts — adds noise, dilutes signal
- Don't endorse skills you don't actually have — interviewers check

---

## Quick Reference Cheat Sheet

```
HEADLINE (pick Option A):
  Backend Engineer | Built 7 microservices for 3 banking partners |
  Java · Spring Boot · Kafka · AWS

TOP 3 PINNED SKILLS:
  Java · Spring Boot · Apache Kafka

ABOUT (pick Version A for job search)

EXPERIENCE BLOCKS (in this order):
  1. Bharat Agent Onboarding   (sole dev, 544 commits)
  2. Utkarsh Banking Platform  (5 microservices from scratch)
  3. Bharat Customer + VAS     (Kafka, SB3 migration)
  4. NSDL Payments Bank        (Lambda + K8s)
  5. Ledger Service            (SB2→3 migration)
  6. Inventory Manager         (dual-auth)
  7. OA Lending Offers         (greenfield)
  8. Risk Engine               (first project)

POSTING CADENCE:
  1 original post/week
  5 thoughtful comments/week
  3-5 network DMs/week

RECOMMENDATIONS TARGET:
  3-5 strong ones before applying heavily

GOLDEN RULE:
  Optimize for recruiter search, not vanity.
  Keywords + numbers + verbs win.
```

---

*Source archive: `02_linkedin_content.txt` (original draft, untouched).*
