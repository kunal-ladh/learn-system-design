---
title: Disaster Recovery
tags: [system-design, resilience]
type: note
status: evergreen
created: 2026-10-01
updated: 2026-10-02
summary: RPO and RTO, backup strategies that actually restore, and the DR tier ladder from backup-restore to multi-region active-active — with the cost shape and data-conflict gotchas of each rung.
related: ["[[Replication]]", "[[Failover]]", "[[Failure Modes in Distributed Systems]]"]
---

# Disaster Recovery

Everything else in this module defends against *components* failing. Disaster recovery (DR) plans for the day a whole layer disappears: a region outage, a deleted production database, ransomware, a bad migration that corrupts data silently for a week. DR is the discipline of answering two questions *before* that day — how much data can we lose, and how long can we be down — and then buying exactly enough infrastructure to honor the answers. The spectrum runs from "restore last night's backup" to "the other region is already serving traffic", and each rung multiplies cost.

## The problem that created it

A growing company runs everything in one region with nightly database backups "configured years ago." Three stories end this arrangement. One: an engineer runs a destructive migration against prod instead of staging — [[Replication|replication]] faithfully replicates the destruction to every follower within milliseconds, because **replication is not backup**: it protects against machine death, not against bad writes, which it copies perfectly. Two: the region itself has a bad day (the 2025 us-east-1 event reminded everyone that regions are single failure domains — [[Failure Modes in Distributed Systems|correlated failure]] at the largest scale). Three — the most common: the backups ran nightly for years, and the first actual restore attempt reveals they're unusable: wrong permissions, missing a critical database, never completing, or silently empty. **An untested backup is not a backup; it's a hope with a cron schedule.**

## What it solves

- **Data-destruction recovery**: bad writes, deletions, corruption, ransomware — the failures replication *spreads* instead of stopping.
- **Site-level failure**: region/datacenter loss, with a pre-agreed path back to service.
- **Bounded, pre-negotiated loss**: business and engineering agree on numbers *before* the incident, so the 2 a.m. decisions are executions, not debates.
- **Regulatory survival**: auditors in finance ask for RPO/RTO, DR drill evidence, and data-residency guarantees by name.

## How it works

### RPO and RTO: the two numbers everything hangs on

- **RPO (Recovery Point Objective)** — maximum acceptable *data loss*, measured backwards in time: "we can lose at most the last N minutes of writes." With nightly backups, your RPO is up to 24 hours. With continuous WAL shipping, minutes. With synchronous cross-region replication, approximately zero.
- **RTO (Recovery Time Objective)** — maximum acceptable *downtime*: time from disaster to serving again. Restore-from-snapshot might be hours; failover to a hot standby, minutes.

Example that fixes the definitions: a payment ledger might demand RPO = 0 (losing an acknowledged transaction is unacceptable — the money moved at the partner bank whether or not you remember it) but tolerate RTO = 1 hour (painful, survivable). A marketing-analytics store can run RPO = 24 h, RTO = days. **Set them per system, not per company** — paying the ledger's requirements for the analytics store is how DR budgets die. Mnemonic: RPO is how much *past* you lose; RTO is how much *future* you lose before service resumes.

### What RTO actually decomposes into

The most common DR planning error is quoting the *restore* time as the RTO. Restore is one term in a sum, and frequently not the largest. Suppose a 500 GB MongoDB dataset, a restore pipeline sustaining 100 MB/s, and an organisation that behaves reasonably:

| Phase | Supposed duration | Why it gets left out of the plan |
|---|---|---|
| Detection | 10 min | Something has to notice. Alerting on data *corruption* is far weaker than alerting on availability |
| Decision | 10 min | "Do we restore?" is a judgement with real blast radius; it needs a woken human with the authority to make it |
| Provisioning the target | 15 min | The cluster does not exist yet — that is what makes this a cold tier |
| Restoring the base backup | ≈83 min | 500 GB at 100 MB/s is 5,000 seconds. This is the number people quote as "the RTO" |
| Replaying logs to the target point | 20 min | PITR's price: oplog or WAL replay grows with how far past the snapshot you restore |
| Application start and warm-up | 10 min | Pods, connection pools, caches. A cold system is not yet a serving system |
| DNS and client cutover | 5 min | Bounded by TTLs chosen long before the incident ([[How DNS Works]]) |

