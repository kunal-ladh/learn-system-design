---
title: Capacity Planning
tags: [system-design, observability]
type: note
status: evergreen
created: 2026-10-01
updated: 2026-10-05
summary: How to find your system's real ceiling with load testing, size headroom with N+1 and safety-factor math, apply the USE method, and decide how much overprovisioning an outage is worth.
related: ["[[Autoscaling]]", "[[Back of the Envelope Estimation]]", "[[Scalability]]"]
---

# Capacity Planning

Capacity planning answers a question most teams only answer during an outage: *how much load can this system actually take, and how much will it need to take?* It has two halves — **measuring the real ceiling** (load testing, because the ceiling is never where the architecture diagram says it is) and **sizing against the future** (forecasting, headroom math, and the economics of how much idle capacity an outage is worth). [[Back of the Envelope Estimation]] gives you the paper number; capacity planning is the discipline of replacing paper with measurement.

## Why this matters

Systems fail at their *limits*, and limits are discovered empirically. The architecture handles "10K RPS" until a load test reveals the Mongo connection pool caps out at 3K, or one service's thread pool is sized at 200 and everything behind it is irrelevant. Three forces make planning unavoidable:

1. **Bursts outrun reaction.** [[Autoscaling]] takes minutes to add real capacity (and JVM warmup stretches that); a traffic step is served by whatever headroom existed the second it arrived. Headroom is a *planned* quantity.
2. **Growth compounds quietly.** 15% month-over-month growth doubles load in about 5 months. A system at 60% utilization today is past saturation within two quarters if nobody is watching the trend.
3. **Capacity is money in both directions.** Overprovisioning burns a predictable monthly sum; underprovisioning burns revenue, reputation, and — in regulated fintech — possibly an incident report to the regulator. Planning is choosing the exchange rate deliberately.

## How it works

### Find the real ceiling: load testing

The ceiling is measured by pushing controlled load at a production-like environment until something gives. Four test shapes, four questions:

| Test | Shape | Question it answers |
|---|---|---|
| **Load test** | Expected peak, sustained | Do we meet SLOs at the traffic we claim to handle? |
| **Stress test** | Ramp until failure | Where is the ceiling, what breaks first, and *how* does it fail — graceful or cliff? |
| **Soak test** | Moderate load, hours to days | What leaks? (memory, connections, file handles, disk from logs, slow heap fragmentation) |
| **Spike test** | Instant step to multiples of normal | Does the system survive the burst window before autoscaling helps? Do retries make it worse? |

Tools in one line each: **k6** scripts load in JavaScript with good CI integration; **Gatling** does high-throughput scenarios on the JVM (natural for Java shops). (JMeter is the venerable incumbent; wrk/hey for quick single-endpoint checks.)

Rules that keep results honest: test against production-shaped data volumes and realistic traffic mixes (an all-cache-hits test measures nothing); include downstream dependencies or faithful stubs with production-like latency; ramp gradually and watch *percentiles*, not averages ([[Latency vs Throughput vs Bandwidth]]); and record the failure *mode*, not just the number — a system that degrades by shedding load ([[Load Shedding and Graceful Degradation]]) has a very different risk profile from one that collapses into timeouts and retry storms ([[Failure Modes in Distributed Systems]]). The output you want per service: **max RPS per pod at which p99 stays within SLO** — this number drives both autoscaling targets and the sizing math below.

### The USE method: where is it saturating?

Brendan Gregg's **USE method** is the checklist for interpreting what a loaded system is telling you. For **every resource** (CPU, memory, network, disk IO, and the logical ones — thread pools, connection pools, queue depths), check three things:

- **Utilization** — what fraction of the resource is busy?
- **Saturation** — how much work is *queued* waiting for it? (run-queue length, pool wait time, queue depth)
- **Errors** — is it failing outright? (allocation failures, connection timeouts, dropped packets)

The insight that makes USE valuable: **saturation, not utilization, is the pain signal.** A resource at 100% utilization with no queue is merely busy; a resource at 80% with a growing queue is already hurting — latency lives in the queue. Classic application: a Java service's CPU reads 40% ("fine"), but the HTTP thread pool's queue is growing and connection-pool wait time is climbing — the service is saturated on a logical resource invisible to infrastructure dashboards. Instrument pools as first-class resources (Micrometer exposes pool metrics) or the USE method has nothing to read.

### Headroom math: N+1 and peak × safety factor

