# Resume Playbook (Refined) — Kunal Ladhani

> **Companion docs:**
> - `02_linkedin_content-refined.md` — LinkedIn playbook (parallel content)
> - `advice-refined.md` — job search execution (HR/HM rounds, applications)
> - `tech-refined.md` — overall SDE2 prep plan
>
> **Goal**: produce a 1-page (max 2) ATS-friendly SDE2 resume that gets past keyword filters AND tells your story to a human reader in under 30 seconds.

---

## Table of Contents

1. [Quick-Apply Checklist](#1-quick-apply-checklist)
2. [Resume Strategy](#2-resume-strategy-1-page-vs-2-page)
3. [Header / Contact](#3-header--contact)
4. [Professional Summary](#4-professional-summary-3-versions)
5. [Technical Skills](#5-technical-skills)
6. [Work Experience](#6-work-experience)
7. [Top 15 Bullets — Ranked](#7-top-15-bullets--ranked-by-impact)
8. [Projects Section](#8-projects-section-optional)
9. [Education & Certifications](#9-education--certifications)
10. [ATS Optimization](#10-ats-optimization)
11. [Common Mistakes to Avoid](#11-common-mistakes-to-avoid)
12. [Tailoring Per Role](#12-tailoring-per-role)

---

## 1. Quick-Apply Checklist

### 15-Minute Version (do today)
1. Use **Summary Version A** (Section 4) as your professional summary
2. Add **Technical Skills table** from Section 5 — exactly as-is
3. Lead Experience with the **Agent Onboarding** block (Section 6) — your strongest story

### 60-Minute Version (do this weekend)
- Everything above
- Add 6 more project blocks from Section 6
- Run through Section 10 (ATS Optimization)
- Run through Section 11 (Common Mistakes) and audit your draft
- Save as PDF, name it `Kunal_Ladhani_SDE2_Resume.pdf`
- Test ATS readability: paste into [resumeworded.com](https://resumeworded.com) — should score 80+

---

## 2. Resume Strategy (1-page vs 2-page)

### Default: 1 page
3+ YOE in India = 1 page is the norm for SDE2. Recruiters scan in 6-10 seconds.

### Switch to 2 pages only if:
- Applying abroad (US/EU often expects 2 pages by mid-level)
- You're applying to research-heavy roles
- You have 5+ years and have published / patented / spoken publicly

**For you specifically**: stay 1-page. Your strongest signal is *depth on a few projects*, not breadth of 14 repos. Pick 3-4 projects, go deep, leave the rest off.

### Layout principles
- 10-11pt font (Helvetica, Inter, or Calibri)
- 0.5-0.75 inch margins
- Single column (ATS-friendly — two-column resumes break parsers)
- No graphics, no photo (in India — graphics are noise; in some countries photos are expected, but never for tech roles)
- Save and submit as **PDF**, not Word

---

## 3. Header / Contact

```
KUNAL LADHANI
Backend Engineer · 3+ YOE · Fintech / Banking Microservices
Bengaluru, India · kunal.ladhani@goniyo.com · +91-XXXXXXXXXX
LinkedIn: linkedin.com/in/<your-handle> · GitHub: github.com/<your-handle>
```

Tip: replace email with a **personal Gmail** (not work email) on the resume you upload to job boards. Work email = signal you might leave.

---

## 4. Professional Summary (3 Versions)

### Version A — Recommended (Impact-First, ~50 words)

```
Backend engineer with 3+ years building production banking systems at Niyo Solutions.
Shipped 7+ microservices from scratch (4,100+ commits across 14 repos) for 3 partner
banks. Led Spring Boot 2→3 and Facets Cloud migrations. Strong in Java, Spring Boot,
Kafka, AWS, and microservices architecture. Seeking SDE-2 roles with system-design
ownership.
```

### Version B — Stack-Heavy (for keyword-filtered ATS)

```
Backend engineer specializing in fintech microservices. 3+ years building customer
onboarding, agent onboarding, fund transfers, card management, lending, and
notifications for neobanking products. Experienced across Java/Spring Boot 3 and
TypeScript/NestJS with event-driven architectures (Kafka MSK, SQS, EventBridge),
AWS, Kubernetes, and CI/CD with security scanning.
```

### Version C — Terse (for design-heavy resumes that prioritize white space)

```
Software Engineer | 3+ YOE | Java · TypeScript · Spring Boot · NestJS · Kafka · AWS
Built and own banking platform backends across 3 partner banks at Niyo Solutions.
```

**Pick**: Version A for most applications. Version B if the JD is keyword-heavy and ATS-driven.

---

## 5. Technical Skills

ATS-friendly format. Single column, no fancy bars or rating dots.

| Category | Skills |
|---|---|
| **Languages** | Java 17, TypeScript, JavaScript, SQL |
| **Frameworks** | Spring Boot 3.x, Spring Security, NestJS, Express.js, Hibernate, JPA |
| **Databases** | MongoDB, MySQL, Liquibase |
| **Messaging** | Apache Kafka (Confluent, AWS MSK), AWS SQS, AWS EventBridge, RabbitMQ |
| **Cloud & Infra** | AWS (S3, KMS, ECR, EKS, ASM, SQS, Lambda, SAM, API Gateway), Docker, Kubernetes, Facets Cloud |
| **CI/CD** | Bitbucket Pipelines, SonarQube, Snyk, Trivy, OSV Scanner |
| **Monitoring** | Datadog, OpenTelemetry, Graylog |
| **Testing** | JUnit 5, Mockito, Jest, HyperTest |
| **APIs & Protocols** | REST, OpenAPI/Swagger, gRPC, SSE, SFTP, Retrofit |
| **Architecture** | Microservices, BFF Pattern, Event-Driven Architecture, Serverless, Worker Pattern |

> ATS tip: keep the keyword separators consistent (commas, no bullets, no fancy dividers). Some parsers choke on em-dashes.

---

## 6. Work Experience

```
NIYO SOLUTIONS INC.                                          Bengaluru, India
Software Development Engineer                              Dec 2022 – Present
```

### Project Block: Bharat Agent Onboarding (Java/Spring Boot) | Oct 2024 – Present

- Built the entire agent onboarding microservice from scratch as **sole developer (544 commits)** — login/OTP, agent invite, BGV with face match, risk assessment, business approvals, Kafka-based activity tracking
- Designed multi-channel API architecture (`/ag`, `/app`, `/wb`) enabling independent API evolution for agent, mobile, and web backoffice clients
- Built end-to-end Background Verification (BGV) system — Digio face match API integration, dynamic HTML-to-PDF report generation with PDFBox, PAN masking for PII compliance, RBAC-based access control
- Led Facets Cloud migration (60+ commits) — Datadog APM, AWS Secrets Manager, custom Spring Security auth system

### Project Block: Utkarsh Banking Platform (TypeScript/NestJS) | Jun 2023 – Jul 2024

- Architected and built **5 microservices from scratch** for neobanking platform with Utkarsh SFB — Customer Service, BFF, Transaction Service, Card Service, Bank Connector
- Designed complete customer onboarding flow — registration, eKYC OTP, PAN validation, Video KYC, bank registration, and account opening
- Built fund transfer system (NEFT/IMPS/IFT) with ₹5L daily limits, OTP authorization, duplicate prevention, and beneficiary cooldown periods
- Implemented AOF PDF generation pipeline — HTML templating, PDF converter, SFTP upload with SSH key authentication, Kafka async processing, S3 backup
- Built real-time event streaming using Kafka MSK + Server-Sent Events (SSE) for fund transfer and beneficiary status updates

### Project Block: Bharat Customer Service & VAS (Java + TypeScript) | Oct 2023 – Present

- Own production banking backends — customer service (Java/Spring Boot 3) and VAS (TypeScript, **443 commits over 2.5 years**)
- Led **Spring Boot 3 migration** — upgraded 20+ dependencies (Kafka, MongoDB, JPA/Hibernate, Swagger, Datadog), resolved circular dependency and bean configuration issues, zero downtime
- Migrated Kafka infrastructure from Confluent to AWS MSK with ZSTD compression across API and worker modules
- Built template-based SMS notification system with 15+ templates covering full account lifecycle (activation, closure, card operations, fund transfers, phone updates)
- Led AWS SDK v2→v3 migration on VAS and fixed critical Kafka consumer rebalancing issue in production

### Project Block: NSDL Payments Bank Services (Java + AWS Lambda) | Jun 2024 – Present

- Built **4 microservices** for NSDL PB — customer service (K8s), customer service (Lambda), NSDL Lambda, and cron Lambda (built from scratch with SAM/EventBridge)
- Created S2S callback API with IP-whitelisted gateway, SQS integration, Lambda SnapStart for cold-start optimization
- Led Lambda-to-K8s consolidation migration via Facets Cloud

### Project Block: Ledger Service (Java/Spring Boot) | Sep 2024 – Present

- Led **Spring Boot 2→3 migration** (60+ commits, Java 8→17, javax→jakarta) with zero downtime in production
- Built bulk transaction sync via Kafka with rate limiting and ZSTD compression

### Project Block: Inventory Manager (Java/Spring Boot) | Jul 2024 – Present

- Built Partner Agent Kit Request module from scratch
- Designed custom dual-auth system (19 files, 1,400+ lines) supporting JWT-Visionary tokens and S2S API calls
- Built Asset Transfer V2 APIs with S3 presigned URLs and Excel report generation

### Project Block: OA Lending Offers Service (Java/Spring Boot) | Nov 2024 – Jun 2025

- Built greenfield personal loan offer service — CSV ingestion from ABFL, EMI calculation engine, consent capture with SSE, Kafka integration

### Project Block: Risk Engine (Java/Spring Boot) | Jan 2023 – Apr 2023

- Built configurable risk assessment engine from scratch — multi-module Maven project with API, Worker, Commons modules
- Implemented 10+ risk validators (Address, CKYC, CRIF, DeviceId, Email, Income, IP, Occupation, PIN) with parameterized unit tests
- Integrated Sign3 API for social/digital footprint verification
- Built maker-checker approval workflow for risk rule changes

> **For your 1-page resume**: Pick the **top 4 blocks** (Agent Onboarding, Utkarsh, Bharat Customer/VAS, NSDL or Ledger). Leave the rest off — they're in the "Top 15 Bullets" section in compressed form if you need to reference them.

---

## 7. Top 15 Bullets — Ranked by Impact

Use these if you prefer a flat bullet-list experience section instead of project blocks.

1. Built 7+ microservices from scratch across 14 repos (4,100+ commits) for neobanking products with 3 partner banks (Utkarsh SFB, Yes Bank, NSDL PB)
2. Built entire agent onboarding service (544 commits, **sole developer**) — login/OTP, BGV with face match, risk assessment, business approvals, multi-channel API architecture, Kafka event-driven activity tracking
3. Architected 5 NestJS microservices from scratch for Utkarsh neobanking platform — customer onboarding, fund transfers, card management, account statements
4. Led Spring Boot 2→3 migrations across 3+ services — Java 8→17, Spring Security 5→6, javax→jakarta, zero-downtime deployments
5. Led Facets Cloud migration across 5+ services — deleted legacy K8s configs, integrated Datadog APM, migrated secrets to AWS Secrets Manager
6. Built BGV system with Digio face match API, dynamic PDF report generation, PAN masking for PII compliance, and RBAC access control
7. Designed fund transfer system with NEFT/IMPS/IFT support, ₹5L daily limits, OTP authorization, and duplicate transaction prevention
8. Built AOF PDF generation pipeline with SFTP upload, Kafka async processing, and S3 backup — automating bank document delivery
9. Built greenfield lending offers service — CSV ingestion from ABFL, EMI calculation engine, consent capture with SSE, Kafka integration
10. Implemented event-driven architectures with Kafka (MSK + KafkaJS), SQS, EventBridge, and SSE across 10+ services
11. Built custom dual-auth system (19 files, 1,400+ lines) supporting JWT-Visionary tokens and S2S API calls
12. Created AWS Lambda functions with SAM templates, EventBridge scheduling, Lambda SnapStart, and IP-whitelisted API Gateways
13. Migrated AWS SDK v2→v3, fixed critical Kafka consumer rebalancing in production, implemented ZSTD compression across multiple services
14. Set up CI/CD pipelines with SonarQube, Snyk, Trivy, OSV, and ECR scanning across all services — enforcing SDLC compliance
15. Built configurable risk engine with 10+ validators, Sign3 third-party API integration, and maker-checker approval workflow

---

## 8. Projects Section (Optional)

If your resume has space (rare on 1-page) and your role count is short, a Projects section helps. Use this if you have personal/side projects too.

```
NIYO PRODUCTION PROJECTS

Utkarsh Neobanking Platform | NestJS, MongoDB, Kafka MSK, AWS, K8s
   - Architected 5 microservices from scratch for a full neobanking product
   - Customer onboarding, fund transfers, card management, AOF PDF pipeline
   - Real-time events via Kafka + SSE; secure file delivery via SFTP

Bharat Agent Onboarding Service | Java 17, Spring Boot 3, MongoDB, Kafka
   - Sole developer (544 commits) — login, BGV with face match, risk, approvals
   - Multi-channel API architecture (/ag, /app, /wb)
   - Led Facets Cloud + Datadog + AWS Secrets Manager integration

Risk Assessment Engine | Java, Spring Boot, MongoDB, Kafka, Excel processing
   - Configurable rule-based risk scoring engine
   - 10+ validators, Sign3 API integration, maker-checker workflow
```

> Most SDE2 resumes in India don't need a separate Projects section — experience blocks cover it. Use Projects only if you have meaningful personal projects to add.

---

## 9. Education & Certifications

```
EDUCATION

Birla Institute of Technology and Science (BITS), Pilani
Bachelor of Engineering, Electrical & Electronics Engineering | <Year>


CERTIFICATIONS (in progress / planned)
- AWS Certified Developer – Associate              <plan to take by: YYYY-MM>
- AWS Certified Solutions Architect – Associate    <plan to take by: YYYY-MM>
```

### Honest take on certifications
- **AWS DVA-C02** is high-ROI for SDE2 backend roles. Take it within 60 days. Cost ~$150, 4 weeks of evening prep.
- **AWS SAA-C03** is good but optional at SDE2.
- **Kafka certifications** (Confluent) are nice-to-have, not needed.
- Skip generic "Coursera certificate" lines — recruiters ignore them.

---

## 10. ATS Optimization

ATS (Applicant Tracking System) parses your resume. Most rejections are silent ATS rejections, not human ones.

### Do
- Use standard section headers: "Experience", "Skills", "Education" (not "My Journey", "What I Build")
- Use single column
- Use standard fonts: Helvetica, Calibri, Arial, Inter
- Use bullet points (•) or hyphens (-) — no fancy glyphs
- Include keywords from the JD verbatim where they apply
- Save as PDF (most ATS handle PDF; Word also works)
- Spell out acronyms once: "Background Verification (BGV)"

### Don't
- Use two-column layouts (parsers read top-to-bottom, get confused)
- Use tables for layout (ATS may read them sideways)
- Put critical content in headers/footers (parsers skip these)
- Use graphics, icons, or progress bars
- Use creative fonts or color
- Add a photo (irrelevant for tech in India, breaks parsers)

### ATS test
Paste your PDF text into a plain text editor. If it reads coherently top-to-bottom, the ATS can read it. If it's jumbled, fix the layout.

Free tools: [resumeworded.com](https://resumeworded.com), [jobscan.co](https://jobscan.co) (paid but worth one month if applying heavily).

---

## 11. Common Mistakes to Avoid

| Mistake | Fix |
|---|---|
| Generic summary ("passionate developer who loves coding") | Lead with concrete scope: "Built 7 microservices for 3 banking partners" |
| Listing every technology you've touched | Pick top 20-25 relevant to the role |
| Tasks instead of impact | "Wrote APIs" → "Built fund transfer system supporting NEFT/IMPS/IFT with ₹5L daily limits" |
| No numbers | Add quantities everywhere: commits, services, users, repos, latency |
| Same resume for every role | Tailor the top 5 bullets per role — leave the rest static |
| Listing your current title only | Show progression / scope changes if any |
| Buzzword soup ("synergized stakeholder alignment") | Use engineer language: "Built", "Designed", "Migrated", "Led" |
| Resume photo (in India) | Remove. Period. |
| Two pages for 3 YOE in India | Cut to one page |
| Hobbies / interests section | Leave off unless directly relevant (e.g., contributing to OSS) |

---

## 12. Tailoring Per Role

You don't need 14 versions of your resume. You need **one base resume + a 5-bullet swap rule**.

### The Swap Rule
1. Identify the JD's top 3 keywords (e.g., "Kafka, microservices, AWS")
2. Re-order your top 5 experience bullets to lead with bullets matching those keywords
3. Update your professional summary's stack mention
4. Don't change anything else

### Examples

**For a Kafka-heavy role (Confluent, Stripe, fintech)**
Lead with:
- Kafka MSK migration bullet
- Consumer rebalancing fix bullet
- Event-driven architecture bullet
- ZSTD compression / SDK v2→v3 bullet

**For an AWS-heavy role (any AWS-native shop)**
Lead with:
- AWS Lambda + SAM + EventBridge bullet
- Facets Cloud / K8s consolidation bullet
- Secrets Manager + IP-whitelisted Gateway bullet

**For a system-design-heavy role (FAANG, Atlassian, Razorpay)**
Lead with:
- Architected 5 microservices from scratch bullet
- Multi-channel API architecture bullet
- Dual-auth system (19 files, 1400+ lines) bullet

---

## Quick Reference Cheat Sheet

```
TARGET LENGTH:        1 page (India), 2 pages max (abroad)
SUMMARY VERSION:      A (impact-first) unless JD is keyword-heavy → B
TOP 4 PROJECTS:       Agent Onboarding · Utkarsh · Bharat Customer/VAS · NSDL or Ledger
FILE NAME:            Kunal_Ladhani_SDE2_Resume.pdf
SAVE AS:              PDF (single column, no graphics, no photo)
EMAIL:                Personal (not work)
ATS SCORE TARGET:     80+ on resumeworded.com

GOLDEN BULLET FORMAT:
  Verb + Specific thing + Quantified outcome
  "Built fund transfer system supporting NEFT/IMPS/IFT with
  ₹5L daily limits, OTP auth, and duplicate prevention"

VERBS TO USE:
  Built · Designed · Architected · Led · Migrated · Owned ·
  Implemented · Optimized · Reduced · Established

VERBS TO AVOID:
  Helped · Worked on · Assisted · Participated · Contributed to
  (these signal junior — use them sparingly and only when accurate)

THE TEST:
  Read your resume in 30 seconds. Could a recruiter answer:
  - What's your stack?
  - How senior are you?
  - What's the biggest thing you've built?
  If yes → ship it. If not → cut and reorder.
```

---

*Source archive: `03_resume_content.txt` (original draft).*
