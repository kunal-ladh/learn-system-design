# Observability 101 — Concepts First

> A ground-up learning guide for engineers new to observability.
> Covers: latency · availability · throughput · percentiles · metrics · gauges · Prometheus · OpenTelemetry · open source stack.
> Stack: Spring Boot 3 → 4 · Java 17 → 21 → 25 · **no vendor lock-in**.
>
> For implementation code and SME service specifics, see [`observability-guide.md`](./observability-guide.md).

---

## Table of Contents

1. [What Is Observability? (And Why Not Just "Monitoring"?)](#1-what-is-observability)
2. [The Four Golden Signals](#2-the-four-golden-signals)
   - [Latency](#21-latency)
   - [Throughput](#22-throughput)
   - [Error Rate](#23-error-rate)
   - [Saturation](#24-saturation)
3. [Availability — The Fifth Must-Know Metric](#3-availability)
4. [Percentiles — Why Averages Lie](#4-percentiles--why-averages-lie)
5. [SLI, SLO, SLA](#5-sli-slo-sla)
6. [The Three Pillars of Observability](#6-the-three-pillars)
7. [Metrics — Every Type Explained](#7-metrics--every-type-explained)
8. [Logs — Structured and Correlated](#8-logs)
9. [Traces — The Journey of a Request](#9-traces)
10. [How the Three Pillars Connect](#10-how-the-three-pillars-connect)
11. [Prometheus — Pull-Based Metrics](#11-prometheus)
12. [OpenTelemetry — The Vendor-Neutral Standard](#12-opentelemetry)
13. [The Open Source Observability Stack](#13-the-open-source-stack)
14. [Vendor Lock-In: The Problem and the Escape](#14-vendor-lock-in)
15. [Spring Boot & Java Integration Roadmap](#15-spring-boot--java-integration-roadmap)
16. [Glossary](#16-glossary)
17. [Further Reading](#17-further-reading)

---

## 1. What Is Observability?

**Observability** is the ability to understand what is happening *inside* a system just by looking at its *outputs* — without needing to SSH into a server or add debug logs for every scenario.

The word comes from control theory. A system is "observable" if you can determine its internal state from its external outputs alone.

**Monitoring vs Observability — the key difference:**

| | Monitoring | Observability |
|--|------------|---------------|
| Tells you | *That* something is broken | *Why* it is broken |
| Questions answered | "Is the service up?" | "Why are payouts slow only for NEFT transfers on salary day?" |
| Requires predefined alerts | Yes — you had to anticipate the failure | No — you can explore unknown failures |
| Tool analogy | Dashboard with gauges | A microscope you can aim at anything |

**A concrete example with the SME service:**

You get a PagerDuty alert: "salary payouts failing." 

- With only **monitoring**: You SSH into pods, grep logs for ERROR, guess at the cause. Takes 45 minutes.
- With **observability**: You look at the `bharat.sme.cardhost.debit.duration` histogram, see p99 spiked to 12s at 9:02 AM. You pull the trace for one failed transaction. The trace shows the card host API call hung for 11 seconds before timing out. The logs for that trace show `responseCode=504`. Root cause found in 3 minutes.

**The foundation is three types of data** — metrics, logs, and traces — which together give you a complete picture. We'll cover each in depth below.

---

## 2. The Four Golden Signals

Google's Site Reliability Engineering (SRE) book defined the **Four Golden Signals** — the minimum set of measurements that tells you whether your service is healthy. These four signals should always be measured before anything else.

---

### 2.1 Latency

**What it is**: The time it takes to handle a single request from the moment it arrives to the moment the response is sent.

Think of a restaurant: a customer places an order (request arrives), the kitchen cooks (processing), and the food is delivered (response sent). Latency is the total wait time.

```
A single salary payout request:

t=0ms    ─── Request arrives at /payouts/initiate
t=2ms    ─── JWT validated
t=8ms    ─── MongoDB: load corporate wallet
t=12ms   ─── MongoDB: create transaction record
t=190ms  ─── HTTP: POST card-host.internal/debit   ← most of the time is here
t=205ms  ─── MongoDB: update transaction status
t=210ms  ─── Kafka: publish to success topic
t=212ms  ─── Response sent (HTTP 200)

Latency = 212ms
```

**Why it matters**: Latency is invisible without measurement. Your service could be "returning 200 OK" but taking 8 seconds per request — users see timeouts, retries pile up on Kafka, workers back up. Latency tells you *how fast* you are, not just *whether* you work.

**Types of latency you care about in SME service:**

| Type | Measured At | Typical Healthy Value |
|------|------------|----------------------|
| API response latency | Controller layer | p99 < 500ms |
| Card host debit latency | Retrofit client | p99 < 2s |
| YBL bank transfer latency | Retrofit client | p99 < 5s |
| MongoDB query latency | Repository layer | p99 < 50ms |
| Kafka consumer processing latency | Worker | p99 < 1s per message |

**Key insight — latency and errors are different.** A request that times out (returns after 30s) has **high latency**. A request that returns HTTP 500 immediately has **zero latency but an error**. Track both independently.

---

### 2.2 Throughput

**What it is**: The number of operations your service handles per unit of time. Usually expressed as **requests per second (RPS)** or **transactions per second (TPS)**.

Think of it like a highway:
- **Latency** = how long it takes one car to drive from A to B
- **Throughput** = how many cars pass the toll booth per hour

```
SME service throughput across a salary day:

06:00 — 12 payouts/sec    (night, low traffic)
09:00 — 45 payouts/sec    (morning, normal)
10:00 — 850 payouts/sec   (salary run started — 16x spike!)
10:30 — 820 payouts/sec   (salary run ongoing)
11:00 — 90 payouts/sec    (salary run complete)
14:00 — 40 payouts/sec    (afternoon, normal)
```

**Why it matters:**
1. **Capacity planning**: If you handle 850 tx/sec today, and the business grows 3x, will the service survive?
2. **Diagnosing latency**: Is latency high because of a bug (happens at any throughput) or because of overload (only happens above 600 tx/sec)?
3. **Alerting on drops**: If throughput suddenly falls to 0 during salary hours, payouts have stopped silently.

**Throughput in Prometheus (PromQL):**
```promql
# Salary debits per second, averaged over last 5 minutes
rate(bharat_sme_salary_transfer_debit_total[5m])
```

---

### 2.3 Error Rate

**What it is**: The percentage of requests that result in an error (failed business outcome, HTTP 5xx, exception, or timeout).

```
Error rate = (failed requests ÷ total requests) × 100

Example:
  Total salary debits:   10,000
  Failed:                    50
  Error rate:              0.5%
```

**Why it matters**: Error *rate* lets you compare across time and volume. "50 failures" is meaningless without context — but "0.5% failure rate, up from 0.1% baseline" clearly signals a problem.

**The danger of thinking in absolute numbers only:**

| Scenario | Absolute | Rate | Verdict |
|----------|----------|------|---------|
| Salary day: 50 failures out of 10,000 | Sounds concerning | 0.5% — within SLO | OK |
| Quiet night: 50 failures out of 100 | Sounds the same | 50% — catastrophic | Alert! |

**Business impact framing**: Always translate the rate back to absolute numbers for stakeholders:
> "Our error rate is 0.5% on a 10,000-transaction run — that's 50 employees who didn't receive their salary. We need to fix it."

**Types of errors to distinguish:**

| Category | Example | Resolution |
|----------|---------|------------|
| Infrastructure errors | MongoDB timeout, Kafka unavailable | Ops/infra team |
| External dependency errors | Card host 504, YBL 500 | Partner bank issue |
| Business logic errors | Insufficient funds, invalid IFSC | Expected, track separately |
| Client errors | HTTP 400, invalid request | Usually a calling system bug |

---

### 2.4 Saturation

**What it is**: How close a resource is to its **capacity limit**. When a resource is saturated, latency and errors follow — saturation is a *leading indicator* that lets you act before users are impacted.

```
Saturation example — Kafka consumer lag:

09:58 — lag=0       (workers consuming faster than messages arrive)
10:00 — lag=500     (salary run starts, producers slightly faster)
10:02 — lag=5,000   (consumers saturating — can't keep up!)
10:03 — lag=25,000  (fully saturated)
10:04 — latency spikes from 200ms to 8,000ms
10:05 — error rate rises (transaction timeouts)

If you had alerted on lag > 1,000 at 10:00, you'd have had 2 minutes
to scale up workers before users felt anything.
```

**Key resources to monitor for saturation:**

| Resource | Metric | Saturation Threshold |
|----------|--------|---------------------|
| JVM Heap | `jvm.memory.used` / `jvm.memory.max` | > 80% → GC storms incoming |
| CPU | `process.cpu.usage` | > 85% → thread starvation |
| DB Connection Pool | `hikaricp.connections.pending` | > 0 → threads waiting for connections |
| Kafka Consumer Lag | `kafka.consumer.records.lag` | Growing trend = saturating |
| HTTP Thread Pool | Active threads / max threads | > 90% → requests queuing |

**The critical insight**: Saturation predicts the *next* failure. By the time latency and error rate spike, you're already in the incident. Saturation alerts buy you time.

---

## 3. Availability

**What it is**: The percentage of time your service is correctly handling requests. Usually expressed as a percentage over a rolling 30-day window.

```
Availability table (must memorize these):

99%     → 7.3 days of downtime/year     (totally unacceptable for payments)
99.9%   → 8.76 hours of downtime/year   ("three nines" — minimum for a payments API)
99.95%  → 4.38 hours of downtime/year
99.99%  → 52.6 minutes of downtime/year ("four nines" — target for salary processing)
99.999% → 5.26 minutes of downtime/year ("five nines" — core banking standard)
```

**Availability is NOT just "is the process running."** A service that is returning HTTP 500 to every request is "up" (process is running) but has 0% availability. The correct formula:

```
Availability = successful requests ÷ total requests

Where "successful" means:
  - HTTP 2xx response
  - Within an acceptable latency threshold
  - Correct business outcome (not a silent failure)
```

**A common trap**: Your uptime monitor says "99.9% uptime" because the health endpoint returned 200. But your salary payouts were silently failing for 40 minutes because of a bug in the payment logic. Health checks and availability are not the same thing.

**Availability in PromQL:**
```promql
# 30-day availability for payout API
1 - (
  sum(rate(http_server_requests_seconds_count{uri="/sme/payouts/initiate", status=~"5.."}[30d]))
  /
  sum(rate(http_server_requests_seconds_count{uri="/sme/payouts/initiate"}[30d]))
)
```

---

## 4. Percentiles — Why Averages Lie

This is the most important concept in observability. Read it carefully.

**Never use averages to measure latency.** Averages hide the most important information.

### The Problem With Averages

```
Scenario: 10 card host debit calls (latency in ms):
  80, 85, 82, 88, 84, 90, 83, 87, 200, 5000

Average: (80+85+82+88+84+90+83+87+200+5000) ÷ 10 = 588ms

The average says: "your service takes 588ms"
The truth is:      9 out of 10 calls finish in under 90ms
                   1 call out of 10 hangs for 5 seconds
                   The average represents nobody's actual experience
```

The average is dragged upward by the one slow call (5000ms). If the average looks fine (say 200ms), you'd never investigate — but 10% of users are waiting 5 seconds.

### Percentiles Tell the Truth

A **percentile** tells you: *"What is the maximum latency that X% of requests complete within?"*

```
For the same 10 calls: 80, 82, 83, 84, 85, 87, 88, 90, 200, 5000

p50  (50th percentile) = 85ms
  → Half of calls complete in 85ms or less
  → The "median" — the typical user experience

p75  (75th percentile) = 88ms
  → 75% of calls complete in 88ms or less

p90  (90th percentile) = 200ms
  → 90% of calls complete in 200ms or less

p95  (95th percentile) = 5000ms  (with only 10 samples)
  → 95% complete in this time or less

p99  (99th percentile) = 5000ms
  → 99% complete in this time or less
  → At scale (1000+ calls), p99 would separate from p95
```

### Visualizing Percentiles at Scale

With 1,000 card host calls:

```
Latency distribution (hypothetical):

  70-100ms  ████████████████████████████████  80% of calls (fast path)
 100-200ms  ████████                          8% (slight delay)
 200-500ms  ████                              4% (noticeable delay)
 500ms-1s   ██                                2% (slow)
   1s-2s    █                                 1% (very slow, p99 threshold)
     2s+    ░                                 0.1% (p99.9 threshold — tail latency)
   10s+     ·                                 0.01% (timeouts — these show up in errors)

Average:  ~180ms   (looks fine! hides the 5% who are suffering)
p50:      ~90ms    (typical user)
p95:      ~500ms   (5% of users wait this long)
p99:      ~2000ms  (1% — that's 10 users per 1000 requests)
p99.9:    ~10000ms (1 in 1000 — might time out, causing errors)
```

### Which Percentile to Care About?

| Percentile | When to Use | Why |
|------------|------------|-----|
| **p50** | Baseline performance | "What's the typical experience?" |
| **p95** | Degradation detection | "5% of users are affected — investigate" |
| **p99** | SLO alerting | "1 in 100 is slow — this is real user pain" |
| **p99.9** | High-volume services | At 10,000 req/s, p99.9 = 10 bad requests/sec |

**For salary processing**: Your p99 matters most. If you process 10,000 salary debits, p99 = 100 employees are experiencing that latency. p95 = 500 employees. These are real people's salaries.

### How Histograms Enable Percentiles

You can't calculate percentiles from counters. You need a **histogram** — which records how many observations fall into predefined "buckets":

```
Histogram buckets for cardhost.debit.duration:
  [0, 50ms)   : 120 observations
  [50, 100ms) : 650 observations
  [100, 200ms): 150 observations
  [200, 500ms): 50 observations
  [500ms, 1s) : 20 observations
  [1s, 2s)    : 8 observations
  [2s, +inf)  : 2 observations
  Total: 1000

From these buckets, Prometheus calculates:
  p50  = somewhere in [50, 100ms) bucket
  p95  = somewhere in [100, 200ms) bucket  
  p99  = somewhere in [500ms, 1s) bucket
```

In Micrometer:
```java
Timer.builder("bharat.sme.cardhost.debit.duration")
    .publishPercentileHistogram()   // stores the buckets above
    .publishPercentiles(0.5, 0.95, 0.99)  // pre-computes these client-side too
    .register(registry);
```

In PromQL (server-side calculation from buckets — more accurate):
```promql
# p99 latency of card host debit calls
histogram_quantile(0.99,
  rate(bharat_sme_cardhost_debit_duration_seconds_bucket[5m])
)
```

---

## 5. SLI, SLO, SLA

These three terms define the framework for making reliability commitments. They are always confused — here is the cleanest way to understand them:

```
SLI → what you measure
SLO → the target you set for that measurement (internal)
SLA → the commitment you make to customers (external, legal)
```

### SLI — Service Level Indicator

The actual metric you measure. Must be something you can calculate from real data.

```
Examples:
  - "The p99 latency of POST /sme/payouts/initiate"
  - "The percentage of salary debits that succeed"
  - "The availability of the payouts API over the last 30 days"
```

### SLO — Service Level Objective

Your internal target for an SLI. This is not a contract — it's your own engineering goal.

```
Examples:
  - "p99 latency of payouts API < 2000ms, measured over 30 days"
  - "Salary debit success rate > 99.5%"
  - "Payouts API availability > 99.9% per month"
```

### SLA — Service Level Agreement

A contractual commitment (legal document) made to a customer. If you breach it, there are business consequences (refunds, penalties, contract clauses).

```
Example:
  "Niyo guarantees 99.9% API availability for corporate salary processing.
   If monthly availability falls below 99.9%, the corporate client receives
   a 10% fee credit."
```

### Error Budget

The amount of unreliability your SLO "allows."

```
SLO: 99.9% availability per month

Error budget = 1 - 0.999 = 0.1% of all requests may fail
             = 43.2 minutes of downtime per month

If you've used up 30 minutes of your error budget by the 20th of the month,
you have 13 minutes left. Stop shipping risky features. Focus on reliability.
```

**Practical SLOs for the SME service:**

```yaml
# SLO 1: Payout API Latency
sli:  p99 of bharat.sme.cardhost.debit.duration
slo:  < 2000ms over rolling 30 days
why:  Salary day call volume → p99 is real user pain (100 employees per 10k run)

# SLO 2: Payout Success Rate
sli:  rate(salary.transfer.debit{status="success"}) / rate(salary.transfer.debit)
slo:  > 99.5%
why:  0.5% failure = 50 employees per 10k run — acceptable; 2% = 200 — not acceptable

# SLO 3: Kafka Processing Lag
sli:  max consumer lag across payouts topics
slo:  < 5,000 messages (growing lag = delayed payouts)
why:  Unbounded lag means employees receive salary late even if no error is thrown
```

---

## 6. The Three Pillars

Observability data comes in three distinct types. Each answers different questions. You need all three.

```
┌─────────────────────────────────────────────────────────────────────┐
│                        Three Pillars                                 │
├─────────────┬───────────────────────┬───────────────────────────────┤
│  METRICS    │  LOGS                 │  TRACES                        │
│             │                       │                                │
│  Numbers    │  Events               │  Request journeys              │
│  over time  │  (text, timestamped)  │  (spans across services)       │
├─────────────┼───────────────────────┼───────────────────────────────┤
│ "Are we     │ "What exactly went    │ "Where did this request        │
│  healthy?"  │  wrong for TX-12345?" │  spend its time?"              │
├─────────────┼───────────────────────┼───────────────────────────────┤
│ Dashboard   │ Grep / full-text      │ Flame graphs, waterfalls       │
│ Alerting    │ search                │                                │
│ Trends      │ Debugging             │ Bottleneck discovery           │
└─────────────┴───────────────────────┴───────────────────────────────┘
```

**When to use which:**

| Question | Pillar |
|----------|--------|
| "Are payouts working right now?" | Metrics (success rate counter) |
| "What's our p99 latency?" | Metrics (timer histogram) |
| "Why did TX-12345 fail?" | Logs (error message + context) |
| "What error did YBL return?" | Logs (response body logged) |
| "Where does the 5s latency come from?" | Traces (span breakdown) |
| "Is the card host the bottleneck?" | Traces (span duration) + Metrics (p99) |
| "How many employees affected?" | Metrics (counter) |
| "Who triggered the failed batch?" | Logs (userId in MDC) |

The three pillars are most powerful when **correlated**: a metric alert leads you to a trace, and the trace links to the specific log line that shows the error. This is what a modern observability stack enables.

---

## 7. Metrics — Every Type Explained

A **metric** is a numeric measurement captured over time. Unlike a log (one event) or a trace (one request), metrics are designed to be aggregated across millions of events.

### Anatomy of a Metric

```
Name:       bharat.sme.salary.transfer.debit
Type:       Counter
Value:      1 (each call increments by 1)
Timestamp:  2026-05-06T10:23:45Z
Tags:       status=success
            transfer_type=NEFT
            env=production
```

The **tags** (also called labels) are what make metrics powerful. They let you slice one metric into dimensions without creating separate metric names.

### 7.1 Counter

**What it is**: A value that only goes **up**. Resets to zero when the application restarts.

**Use for**: Counting events — requests, errors, messages consumed, transactions processed.

```java
// Every time a salary debit is attempted, increment this counter
meterRegistry.counter("bharat.sme.salary.transfer.debit",
    "status", "success",
    "transfer_type", txn.getTransferType().name()
).increment();
```

**What you can ask a counter:**
- "How many salary debits happened in the last hour?" → `increase(counter[1h])`
- "What's the rate of failures per second?" → `rate(counter{status="failed"}[5m])`
- "What fraction of debits succeeded?" → `rate(success[5m]) / rate(total[5m])`

**Important:** Counters reset on restart. Use `rate()` in PromQL, never raw counter values — raw values are meaningless across restarts.

### 7.2 Gauge

**What it is**: A value that can go **up and down**. Captures the current state of something.

**Use for**: Current state snapshots — queue depth, active connections, memory usage, thread count.

```java
// Track how many payouts are currently being processed
AtomicInteger activePayouts = new AtomicInteger(0);

Gauge.builder("bharat.sme.payouts.active", activePayouts, AtomicInteger::get)
    .description("Payouts currently in-flight")
    .register(meterRegistry);

// When a payout starts:
activePayouts.incrementAndGet();
// When it finishes (success or failure):
activePayouts.decrementAndGet();
```

**More gauge examples:**
```java
// Track Kafka consumer lag (provided by Spring Kafka auto-instrumentation)
// Auto-registered as: kafka.consumer.records.lag

// Track DB connection pool (provided by HikariCP auto-instrumentation)
// Auto-registered as: hikaricp.connections.active, hikaricp.connections.idle

// Track JVM heap (provided by Spring Boot Actuator)
// Auto-registered as: jvm.memory.used, jvm.memory.max
```

**Counter vs Gauge — the decision rule:**

> If the value can only go up → **Counter**
> If the value can go up and down → **Gauge**

| What You're Measuring | Type | Why |
|-----------------------|------|-----|
| Requests processed | Counter | Never decrements |
| Active HTTP connections | Gauge | Opens and closes |
| Errors thrown | Counter | Never decrements |
| Kafka consumer lag | Gauge | Can grow and shrink |
| JVM heap memory | Gauge | GC causes it to drop |
| Bytes transferred | Counter | Never decrements |
| Thread pool active count | Gauge | Tasks complete |

### 7.3 Timer (Histogram Under the Hood)

**What it is**: Measures the **duration** of events AND counts them. A Timer automatically gives you: count, total time, max, and histogram buckets for percentile calculation.

**Use for**: Latency measurement — API response time, DB query time, external call time.

```java
Timer.builder("bharat.sme.cardhost.debit.duration")
    .description("Time to complete a card host debit call")
    .tag("transfer_type", txn.getTransferType().name())
    .publishPercentileHistogram()          // enables p50/p95/p99 in Prometheus
    .publishPercentiles(0.5, 0.95, 0.99)   // also pre-compute client-side
    .minimumExpectedValue(Duration.ofMillis(10))
    .maximumExpectedValue(Duration.ofSeconds(30))
    .register(meterRegistry);
```

**What a single Timer registration gives you for free:**

| Auto-generated Metric | Meaning |
|-----------------------|---------|
| `..._seconds_count` | Number of calls (a counter) |
| `..._seconds_sum` | Total time across all calls |
| `..._seconds_max` | Worst single call duration (rolling window) |
| `..._seconds_bucket` | Histogram buckets for `histogram_quantile()` |

**Using a Timer to wrap code:**
```java
// Option 1: Wrap a lambda
DebitResponse response = debitTimer.record(() -> cardHostClient.debit(request));

// Option 2: Manual start/stop (for try/catch flows)
Timer.Sample sample = Timer.start(meterRegistry);
try {
    DebitResponse response = cardHostClient.debit(request);
    sample.stop(meterRegistry.timer("bharat.sme.cardhost.debit.duration", "status", "success"));
} catch (Exception e) {
    sample.stop(meterRegistry.timer("bharat.sme.cardhost.debit.duration", "status", "error"));
    throw e;
}
```

### 7.4 Histogram

A **histogram** is the underlying data structure inside a Timer. You can also use `DistributionSummary` for non-time values.

**What it is**: Counts observations in predefined buckets. Enables percentile queries.

**Use for**: Any value distribution — payload sizes, batch sizes, monetary amounts (for operational purposes, not financial ledger).

```java
DistributionSummary.builder("bharat.sme.payout.batch.size")
    .description("Number of transactions per payout batch file")
    .publishPercentiles(0.5, 0.95)
    .register(meterRegistry);

// Record a batch size
batchSizeSummary.record(employeeList.size());
```

**Histogram buckets explained visually:**

```
Observation: "how long did the debit call take?"

Buckets:   [≤10ms] [≤25ms] [≤50ms] [≤100ms] [≤250ms] [≤500ms] [≤1s] [≤2s] [+∞]
Counts:        0      0      80      600      280       30       8    2    0

From this, Prometheus can calculate:
  Total observations = 1000
  p50 ≈ 100ms  (600/1000 = 60% are ≤100ms, so p50 is somewhere around 100ms)
  p95 ≈ 250ms
  p99 ≈ 1s
```

### 7.5 Cardinality — The #1 Metrics Trap

**Cardinality** is the number of unique time series a metric produces:

```
unique time series = unique combinations of (metric_name × tag_value_1 × tag_value_2 × ...)

Example:
  metric: bharat.sme.salary.transfer.debit
  tag: status (3 values: success, failed, pending)
  tag: transfer_type (4 values: NEFT, IMPS, RTGS, UPI)

  Total time series = 1 × 3 × 4 = 12  ✅ Fine

Now add:
  tag: transaction_id (1,000,000 unique values)

  Total time series = 1 × 3 × 4 × 1,000,000 = 12,000,000  🔥 Destroys Prometheus
```

**Rule**: Never use unbounded values (user IDs, transaction IDs, timestamps, request bodies) as tag values. Put them in **logs** and **traces** instead.

```java
// WRONG — cardinality explosion
meterRegistry.counter("debit", "txn_id", txn.getId()).increment();

// RIGHT — bounded tags only
meterRegistry.counter("debit",
    "status", result.getStatus().name(),      // ~5 values
    "transfer_type", txn.getTransferType(),   // ~4 values
    "corporate_id", txn.getCorporateId()      // ~500 values — borderline, use cautiously
).increment();
```

---

## 8. Logs

A **log** is a discrete, timestamped text record of something that happened. Unlike metrics (aggregated numbers), logs capture individual events with full context.

### Structured vs Unstructured Logs

**Unstructured (avoid):**
```
2026-05-06 10:23:45 ERROR PayoutsWorker - Debit failed for TX-12345, code 51
```

This is a human-readable string. To search it, you need `grep`. You can't filter by `responseCode=51` specifically — it's just text.

**Structured (JSON — use this):**
```json
{
  "timestamp": "2026-05-06T10:23:45.123Z",
  "level": "ERROR",
  "logger": "c.n.s.w.PayoutsTransactionsWorker",
  "message": "Debit failed",
  "transactionId": "TX-12345",
  "corporateId": "CORP-789",
  "responseCode": "51",
  "reason": "Insufficient funds",
  "transferType": "NEFT",
  "traceId": "abc123def456",
  "spanId": "span789",
  "correlationId": "app-sme-uuid-here"
}
```

Now you can query: `responseCode=51 AND corporateId=CORP-789` — and find every affected transaction.

Your service already produces structured JSON logs via `logstash-logback-encoder`. The `logback-spring.xml` in both API and worker modules outputs this format.

### MDC — How Context Gets Into Logs

**MDC** (Mapped Diagnostic Context) is a thread-local key-value store. Whatever you put in MDC appears in every log line from that thread — you don't have to pass context through every method.

```java
// MDCFilter (already in your codebase) sets these on every request:
MDC.put("correlationId", correlationId);   // from x-correlation-id header
MDC.put("userId", userId);                 // from JWT token
MDC.put("deviceId", deviceId);             // from x-device-id header

// Then any log in any class down the call chain includes them automatically:
log.error("Debit failed for txnId={}", txn.getId());
// Output includes: "correlationId": "app-sme-abc123", "userId": "user-456"
// Without you having to pass correlationId to every method
```

**The missing piece in your current setup**: `traceId` and `spanId` from distributed tracing are not currently in your MDC. When you add Micrometer Tracing (see [Section 15](#15-spring-boot--java-integration-roadmap)), they will be added automatically — enabling you to jump from a log line to a full trace in one click.

---

## 9. Traces

A **trace** represents the complete journey of a single request through your system. It's composed of **spans**.

### What Is a Span?

A **span** is one unit of work within a trace:

```
Trace ID: abc123def456  (one salary payout request)

├─ Span: POST /sme/payouts/initiate           [0ms → 212ms]
│  ├─ Span: SmeAuthenticationFilter           [0ms → 2ms]
│  ├─ Span: MongoDB findById (corporate)      [3ms → 8ms]
│  ├─ Span: MongoDB save (transaction)        [9ms → 12ms]
│  ├─ Span: HTTP POST card-host/debit         [13ms → 193ms]  ← 180ms — bottleneck!
│  ├─ Span: MongoDB update (transaction)      [194ms → 202ms]
│  └─ Span: Kafka produce (success-topic)     [203ms → 210ms]
```

Each span has:
- A name (describes the operation)
- Start time and duration
- A parent span (forming a tree)
- Tags/attributes (metadata like `transaction.id`, `response.code`)
- Status (OK or ERROR)
- Events (timestamped annotations: "retried at t=150ms")

### Why Traces Are Irreplaceable

You can't find that 180ms card-host span from metrics alone. Metrics tell you "p99 is 180ms." The trace tells you *which part* of the 180ms is the card host vs MongoDB vs Kafka — for *this specific request*.

**Without tracing:**
```
Alert fires: p99 latency > 2s
You: grep logs for slow requests... find TX-12345 took 2.1s
You: which part was slow? look at all the log timestamps manually...
     "DB query at 10:23:45.100"
     "Card host returned at 10:23:47.200"
     "Response sent at 10:23:47.220"
     Calculation: card host took 2100ms. ok, card host is the problem.
     Time to find: 15 minutes
```

**With tracing:**
```
Alert fires: p99 latency > 2s
You: open Tempo/Jaeger, search traceId from alert
You: see flame graph instantly
     card-host span is red (2100ms), everything else is green
     Time to find: 30 seconds
```

### Trace Context Propagation

For traces to work across services, the `traceId` must travel with the request:

```
sme-service-apis ──── Kafka ────► sme-service-workers

APIs creates Span A with traceId=abc123
APIs publishes Kafka message with header: traceparent=abc123/spanA

Workers consumes message, reads traceparent header
Workers creates Span B as a CHILD of Span A, with traceId=abc123

Result: one trace showing the full journey from API → Kafka → Worker
```

The standard format for this header is **W3C Trace Context**: `traceparent: 00-{traceId}-{spanId}-{flags}`.

---

## 10. How the Three Pillars Connect

The full power of observability comes from *correlation* — being able to jump between metrics, logs, and traces for the same event.

```
ALERT fires in Grafana:
  bharat.sme.salary.transfer.debit{status="failed"} rate > 1%
  at 10:03:45

                    │
                    ▼ (click through from metric alert)

TRACES in Tempo:
  Show all failed "cardhost.debit" spans in that 1-minute window
  Filter: status=ERROR
  Find: traceId=abc123def456 — card host returned 504

                    │
                    ▼ (click from span to logs)

LOGS in Loki:
  Query: traceId="abc123def456"
  Find: full card host response body, correlationId, userId
  See: HTTP 504 Gateway Timeout from YBL upstream
  See: 47 other transactions with same pattern

                    │
                    ▼ (conclusion)

ROOT CAUSE:
  YBL gateway degraded at 10:03 UTC
  47 transactions affected
  Notify YBL, trigger retry mechanism
  Total time to root cause: 4 minutes
```

**The link between them is `traceId`**:
- The metric alert tells you *when* and *what*
- The trace tells you *where* in the call chain
- The log (filtered by traceId) tells you the exact error message and context

Currently your service has `correlationId` flowing through logs. When you add Micrometer Tracing (see Section 15), `traceId` will be in logs automatically, completing this chain.

---

## 11. Prometheus

**Prometheus** is an open-source monitoring system and time-series database. It is the de facto standard for metrics in Kubernetes environments.

### How Prometheus Works (Pull Model)

Unlike Datadog (which you push metrics *to*), Prometheus *pulls* metrics *from* your application:

```
Your App                            Prometheus Server
┌──────────────────────┐            ┌─────────────────┐
│  Spring Boot Actuator │            │                 │
│  /actuator/prometheus │◄─ scrape ──│  every 15-30s   │
│                       │           │                 │
│  Output (text format):│           │  Stores in      │
│  # HELP bharat_sme_.. │           │  time-series DB │
│  bharat_sme_salary_.. │           │                 │
│    _transfer_debit_   │           │  Query with     │
│    total{status=".."} │           │  PromQL         │
│    1247               │           │                 │
└──────────────────────┘            └─────────────────┘
```

Your service already exposes `/actuator/prometheus` (on port 8081). Groundcover scrapes it. If you run Prometheus yourself, it would do the same.

### The Prometheus Text Format

When Prometheus scrapes your app, it gets plain text like this:

```
# HELP bharat_sme_salary_transfer_debit_total Number of salary debit transactions
# TYPE bharat_sme_salary_transfer_debit_total counter
bharat_sme_salary_transfer_debit_total{status="success",transfer_type="NEFT"} 8342.0
bharat_sme_salary_transfer_debit_total{status="success",transfer_type="IMPS"} 1205.0
bharat_sme_salary_transfer_debit_total{status="failed",transfer_type="NEFT"}  47.0

# HELP bharat_sme_cardhost_debit_duration_seconds Time for card host debit
# TYPE bharat_sme_cardhost_debit_duration_seconds histogram
bharat_sme_cardhost_debit_duration_seconds_bucket{le="0.05"} 120
bharat_sme_cardhost_debit_duration_seconds_bucket{le="0.1"}  750
bharat_sme_cardhost_debit_duration_seconds_bucket{le="0.25"} 900
bharat_sme_cardhost_debit_duration_seconds_bucket{le="0.5"}  950
bharat_sme_cardhost_debit_duration_seconds_bucket{le="1.0"}  980
bharat_sme_cardhost_debit_duration_seconds_bucket{le="+Inf"} 1000
bharat_sme_cardhost_debit_duration_seconds_sum 98.5
bharat_sme_cardhost_debit_duration_seconds_count 1000
```

Note: Micrometer automatically converts dot-notation (`bharat.sme.salary.transfer.debit`) to underscore-notation (`bharat_sme_salary_transfer_debit`) for Prometheus.

### Basic PromQL

**PromQL** (Prometheus Query Language) is how you query this data:

```promql
# 1. Current counter value (usually not useful — resets on restart)
bharat_sme_salary_transfer_debit_total

# 2. Rate (requests per second, averaged over 5 minutes) — the right way
rate(bharat_sme_salary_transfer_debit_total[5m])

# 3. Filter by tag
rate(bharat_sme_salary_transfer_debit_total{status="failed"}[5m])

# 4. Error rate as a percentage
100 * (
  rate(bharat_sme_salary_transfer_debit_total{status="failed"}[5m])
  /
  rate(bharat_sme_salary_transfer_debit_total[5m])
)

# 5. p99 latency from histogram
histogram_quantile(0.99,
  rate(bharat_sme_cardhost_debit_duration_seconds_bucket[5m])
)

# 6. Sum across all tag combinations
sum(rate(bharat_sme_salary_transfer_debit_total[5m]))

# 7. Sum grouped by a tag
sum by (status) (rate(bharat_sme_salary_transfer_debit_total[5m]))

# 8. Alert condition: error rate > 5%
rate(bharat_sme_salary_transfer_debit_total{status="failed"}[5m])
  /
rate(bharat_sme_salary_transfer_debit_total[5m])
> 0.05
```

---

## 12. OpenTelemetry

**OpenTelemetry (OTel)** is a CNCF (Cloud Native Computing Foundation) project that provides a single, vendor-neutral set of APIs, SDKs, and tools for collecting metrics, traces, and logs. It is the industry standard for vendor-neutral observability.

### Why OpenTelemetry Exists

Before OTel, every vendor had their own agent and SDK:

```
Without OTel (what you have now):
  Your app → Datadog StatsD agent → Datadog
  Your app → Datadog Java agent → Datadog traces

Problems:
  - Change from Datadog to Grafana? Rewrite all instrumentation code
  - Test locally without Datadog? Either install Datadog or comment out metrics
  - Use Prometheus for metrics and Jaeger for traces? Two separate APIs
```

```
With OTel (the goal):
  Your app → OTel SDK → OTel Collector → any backend (Prometheus, Grafana, Datadog, ...)

Benefits:
  - Switch backends by changing Collector config, not app code
  - Run locally with a simple Collector docker-compose — no vendor account needed
  - Same API for metrics, traces, and logs
  - Industry standard — supported by every cloud provider and observability vendor
```

### The Three Parts of OpenTelemetry

```
┌─────────────────────────────────────────────────────────────────────┐
│ 1. OTel API                                                          │
│    The interfaces your code calls. Stable. You write code against   │
│    this. Never changes between implementations.                      │
├─────────────────────────────────────────────────────────────────────┤
│ 2. OTel SDK                                                          │
│    The implementation. Pluggable exporters, samplers, processors.   │
│    You configure this but don't write code against it directly.     │
├─────────────────────────────────────────────────────────────────────┤
│ 3. OTel Collector                                                    │
│    A separate process (deployed alongside your app) that receives   │
│    telemetry via OTLP, processes it, and exports to backends.       │
│    Your app's single telemetry destination — the hub.               │
└─────────────────────────────────────────────────────────────────────┘
```

### OTLP — The Wire Protocol

**OTLP** (OpenTelemetry Protocol) is the standard format for sending telemetry data. Think of it like JSON for metrics/traces/logs — a universal language.

```
Your App ──── OTLP/HTTP (port 4318) ────► OTel Collector
         OR── OTLP/gRPC (port 4317) ────►
```

Once your data is in OTLP format, the Collector can send it to:
- Prometheus (for metrics)
- Jaeger or Tempo (for traces)
- Loki or OpenSearch (for logs)
- Datadog (if you still want it as one of many backends)
- Any combination — simultaneously

### OTel Java Agent — Zero-Code Instrumentation

The OTel Java Agent is a JAR you attach to the JVM at startup. It instruments your code automatically using bytecode manipulation — no code changes required:

```dockerfile
ENV JAVA_TOOL_OPTIONS="-javaagent:/opt/opentelemetry-javaagent.jar"
ENV OTEL_SERVICE_NAME=sme-service-workers
ENV OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4318
ENV OTEL_RESOURCE_ATTRIBUTES=deployment.environment=production
```

The agent auto-instruments:
- Spring MVC HTTP endpoints
- Kafka producers and consumers (including trace context propagation across messages)
- MongoDB driver (every query becomes a span)
- OkHttp / Retrofit (every external call becomes a span)
- `@Scheduled` methods
- 100+ other libraries

**In your service, the agent would automatically create spans for every Retrofit call to YBL, every MongoDB query, and every Kafka message consume/produce — with zero code changes.**

### Micrometer + OTel — How They Work Together in Spring Boot

You don't have to choose between Micrometer and OTel. Spring Boot's recommended approach uses both:

```
Your code writes:  meterRegistry.counter("my.metric").increment()
                                        ↓
                   Micrometer Core (the abstraction layer)
                      ↓                          ↓
         micrometer-registry-prometheus    micrometer-registry-otlp
                      ↓                          ↓
         /actuator/prometheus             OTel Collector (port 4318)
         (Groundcover scrapes this)       (Grafana/Jaeger receives this)

Your code writes:  Observation.create("my.operation", registry).observe(...)
                                        ↓
                   Micrometer Tracing (the tracing abstraction)
                              ↓
                   micrometer-tracing-bridge-otel
                              ↓
                   OTel SDK → OTLP → OTel Collector → Tempo/Jaeger
```

**The rule of thumb:**
- Use **Micrometer API** for writing custom metrics (it's simpler and Spring-native)
- Use **OTLP exporter** (via `micrometer-registry-otlp`) to ship metrics vendor-neutrally
- Use **Micrometer Tracing + OTel bridge** for distributed tracing
- Optionally add **OTel Java Agent** for zero-code auto-instrumentation

---

## 13. The Open Source Stack

Here is the complete vendor-neutral, open-source observability stack. Each tool is replaceable — the choice of backend is decoupled from your application code via the OTel Collector.

```
┌─────────────────────────────────────────────────────────────────────┐
│                         Your Application                             │
│    Spring Boot + Micrometer + Micrometer Tracing + OTel Bridge      │
└──────────────────────┬──────────────────────────────────────────────┘
                       │ OTLP (metrics + traces + logs)
                       ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    OpenTelemetry Collector                            │
│  receivers:   otlp (grpc:4317, http:4318)                           │
│               prometheus (scrape /actuator/prometheus)              │
│  processors:  batch, memory_limiter, resource                       │
│  exporters:   prometheus → Prometheus/Mimir                         │
│               otlp → Tempo (traces)                                 │
│               loki → Loki (logs)                                    │
└──────────┬─────────────────────┬────────────────────┬──────────────┘
           │                     │                    │
           ▼                     ▼                    ▼
┌──────────────────┐  ┌──────────────────┐  ┌──────────────────────┐
│    Prometheus    │  │      Tempo        │  │       Loki           │
│  (metrics store) │  │  (traces store)   │  │   (logs store)       │
└────────┬─────────┘  └────────┬──────────┘  └──────────┬───────────┘
         │                     │                         │
         └─────────────────────┴─────────────────────────┘
                               │ all three data sources
                               ▼
                    ┌──────────────────────┐
                    │       Grafana        │
                    │  (visualization,     │
                    │   dashboards,        │
                    │   alerting,          │
                    │   correlation)       │
                    └──────────────────────┘
```

### Tool Glossary

| Tool | Role | What It Replaces |
|------|------|-----------------|
| **Prometheus** | Pull-based metrics store + alerting rules | Datadog metrics |
| **Grafana** | Visualization, dashboards, alerting UI | Datadog dashboards |
| **Tempo** | Distributed trace storage and querying | Datadog APM |
| **Loki** | Log aggregation and querying (like Prometheus for logs) | Datadog logs |
| **OTel Collector** | Telemetry hub — receives, processes, routes | Datadog agent |
| **Mimir** | Scalable long-term Prometheus metrics storage | Datadog metrics (at scale) |

**The Grafana LGTM stack** (Loki + Grafana + Tempo + Mimir) is the most popular open-source combination. Grafana Cloud offers it as a managed service if you don't want to self-host.

### Local Development Docker Compose

```yaml
# docker-compose.observability.yml
# Run: docker-compose -f docker-compose.observability.yml up -d

version: "3.8"
services:
  otel-collector:
    image: otel/opentelemetry-collector-contrib:latest
    ports:
      - "4317:4317"   # OTLP gRPC
      - "4318:4318"   # OTLP HTTP
      - "8889:8889"   # Prometheus scrape (collector's own metrics)
    volumes:
      - ./otel-collector-config.yml:/etc/otel-collector-config.yml
    command: ["--config=/etc/otel-collector-config.yml"]

  prometheus:
    image: prom/prometheus:latest
    ports:
      - "9090:9090"
    volumes:
      - ./prometheus.yml:/etc/prometheus/prometheus.yml

  tempo:
    image: grafana/tempo:latest
    ports:
      - "3200:3200"   # Tempo HTTP API
      - "4317"        # OTLP gRPC (internal)
    volumes:
      - ./tempo-config.yml:/etc/tempo-config.yml
    command: ["-config.file=/etc/tempo-config.yml"]

  loki:
    image: grafana/loki:latest
    ports:
      - "3100:3100"

  grafana:
    image: grafana/grafana:latest
    ports:
      - "3000:3000"
    environment:
      - GF_AUTH_ANONYMOUS_ENABLED=true
      - GF_AUTH_ANONYMOUS_ORG_ROLE=Admin
    volumes:
      - ./grafana-datasources.yml:/etc/grafana/provisioning/datasources/datasources.yml
```

With this running locally, your Spring Boot app can send telemetry to `http://localhost:4318` and see everything in Grafana at `http://localhost:3000` — no Datadog account needed.

---

## 14. Vendor Lock-In

### Why You're Currently Locked In

Your service currently uses:

```xml
<!-- StatsD push to Datadog-flavored agent (Groundcover) -->
<dependency>micrometer-registry-statsd</dependency>
  → hardcoded: flavor=datadog, host=groundcover-vector:8125

<!-- Legacy OpenTracing API (Datadog-era tracing API, now deprecated) -->
<dependency>io.opentracing:opentracing-util:0.33.0</dependency>
```

**The lock-in problems:**
1. `micrometer-registry-statsd` with `flavor=datadog` uses Datadog's StatsD extension (DogStatsD). You can't easily redirect this to a non-Datadog backend.
2. `opentracing-util` is the old (pre-OTel) tracing API, now superseded by OpenTelemetry. If a tracer is injected (e.g., Datadog Java Agent), the calls in `MDCFilter` to `GlobalTracer.get().activeSpan()` are Datadog-specific.
3. `DatadogMetricsConstants` and `DDMetricRegister` bake Datadog naming conventions into your service name.

### The Migration Path (Gradual, No Big Bang)

**Phase 1 — Add OTel alongside existing setup (zero disruption)**
```yaml
# Add OTLP export — both StatsD and OTLP run simultaneously
management:
  metrics:
    export:
      statsd:
        enabled: true   # keep for now
      otlp:
        enabled: true   # add this
        url: http://otel-collector:4318/v1/metrics
      prometheus:
        enabled: true   # already working
```

**Phase 2 — Add Micrometer Tracing (proper distributed tracing)**
```xml
<!-- Replace opentracing-util with this -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-otel</artifactId>
</dependency>
<dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-exporter-otlp</artifactId>
</dependency>
```
Now `traceId` and `spanId` are in your MDC automatically. The `MDCFilter` code touching `GlobalTracer` can be removed.

**Phase 3 — Remove StatsD**
Once OTLP is confirmed working, remove `micrometer-registry-statsd` and the `STATSD_HOST` environment variable.

**Phase 4 — Rename `DatadogMetricsConstants` → `MetricsConstants`**
Cosmetic but important for cognitive load. Metric names themselves (`bharat.sme.*`) are already vendor-neutral.

**The key insight**: Your application's metric *names* and *logic* are already vendor-neutral. The lock-in is only in the transport layer (StatsD → Datadog) and the tracing API (OpenTracing → OTel). Both are fixable without rewriting business logic.

---

## 15. Spring Boot & Java Integration Roadmap

### Spring Boot 3 (Now)

```xml
<!-- These four dependencies give you full observability -->

<!-- 1. Metrics API + auto-instrumentation (JVM, HTTP, Kafka, MongoDB) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>

<!-- 2. Prometheus endpoint (Groundcover scrapes this — already working) -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>

<!-- 3. OTLP push export (vendor-neutral — send to any backend) -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-otlp</artifactId>
</dependency>

<!-- 4. Distributed tracing via OTel -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-otel</artifactId>
</dependency>
<dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-exporter-otlp</artifactId>
</dependency>
```

**The Observation API — Spring Boot 3's biggest observability addition:**

```java
// ONE instrumentation call produces BOTH a metric AND a trace span
return Observation.createNotStarted("cardhost.debit", observationRegistry)
    .lowCardinalityKeyValue("transfer.type", txn.getTransferType().name())  // → metric tag
    .highCardinalityKeyValue("transaction.id", txn.getId())                 // → trace attribute only
    .observe(() -> cardHostClient.debit(request));
```

`lowCardinalityKeyValue` → goes into the Timer metric (must be bounded values)
`highCardinalityKeyValue` → goes into the trace span only (unbounded values like IDs are safe here)

This is the recommended way to instrument code in Spring Boot 3+. It replaces manually writing both a `Timer.record()` and a `Span.start()`.

### Java 21 (Next)

**Virtual threads** are the biggest change for observability:

```yaml
# Enable virtual threads in Spring Boot 3.2+
spring:
  threads:
    virtual:
      enabled: true
```

With virtual threads, thread pool saturation metrics become less important (virtual threads are cheap, you can have millions). Instead, focus on:
- Request concurrency (how many requests are in-flight)
- I/O latency (network calls, DB calls — where virtual threads spend their time)
- Kafka consumer throughput (workers can process far more messages in parallel)

**ThreadLocal context propagation (MDC) still works with virtual threads in Java 21**, but it's inherited at the point the virtual thread is created and not shared back. For Kafka consumers and async processing, use `ScopedValues` (Java 21 preview, stable in 25) or ensure MDC is set at the start of each virtual thread.

### Java 25 (Future)

**Stable Scoped Values** replace ThreadLocal for context propagation:

```java
// Java 25: ScopedValue — cleaner than MDC for virtual threads
private static final ScopedValue<String> TRACE_ID = ScopedValue.newInstance();
private static final ScopedValue<String> CORRELATION_ID = ScopedValue.newInstance();

public void processKafkaMessage(ConsumerRecord<String, PayoutsTransaction> record) {
    String traceId = extractTraceId(record.headers());
    String correlationId = extractCorrelationId(record.headers());

    ScopedValue.where(TRACE_ID, traceId)
               .where(CORRELATION_ID, correlationId)
               .run(() -> {
                   // All code in this scope — including forked virtual threads —
                   // can read TRACE_ID.get() and CORRELATION_ID.get()
                   processTransaction(record.value());
               });
}
```

This is the future-proof way to propagate observability context through virtual threads.

### Spring Boot 4 (Future)

Expected changes for observability:
- **Java 21 minimum** — virtual threads become the default for web and Kafka
- **Micrometer 2.x** — API largely backward-compatible for basic usage
- **Structured logging built-in** — `logging.structured.format=ecs` works without logstash-logback-encoder
- **OTel first-class** — the `micrometer-tracing-bridge-otel` dependency may be folded into Spring Boot itself
- **Jakarta EE 11** — no `javax.*` anywhere, only `jakarta.*`

If you follow this guide's approach (Micrometer API for metrics, OTLP for export, Micrometer Tracing for traces), your migration to Spring Boot 4 will be largely configuration changes, not code rewrites.

---

## 16. Glossary

| Term | Definition |
|------|-----------|
| **Availability** | Percentage of time a service is correctly handling requests. 99.9% = 8.76 hours downtime/year. |
| **Cardinality** | The number of unique time series a metric produces (metric name × unique tag value combinations). High cardinality can overwhelm Prometheus. |
| **Counter** | A metric that only increases (monotonically). Resets on restart. Use `rate()` in PromQL, not raw values. |
| **DistributionSummary** | A histogram for non-time values (sizes, amounts, batch sizes). Like a Timer without the time unit. |
| **Error Budget** | The allowed "budget" of failures under an SLO. SLO 99.9% = 0.1% error budget. Once spent, stop shipping features, fix reliability. |
| **Error Rate** | Percentage of requests that result in an error. `failed / total × 100`. |
| **Exporter** | A component that *pushes* telemetry from your app to a backend (OTLP, StatsD, Datadog). |
| **Four Golden Signals** | Latency, Throughput (Traffic), Error Rate, Saturation — the minimum set of metrics for service health (from Google SRE book). |
| **Gauge** | A metric that can go up and down. Represents current state (queue depth, memory, active connections). |
| **Grafana** | Open-source dashboard and alerting platform. Queries Prometheus, Loki, Tempo. |
| **Histogram** | Counts observations in predefined time/value buckets. Enables percentile (p50/p95/p99) calculation. Required for latency percentiles. |
| **Jaeger** | Open-source distributed tracing backend. Receives and stores traces, provides query UI. Alternative to Tempo. |
| **Latency** | Time to handle one request. Measured in milliseconds. Should be tracked as p50/p95/p99, not average. |
| **Loki** | Open-source log aggregation by Grafana. Like Prometheus but for logs. Uses LogQL query language. |
| **Long Task Timer** | A Timer that shows currently *in-progress* tasks and how long they've been running. Use for batch jobs. |
| **MDC** | Mapped Diagnostic Context. Thread-local key-value store. Keys placed in MDC appear in all logs from that thread without explicit passing. |
| **Meter** | Any Micrometer instrument: Counter, Gauge, Timer, etc. |
| **MeterRegistry** | The Micrometer central component. All meters are registered here. Spring Boot auto-configures a `CompositeMeterRegistry`. |
| **Micrometer** | Java metrics facade — like SLF4J but for metrics. Write code against one API, export to any backend (Prometheus, OTLP, StatsD). |
| **Micrometer Tracing** | Spring Boot 3's tracing abstraction layer. Bridges to OTel or Brave. Automatically puts `traceId`/`spanId` in MDC. |
| **Mimir** | Scalable long-term storage for Prometheus metrics (by Grafana). |
| **Observation API** | Spring Boot 3+ unified API that produces both a Timer metric AND a trace span from one instrumentation call. |
| **OpenTelemetry (OTel)** | CNCF standard for vendor-neutral metrics + traces + logs. One API, any backend. The industry direction. |
| **OpenTracing** | The *old* (pre-OTel) vendor-neutral tracing API. Deprecated. Replaced by OpenTelemetry. Your codebase uses this — it should be migrated. |
| **OTLP** | OpenTelemetry Protocol. The standard wire format for sending telemetry (HTTP/4318 or gRPC/4317). |
| **OTel Collector** | A standalone process that receives telemetry (OTLP, Prometheus scrape, etc.), processes it, and exports to multiple backends. The telemetry hub. |
| **OTel Java Agent** | A JAR attached to the JVM that auto-instruments HTTP, Kafka, MongoDB, etc. Zero code changes required. |
| **p50** | 50th percentile (median). Half of requests are faster than this. The typical user experience. |
| **p95** | 95th percentile. 95% of requests are faster. 5% experience this or worse — first sign of degradation. |
| **p99** | 99th percentile. 99% of requests are faster. 1 in 100 requests experiences this — real user pain at volume. |
| **p99.9** | 99.9th percentile. 1 in 1000 requests. Called "tail latency." Matters at high throughput (10k+ req/s). |
| **Percentile** | A statistical measure: the value below which X% of observations fall. p99 = 99% of observations are below this value. |
| **PromQL** | Prometheus Query Language. Used to query and alert on time-series metrics. |
| **Prometheus** | Open-source pull-based metrics system. Scrapes `/actuator/prometheus` periodically. Stores time-series data. |
| **Registry** | In Micrometer: a backend-specific implementation (PrometheusRegistry, OtlpRegistry, etc.). Routes your metrics to the right destination. |
| **RPS / TPS** | Requests Per Second / Transactions Per Second. The unit of throughput. |
| **Sampling** | Only recording a fraction of traces (e.g., 10%) to reduce storage cost. At 99.9% availability, 10% sampling still captures ~1 trace per 1000 requests. |
| **Saturation** | How close a resource (CPU, memory, thread pool, Kafka consumer) is to its limit. A leading indicator — rising saturation predicts future errors. |
| **SLA** | Service Level Agreement. A contractual/legal commitment to a customer about service reliability. Breach = business consequence. |
| **SLI** | Service Level Indicator. The actual metric you measure (e.g., "p99 latency of POST /payouts"). |
| **SLO** | Service Level Objective. Your internal reliability target for an SLI (e.g., "p99 < 2s"). Not a contract, but violation means reducing error budget. |
| **Span** | One unit of work within a distributed trace (e.g., one HTTP call, one DB query). Has a name, start time, duration, tags, and status. |
| **Structured Logging** | Logs emitted as JSON with consistent fields. Enables filtering by `responseCode=51` instead of text search. |
| **Tempo** | Open-source distributed trace storage by Grafana. Stores and queries traces. Integrates with Grafana dashboards. |
| **Throughput** | Operations handled per unit of time. RPS or TPS. Tells you how much load the system is under. |
| **Time Series** | A sequence of numeric values over time, uniquely identified by metric name + tag combination. |
| **Timer** | Measures duration AND count. Internally stores a histogram. Gives count, sum, max, and histogram buckets for percentile queries. |
| **Trace** | The complete journey of one request across all services, composed of spans in a tree structure. |
| **traceId** | A unique ID shared by all spans within a single trace. Put in logs to correlate log lines with the full trace. |
| **Virtual Threads** | Java 21+ lightweight threads (Project Loom). Cheap enough to create per-request. Changes how thread saturation is measured. |
| **W3C Trace Context** | The standard HTTP header format for propagating trace context: `traceparent: 00-{traceId}-{spanId}-{flags}`. |

---

*For implementation code, configuration examples, and step-by-step migration instructions specific to the SME service, see [`observability-guide.md`](./observability-guide.md).*