Sizing formula, from measurement to replica count:

```
required = ceil( (peak RPS × safety factor) / per-pod capacity ) + redundancy
```

- **Peak RPS** — from history and forecast, not average: plan for the worst hour, not the mean.
- **Safety factor** — covering forecast error, degraded-mode per-pod capacity, and burst-before-autoscaling. A safety factor `f` *is* a **utilization target** of `1/f`, and the two are quoted interchangeably, so do the conversion rather than assuming: 1.25× is an 80% target, 1.4× is ~70%, 2× is 50%. The 50–70% targets this note uses throughout therefore mean a safety factor of roughly **1.4–2×** — and the `+ redundancy` term below pushes realised utilization lower still, which is why a worked example can carry a 1.3 factor and still land inside the band.
- **Redundancy: N+1** (or N+2 at scale) — the fleet must meet peak with one instance lost (deploy, node drain, crash). At small N this dominates: if N=2 pods carry peak, you run 3. The same logic applies a level up as **AZ redundancy**: survive one availability zone's worth of capacity disappearing.

Two pieces of queueing arithmetic make the sizing defensible rather than folkloric:

- **Little's Law**: `concurrency = arrival rate × time in system` (L = λW). At 1,500 RPS with 200 ms average latency, ~300 requests are in flight at any instant — that number must fit inside the fleet's total thread/connection budget, which is how you convert an RPS target into pool sizes. It also runs backward as a sanity check: a pod with 200 worker threads and 200 ms mean latency *cannot* exceed ~1,000 RPS no matter what the CPU graph says, because 200 threads ÷ 0.2 s is the arithmetic ceiling. Use that number as an upper bound only — `W` is not a constant. As you approach the ceiling, queueing inflates time-in-system, so the achievable RPS at an acceptable p99 is always strictly below the Little's Law figure.
- **The utilization-latency curve is nonlinear**: as utilization approaches 100%, queueing delay grows without bound (in the M/M/1 idealization, wait time scales with ρ/(1−ρ) — at 50% utilization the queue term is 1×, at 90% it is 9×). This is the mathematical reason "run everything at 90%" fails: the last fraction of utilization is purchased with your tail latency, which is why utilization targets sit at 50–70%.

### Forecast the growth

Capacity planning is a loop, not an event: plot the demand trend (peak RPS, storage, queue throughput) over months; fit growth (linear or exponential — compounding percentage growth is exponential and *will* surprise a linear extrapolation); mark **lead time** — how long it takes to add capacity (minutes for pods, days for a DB resize, weeks for partner-bank rate-limit increases and anything procurement touches); and act when `projected demand date − lead time` says so, not when utilization crosses a scary line. Business events (product launches, festivals, marketing pushes) are step functions that no trend line predicts — they come from the calendar, and they are planned like the worked example below.

The operational ritual that keeps this alive is a lightweight **capacity review** — quarterly, or monthly for fast-growing flows:

1. Plot peak demand vs measured ceiling per critical service; flag anything projected to cross 70% within the lead-time horizon.
2. Re-validate stale ceilings: any per-pod number older than the last significant release gets a re-run.
3. Walk the event calendar for the next quarter and assign owners for pre-scaling plans.
4. Review the misses: where did last quarter's forecast err, and in which direction? Forecast error is itself a trend worth tracking — it calibrates your safety factor.

### The economics: overprovisioning vs outage

Overprovisioning has a precise, boring cost shape: **idle capacity × unit price × time** — linear, predictable, budgetable. Outages have a lumpy, fat-tailed cost: lost transactions during the window, support and engineering time, customer churn, and in fintech, regulatory exposure and partner-trust damage. Do the comparison with the real multiplier rather than a vibe: dropping from an 80% to a 50% utilization target means `0.8 / 0.5` = **1.6× the fleet**, a 60% larger compute bill for that tier. On a handful of pods that is a few node-hours a day and obviously worth it against one peak-hour payment outage; on a large fleet it is a line item someone will defend in a budget review. Either way the number is knowable in advance, which is the whole argument — one side of this trade can be priced and the other cannot. The honest statement of the trade: you are buying *insurance* whose premium is the idle-capacity bill; choose the deductible (utilization target) per tier — aggressive on stateless easily-scaled tiers, conservative on anything stateful or revenue-bearing.

### Worked example: sizing a fintech flow for a 10× festival-day spike

UPI payment flow; festival sale (think Diwali-scale) projected at **10× normal peak**.