The sum is **153 minutes — 2 hours 33 minutes** — against a restore step of 83. Quote the 83 and you have understated the commitment by nearly two-fold, in a document an auditor may one day hold you to. Two things follow. The cheap improvements all live in the *non-restore* terms: better corruption detection, a pre-agreed decision owner, infrastructure-as-code that provisions without a meeting, shorter DNS TTLs. And every figure in that table is a supposition until a drill replaces it with a measurement — which is the whole case for drills in one sentence.

### Backup strategies, in increasing sophistication

| Technique | Mechanics | RPO it buys | Watch out for |
|---|---|---|---|
| **Snapshots** | Periodic block-level image (EBS, Atlas snapshot) or logical dump (`mongodump`, `pg_dump`) | Up to one full interval | Consistency across multiple databases snapshotted at different instants; `mongodump` without `--oplog` is not a point-in-time image |
| **PITR (point-in-time recovery)** | Base snapshot + continuous log shipping — Postgres WAL, MySQL binlog, or MongoDB's **oplog** — replayed to any second | Minutes → seconds | Log retention window; replay time grows with log volume |
| **Cross-region copies** | Backups replicated to another region (and ideally another *account*) | Same as source, but survives region loss and credential compromise | Egress cost; copy lag; same-account copies die with the account |
| **Immutability/air-gap** | Write-once storage (S3 Object Lock), separate credentials | — | Without it, ransomware or a bad actor deletes the backups too |

PITR deserves emphasis for the bad-migration case: restore to *the second before* the destructive statement, not to last night. And the only metric that counts: **restores tested, regularly, end-to-end, with a measured duration**. A quarterly drill that restores prod backups into a scratch environment and runs integrity checks converts "we have backups" from belief to fact, and replaces every supposed duration in the table above with a measured one.

### The DR tier ladder