Baseline measurements (the part most teams skip):

- Normal peak: **150 RPS** on payment-service → festival target: **1,500 RPS**.
- Load-tested per-pod ceiling: **120 RPS** with p99 inside SLO; beyond that, latency breaches first (thread-pool saturation — a USE finding, not a guess).
- Mongo primary load-tested at **4,000 writes/s**; each payment ≈ 2 writes → 3,000 writes/s at target: fits, ~75% of ceiling — flagged tight, mitigated by deferring non-critical writes to the async queue.
- Partner bank rate limit: **2,000 TPS contractual** — fits 1,500, but only because someone checked; this is the classic immovable ceiling that pods cannot fix.

Pod math:

```
pods = ceil( (1500 × 1.3 safety) / 120 per pod ) + 1
     = ceil(1950 / 120) + 1  =  17 + 1  =  18 pods   (normal day: ceil(150×1.3/120)+1 = 3)
```

Decisions that follow from the mechanics, not from hope:

- **Pre-scale to 18 before the event window** — a 10× step defeats reactive [[Autoscaling]] (JVM warmup makes cold pods useless during the first minutes); autoscaling handles drift *above* the pre-scaled floor, max replicas raised but capped at what Mongo connections and the bank's 2,000 TPS can absorb.
- Connection math re-checked: 18 pods × pool size must stay under Mongo's connection ceiling — pool sizes trimmed accordingly ([[Connection Pooling]]).
- Spike test at 1,500 RPS *with* a simulated bank slowdown — because the failure mode that kills festival days is the downstream slowdown amplified by retries ([[Timeouts Retries and Exponential Backoff]]), not clean overload. [[Rate Limiting]] at Kong set to shed load above 1,600 RPS into a friendly error rather than letting the queue of doom build.
- Verified the plan with a game day a week out; capacity returned to normal via scheduled scale-down two days after the event (paying for 18 pods all month is how overprovisioning quietly becomes the default).

## Common misconceptions

- **"The architecture handles X RPS."** Architectures don't have throughput; deployed configurations do. The number is a property of pool sizes, instance types, data volume, and the slowest dependency — it changes with every release, which is why mature teams re-run load tests continuously, not annually.
- **"High utilization is efficiency."** Past ~70–80%, queueing effects make latency nonlinear — the last 20% of utilization is bought with your p99. Idle capacity at the right tier is a product feature (survival), not waste.
- **"Autoscaling means we don't need capacity planning."** Autoscaling automates *execution* within a planned envelope (signals, per-pod capacity, max replicas, downstream limits all come from planning). Unplanned autoscaling just finds the next bottleneck faster.
- **"Average load is the planning number."** Systems die at peak; averages are for billing. Plan on peak-hour percentiles plus event calendar.
- **"We load tested last year."** The soak leak, the new feature's extra query, the grown dataset — ceilings decay. A number older than the architecture it measured is folklore.

## In your stack

- Your per-pod ceiling measurement matters double because much of the fleet runs at **1 replica**: there is no N+1, so the single pod's measured ceiling minus current peak *is* your entire burst budget. Capacity planning for those services is the honest substitute for the redundancy you've traded away for cost.
- The immovable ceilings in your chain are rarely pods: partner-bank TPS contracts, Mongo primary write throughput, and Redis memory are the real festival-day constraints. Check them first — they have lead times of weeks (commercial negotiation), not minutes.
- Queue-based flows (SQS → Redis Streams) shift the capacity question from "RPS" to "drain rate vs arrival rate": plan consumer capacity so oldest-message age stays inside the staleness SLO at peak arrival, and spike-test the backlog-drain scenario ([[Backpressure and Flow Control]]).
- Gatling is the natural load-testing fit for a Java shop (JVM, reusable with your CI), k6 if the team prefers JS scripting; either way the deliverable is the per-pod RPS-at-SLO number per service, refreshed on a cadence.
- RBI-regulated money movement makes the outage side of the cost equation heavier than typical SaaS: a festival-day payment outage is a regulator conversation, not just lost orders. That justifies conservative utilization targets on the payment path that would look "wasteful" elsewhere — name the cost shape explicitly when proposing it.

## Interview angle