The standard four rungs (AWS's framing), each buying better RPO/RTO for multiplied cost:

```mermaid
graph LR
    A["Backup and restore"] --> B["Pilot light"]
    B --> C["Warm standby"]
    C --> D["Multi-region active-active"]
```

| Tier | What runs in region B | RTO / RPO (rough) | Cost shape |
|---|---|---|---|
| **Backup & restore** | Nothing — backups stored there | Hours–days / since last backup (minutes with PITR) | Storage only: scales with data size, near-zero compute |
| **Pilot light** | Data replicated continuously; minimal core (DB replica) running; app infra defined-but-off | Tens of minutes–hours / seconds–minutes | Storage + one small always-on DB replica + egress; compute only on activation |
| **Warm standby** | A **scaled-down but fully working** copy serving no (or token) traffic; scale up to take over | Minutes / seconds | A fixed fraction (say 10–30%) of prod running 24/7, scaled up to full production capacity on failover |
| **Active-active** | Full capacity in multiple regions, **all serving live traffic** | Near-zero / near-zero | ≥2× full stack everywhere + cross-region data transfer + the engineering tax below |

Cost-shape honesty: each rung roughly shifts cost from *incident-time* to *always-on*. Backup-restore is cheap every day and expensive (in downtime) on the bad day; active-active pays every single day — compute ×N regions, continuous inter-region replication egress, and an unbounded line item: engineering complexity. Also note the quiet failure mode of the middle tiers: a standby that's never exercised drifts (stale AMIs, missing config, expired certs) until it's a pilot light that won't ignite — regular failover *drills* are part of the price, not an optional extra ([[Chaos Engineering]]).

### The data layer is the hard part (active-active gotchas)

Stateless compute in two regions is easy ([[Stateless vs Stateful Services]]); the database is where active-active bites. If both regions accept writes, you've built **multi-leader replication**, and the same user's (or two users') writes can conflict: both regions sell the last seat, both update the same wallet balance, each ordering writes differently. Now you need conflict detection and resolution — last-writer-wins (silently loses data), CRDTs (limited data shapes), or application-level merge (hard) — the full story is in [[Replication]]. Asynchronous cross-region replication also means **failover itself loses the replication lag's worth of acknowledged writes** (your real RPO is the lag), and split-brain looms if both regions think they're primary ([[Failover]], [[Split Brain and Fencing]]). Common honest compromise: **active-passive per dataset** — both regions serve, but each record/tenant/shard has one home region for writes; or accept region-pinned users. For a fintech ledger, "last writer wins" is not a conflict strategy, it's an unreconciled book.

### Failback: the forgotten second half

Every DR plan describes failing *over*; few describe coming *home*, and failback is often harder. While region B served traffic, it accumulated writes that region A never saw — so failback is a reverse migration: re-sync A from B (full restore or catch-up replication), prove A is current, then cut traffic back, ideally during a low-traffic window with the same ceremony as the original failover. Three traps: **dual-write confusion** if A was only partially down and kept taking some writes (now you have a merge problem, not a copy problem); **DNS and client caches** sending stragglers to A before it's ready or to B after it's retired ([[How DNS Works]] — TTLs are your failback schedule); and **the standby being the primary now** — if you stay on B for weeks, B needs its own DR story immediately, not eventually. Plans that skip failback leave you running indefinitely on the region with the least hardened tooling.

### What a real restore drill checks

A quarterly drill is worth more than any architecture diagram. The checklist that makes it real:

1. Restore into a *separate* environment/account from the actual backup artifacts (not a replica) — proving the artifacts alone suffice.
2. Time every phase: locate backup → provision → restore → replay logs to target point → integrity checks → app boots against it. The sum is your measured RTO contribution.
3. Verify *semantic* integrity, not just row counts: balances sum correctly, foreign references resolve, the last N transactions before the restore point exist.
4. Rotate who runs it — the drill also tests the runbook's ability to be executed by someone who didn't write it.
5. File the results where auditors (and future you) can find them; a drill without a written result might as well not have run.

### RBI / fintech data-residency note

For India-regulated payment data, DR design has a jurisdictional constraint: RBI's 2018 directive on Storage of Payment System Data requires payment-system data to be stored in India (for cross-border transactions, a foreign copy of the foreign leg is permitted). Practical consequences: your DR region must also be in India (the standard AWS shape is Mumbai `ap-south-1` paired with Hyderabad `ap-south-2`), "just replicate to Singapore" is a compliance incident rather than a DR plan, and backup copies — including the cross-region, cross-account ones — inherit the same residency requirement. Data-localisation scope belongs in the DR design review, not discovered during the audit.

## Trade-offs

| You gain | You pay |
|---|---|
| Survival of region loss and data destruction | Always-on cost rising steeply per tier |
| Pre-agreed loss/downtime numbers | Drills, runbooks, and standby-drift maintenance forever |
| Auditor-ready evidence | Active-active: conflict resolution complexity leaking into application code |
| Bounded blast radius for the worst days | Cross-region latency/consistency physics ([[PACELC]]) |

## Failure modes

DR's failure modes are unusual in one respect: most of them are *invisible until exercised*, which is why the detection advice here keeps reducing to "run the drill".

- **The backup job is green and the artifact is unrestorable.** The job's success criterion is "the command exited zero", which is not the same claim as "a working database can be produced from this". *Signal*: none exists — the failure is silent by construction. *Confirm*: only a restore attempt confirms it. This is the entire argument for scheduled restore testing: it converts a silent failure into a loud one, on your calendar instead of the disaster's.
- **Retention window shorter than detection latency.** PITR reaches back only as far as retained oplog or WAL. Silent corruption discovered on day 10 against a 7-day window means every reachable point is already contaminated. *Signal*: the gap between "when did this start" and "how far back can we go" is typically discovered mid-incident, which is the worst possible moment. *Confirm*: compare retention settings against your realistic detection latency for *corruption* — weeks — not for outages, which you catch in minutes.
- **Backups sharing fate with production.** Same account, same credentials, same region, no immutability. One compromised principal or one region loss takes the backups along with the thing they protect. *Signal*: an inventory question nobody has asked — "what single actor could destroy both?" *Confirm*: enumerate every principal that can delete a backup and check whether any of them can also damage production. An overlap is the finding.
- **Standby drift.** The never-exercised warm standby accumulates stale images, missing config, expired certs and capacity assumptions that stopped being true a year ago. *Signal*: the drill fails on something mundane — a secret, a certificate, an absent environment variable — rather than on anything to do with data. *Confirm*: that mundane failure *is* the confirmation. Drift is only ever detected by use.
- **Failback dual-write.** The primary was never entirely dead and quietly accepted some writes while the DR region served the rest. *Signal*: after failback, records exist in one region and not the other with overlapping timestamps. *Confirm*: count writes per region across the incident window *before* cutting back — a non-zero count in the supposedly-dead region turns a copy into a merge, and a merge needs a plan and probably a reconciliation.
- **A plan whose first real execution is the disaster.** *Signal*: a runbook with no recorded execution date. *Confirm*: ask who last ran it, and when. If the answer is a name without a date, there is no plan — there is a document.

## When NOT to use it

- **Don't buy active-active for an RTO of hours.** Work backwards from stated RPO/RTO; most systems' honest numbers are met by PITR + pilot light.
- **Don't run one DR tier company-wide** — tier per system criticality: ledger ≠ CMS.
- **Don't confuse HA with DR**: multi-AZ replicas handle machine/AZ failure invisibly ([[Availability Reliability Durability]]); DR handles region loss and *data destruction*. You need backups even with perfect replication — especially with perfect replication, since it perfectly replicates your mistakes.
- **Don't design DR for a system you can rebuild faster from source** (stateless services, derived caches): recovery there is redeploy + rewarm, not restore.

## Common misconceptions

- **"RPO is a property of the backup schedule."** RPO is a property of what you can actually *restore to*, which is governed by log retention and, on a failover path, by the replication lag at the instant of failure. A backup running every five minutes buys nothing if the logs needed to replay past it were rotated away, and a cross-region standby's real RPO is whatever it happened to be behind by — a number that moves with load and is largest exactly during the incidents that trigger failover.
- **"Snapshots taken at the same time are a consistent set."** Two databases snapshotted at the same wall-clock instant are each internally consistent and mutually arbitrary: an order recorded in one may have no matching payment in the other, because the write crossed the boundary between the two snapshot instants. Restoring both and assuming they agree produces a database that never existed in production. Either snapshot a single consistent boundary, or plan a reconciliation pass as part of the restore.
- **"A cross-region copy is an isolated copy."** Only if it is also cross-*account* and immutable. A backup in another region of the same account is protected against a region failing and against nothing else — not a compromised credential, not an automation bug, not an over-broad lifecycle rule. Region is one axis of isolation; identity and mutability are the other two, and they are the ones that fail in the incidents people actually have.
- **"The standby is sized for our traffic."** It was sized for the traffic of whenever it was built. Production grows and the standby does not, because nothing exerts pressure on a system serving no load. A standby that was 30% of prod two years ago may be 8% today, which means failover does not restore service — it relocates the outage. Capacity assumptions belong in the drill's success criteria, not just its plumbing.
- **"DR is for disasters."** The overwhelmingly most frequent use of a restore is not a region loss — it is one human and one bad statement against one collection. Design for *targeted* recovery too: restoring a single collection into a scratch environment and merging it back is a different, more common, and much less rehearsed operation than full-site recovery, and a plan that only describes the latter leaves you improvising the former under pressure.

## In your stack

- Your fleet's honest baseline: single-region, many single-replica services — so DR posture rests almost entirely on the *data layer*: MongoDB backups with oplog-based PITR (Atlas continuous backup if on Atlas), tested restores, and cross-account immutable copies. The stateless Spring services redeploy from ECR in minutes; the data cannot.
- Run the restore drill that measures your true RTO: Mongo restore into a scratch cluster + EKS redeploy + Kong/DNS cutover, timed end-to-end. Until it's measured, your RTO is a guess.
- Per-system RPO/RTO tiering writes itself: transaction/ledger stores (RPO≈0 → oplog PITR + possibly a second-region replica), KYC documents in S3 (versioning + cross-region replication within India + Object Lock), Redis (cache: RPO=∞, rebuildable — but anything *non-cache* in Redis, like [[Redis Streams]] queues, needs an explicit answer), SQS backlogs (transient by design — but in-flight messages at disaster time must be re-derivable or reconciled).
- Residency: both primary and DR copies of payment data stay in India (Mumbai/Hyderabad shape) per the RBI directive above — flag any replication target outside India as new regulatory scope.
- Reconciliation with partner banks is part of DR: after any restore-to-point-in-time, the window between restore point and incident must be re-reconciled against partner records — the ledger's RPO is really "what we can re-derive from partners", which is why settlement files and webhooks should be archived immutably ([[Stripe and Payment System Design]]).

## Interview angle

- **60-second version**: "Two numbers drive everything: RPO — how much data you may lose — and RTO — how long you may be down, set per system. Backups answer data destruction (replication doesn't — it replicates the mistake), with PITR via WAL/oplog getting RPO to seconds, and untested backups counting as no backups. Site loss is answered by the tier ladder — backup-restore, pilot light, warm standby, active-active — each trading always-on cost for recovery speed. Active-active's hard part is the data layer: dual-region writes mean multi-leader conflicts and lag-bounded RPO, so many 'active-active' systems are really active-passive per dataset."
- Classic follow-ups: "Why isn't replication enough?" (bad writes replicate); "What's your actual RPO with async cross-region replication?" (the replication lag at failure time); "How do you know your RTO?" (measured drills, nothing else); "Walk me through a failback" — the forgotten half: returning home after the DR region accumulated writes.
- Fintech-flavored: expect data-residency and reconciliation questions — naming RBI localisation unprompted is a strong signal in India-market interviews.

## Check yourself

> [!question]- Q: Explain RPO vs RTO with a concrete example of a system where one is near-zero and the other is relaxed.
> RPO bounds data loss (how far back the restored state may be); RTO bounds downtime (how long until serving again). A payment ledger: RPO ≈ 0 — an acknowledged transaction must never vanish, because the money actually moved at the partner — but RTO of an hour is survivable (queue inbound work, reconcile after). Inverse example: a live sports scoreboard — RTO near-zero (being down during the match is the product failing) but RPO of minutes is fine (lost score updates re-derive from the feed).

> [!question]- Q: What happens if you rely on replication alone and an engineer drops a production collection?
> Every follower applies the drop within replication-lag time — milliseconds to seconds — because replication's job is faithfully copying writes, including destructive ones ([[Replication]]). Now all copies agree the data is gone. Recovery requires a *backup*: restore the last snapshot and replay the oplog/WAL to the second before the drop (PITR). Delayed replicas (followers applying changes on a deliberate lag) are a partial hedge, but bounded by their delay window and no substitute for tested backups.

> [!question]- Q: Why does a warm standby that's never failed over to tend to decay into one that can't take traffic?
> Drift: the standby misses config changes applied only to prod, runs stale images/AMIs, has unrotated secrets and expired certs, its scaled-down capacity assumptions fall behind traffic growth, and DNS/runbook steps go stale as the primary evolves. Nothing exercises it, so nothing surfaces the rot — the standby silently degrades from warm standby to decorative line item. The fix is institutional, not technical: scheduled failover drills where the standby takes real (or realistic synthetic) traffic, with the drill's success criteria being measured RTO/RPO.

> [!question]- Q: A bad deploy has been writing subtly wrong values into one Mongo collection for eleven days before anyone notices. PITR retention is 7 days; nightly snapshots are kept for 35. What can you actually recover, and what is the lesson?
> PITR cannot reach a clean point: its oldest restorable second is 7 days ago, which is four days *after* the corruption began, so every point it can reach is already contaminated. What remains is the nightly snapshot ladder — a snapshot from day 12 or earlier is clean and well inside the 35-day retention. But restoring it wholesale rolls the cluster back roughly twelve days, discarding every write in that period, the overwhelming majority of which were correct. So the realistic recovery is not a full restore at all: bring the day-12 snapshot up in a *scratch* cluster, extract the affected collection's clean state, and reconcile forward into production — replaying legitimate changes from application logs, event streams, or partner records. Two lessons. First, retention windows must be sized against detection latency for *silent corruption*, which is measured in weeks, rather than against outage detection, which is measured in minutes. Second, for money systems this recovery is only possible if the correct values are re-derivable from somewhere outside the corrupted store, which is why immutable settlement and event archives belong in the DR design and not in the analytics backlog.

> [!question]- Q: Mini design: DR for a UPI-adjacent payment service — Mongo ledger, S3 KYC docs, SQS queues, single region today. Assign tiers and justify.
> Ledger (RPO≈0, RTO≤1h): continuous oplog backup/PITR, cross-region replica *within India* (Mumbai→Hyderabad), immutable cross-account snapshot copies; failover runbook + quarterly restore drill; post-restore partner reconciliation procedure. KYC docs (RPO minutes, RTO hours): S3 versioning + Object Lock + CRR to the India DR region. SQS: redundant across AZs inside its region but with no cross-region replication to buy, so a region loss loses the backlog — producers must tolerate re-send and consumers need [[Idempotency|idempotency]]; document that in-flight messages at disaster time are re-derived from source systems. Compute: pilot light — ECR images replicated, infra as code ready, nothing running. Name the cost shape: storage + one DB replica + egress always-on; full compute only on activation. Everything stays in-country per RBI localisation.

## Sources

- AWS Well-Architected whitepaper, "Disaster Recovery of Workloads on AWS: Recovery in the Cloud" — the four-strategy ladder this note follows.
- Google SRE Book, ch. 26 "Data Integrity" — the 24 combinations of failure × recovery, and why restores (not backups) are the objective.
- *Designing Data-Intensive Applications* (Kleppmann), ch. 5 — replication, multi-leader conflicts, and lag; the theory under the active-active gotchas.
- RBI circular "Storage of Payment System Data" (DPSS.CO.OD No.2785/06.08.005/2017-18, April 2018) — the India data-localisation directive.
- MongoDB documentation: oplog-based point-in-time restore / Atlas Continuous Cloud Backup.