The 60-second answer: "Capacity planning is measurement plus arithmetic plus economics. Measure the real ceiling with load tests — load for SLO-at-peak, stress for where-it-breaks, soak for leaks, spike for burst survival — producing a per-pod RPS-at-SLO number. Diagnose with the USE method: for every resource, physical and logical, check utilization, saturation, errors — saturation is the pain signal, and thread/connection pools are the resources everyone forgets. Size as peak × safety factor ÷ per-pod capacity, plus N+1 redundancy, running at 50–70% target utilization because bursts outrun autoscaling. Forecast growth against capacity lead time, pre-scale for calendar events, and frame the whole thing economically: overprovisioning has a linear predictable cost, outages have a fat-tailed one — you're choosing an insurance premium."

Classic follow-ups: "Utilization vs saturation?" (busy vs queued; queues are where latency lives). "Why not run at 90%?" (queueing nonlinearity, no burst budget, no room for N−1). "How do you size for a known 10× event?" (measure per-pod ceiling, check *downstream* ceilings, pre-scale with safety factor, spike-test including degraded-downstream mode). "What breaks first in practice?" (logical pools and downstream limits, almost never CPU).

## Check yourself

> [!question]- Q: Explain the USE method and why it catches problems that utilization dashboards miss.
> For every resource — CPU, memory, disk, network, plus logical resources like thread pools, connection pools, and queues — check Utilization (how busy), Saturation (how much work is queued waiting), and Errors (outright failures). Utilization dashboards miss two things: logical resources (a thread pool can be saturated while every infrastructure graph is green), and the difference between busy and queued — a resource can be far from 100% utilized while a queue builds against it, which is exactly when latency starts climbing. Saturation is the leading indicator; utilization alone is neither necessary nor sufficient for pain.

> [!question]- Q: What happens if you run your payment fleet at 85% utilization to save cost, and traffic steps up 1.5× for ten minutes?
> The step takes you to ~127% of capacity instantly. Requests queue; latency goes nonlinear and p99 breaches first; thread pools and connection pools saturate; timeouts begin, and upstream retries amplify the overload. Autoscaling reacts within a minute but new JVM pods need minutes to schedule and warm — most of the ten-minute window is served by a fleet 27% past its ceiling, likely in a degraded spiral unless load shedding caps intake. The capacity "saved" bought a guaranteed SLO breach on any 1.2×+ surprise; headroom was the insurance you cancelled.

> [!question]- Q: Your soak test passes at 4 hours but production degrades every ~3 days. What class of problem is this, and how do you hunt it?
> A slow leak — the soak was too short to express it. Candidates: heap growth (old-gen creep → GC pressure → latency), connection or file-handle leaks, unbounded caches, log-volume filling disk, or fragmentation. Hunt: extend the soak (run over a weekend), plot the USE saturation metrics over days, compare heap/GC/connection-count/fd trends between hour 1 and hour 48 — a leak is a trend, not a threshold. In production, the tell is the sawtooth that resets on every deploy: if restarts "fix" it, you're leaking something.

> [!question]- Q: Mini-design — your lending product expects a 10× application spike on a marketing-blitz weekend. Produce the capacity plan outline.
> (1) Baselines: current peak RPS per service in the application flow; load-test each for per-pod RPS-at-SLO. (2) Walk the chain for fixed ceilings: Mongo write throughput, credit-bureau API rate limits (contractual — weeks of lead time to raise), KYC-vendor TPS, Redis memory. The lowest ceiling defines feasibility; negotiate or queue around it. (3) Pod math per service: 10× peak × 1.3 ÷ per-pod capacity, +1; verify connection-pool totals fit downstream limits. (4) Pre-scale before the window; autoscaling for drift above the floor; Kong rate limit just above target to shed, not collapse. (5) Spike test the full flow at target *with* a slow bureau simulated; game day a week out. (6) Scheduled scale-down after, and a one-page cost note: the weekend's extra node-hours vs the revenue of the campaign — the premium is trivially justified; say so with numbers.

## Sources

- Brendan Gregg, "The USE Method" (brendangregg.com) and *Systems Performance* (2nd ed., 2020) — the USE checklist and saturation-first analysis.
- Google SRE Book (2016), ch. 18 "Software Engineering in SRE" (capacity planning context) and *SRE Workbook* (2018), ch. 11 "Managing Load".
- AWS Builders' Library — "Using load shedding to avoid overload" and "Reliability, constant work" essays on headroom and overload behavior.
- k6 and Gatling official documentation — test types and scripting models.
- *Designing Data-Intensive Applications* (Kleppmann), ch. 1 — percentiles, SLO framing, and why tail latency governs capacity.
