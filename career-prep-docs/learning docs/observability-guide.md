# Observability Guide for Java/Spring Boot Engineers

## Table of Contents

1. [What is Observability?](#1-what-is-observability)
2. [The Three Pillars of Observability](#2-the-three-pillars-of-observability)
3. [Metrics Deep Dive](#3-metrics-deep-dive)
4. [Metric Types Explained](#4-metric-types-explained)
5. [Custom Metrics vs Auto-Instrumented Metrics](#5-custom-metrics-vs-auto-instrumented-metrics)
6. [Micrometer — The Java Metrics Facade](#6-micrometer--the-java-metrics-facade)
7. [OpenTelemetry — The Vendor-Neutral Standard](#7-opentelemetry--the-vendor-neutral-standard)
8. [Micrometer vs OpenTelemetry — When to Use What](#8-micrometer-vs-opentelemetry--when-to-use-what)
9. [Spring Boot Actuator & Auto-Configuration](#9-spring-boot-actuator--auto-configuration)
10. [Metric Registries & Exporters](#10-metric-registries--exporters)
11. [Distributed Tracing](#11-distributed-tracing)
12. [Structured Logging](#12-structured-logging)
13. [Practical Implementation in Your SME Service](#13-practical-implementation-in-your-sme-service)
14. [Observability Backends & Collectors](#14-observability-backends--collectors)
15. [Best Practices & Anti-Patterns](#15-best-practices--anti-patterns)
16. [Spring Boot 4 & Java 21/25 Considerations](#16-spring-boot-4--java-2125-considerations)
17. [Glossary](#17-glossary)

---

## 1. What is Observability?

Observability is the ability to understand the internal state of a system by examining its external outputs.

Think of it like this: **monitoring** tells you *when* something is broken. **Observability** tells you *why* it's broken, even for failures you didn't anticipate.

### Why Does This Matter for Your SME Service?

Your service processes salary payouts, insurance debits, inward remittances, and wallet transfers. When a salary payout fails at 2 AM:

- **Without observability**: You get a PagerDuty alert saying "payouts failing." You SSH into pods, grep logs, guess at the cause.
- **With observability**: You see the counter `bharat.sme.salary.transfer.debit.failed` spiked, trace the specific request through Kafka → Worker → Card Host API, see the latency histogram shows the card host is timing out at p99, and the logs show the exact error response code.

### The Mental Model

```
┌─────────────────────────────────────────────────────────────┐
│                    YOUR APPLICATION                          │
│                                                             │
│  ┌─────────┐    ┌─────────┐    ┌─────────────────────┐     │
│  │ Metrics │    │  Logs   │    │  Traces (Spans)     │     │
│  │         │    │         │    │                     │     │
│  │ Counter │    │ ERROR   │    │ Request A           │     │
│  │ Gauge   │    │ WARN    │    │  ├─ Kafka consume   │     │
│  │ Timer   │    │ INFO    │    │  ├─ DB query        │     │
│  │ Histo   │    │ DEBUG   │    │  ├─ HTTP call       │     │
│  └────┬────┘    └────┬────┘    │  └─ Kafka produce   │     │
│       │              │         └──────────┬──────────┘     │
└───────┼──────────────┼────────────────────┼─────────────────┘
        │              │                    │
        ▼              ▼                    ▼
┌──────────────────────────────────────────────────────────────┐
│              OTEL Collector / Groundcover Agent               │
└──────────────────────────────────────────────────────────────┘
        │              │                    │
        ▼              ▼                    ▼
┌────────────┐  ┌────────────┐  ┌────────────────────┐
│ Prometheus │  │   Loki /   │  │  Tempo / Jaeger    │
│ Grafana    │  │ OpenSearch  │  │                    │
└────────────┘  └────────────┘  └────────────────────┘
```

---

## 2. The Three Pillars of Observability

### Pillar 1: Metrics

**What**: Numeric measurements aggregated over time.
**When to use**: Dashboards, alerting, capacity planning, SLOs.
**Example**: "How many salary payouts succeeded in the last hour?"

```
bharat.sme.salary.transfer.debit.success = 1,247
bharat.sme.salary.transfer.debit.failed = 3
```

### Pillar 2: Logs

**What**: Discrete, timestamped text records of events.
**When to use**: Debugging specific failures, audit trails, understanding *what happened*.
**Example**: "Why did payout TX-12345 fail?"

```
2026-05-06T10:23:45.123Z ERROR [worker-payouts-3] c.n.s.w.PayoutsTransactionsWorker - 
  Debit failed for txnId=TX-12345, corporateId=CORP-789, responseCode=51, 
  reason="Insufficient funds", traceId=abc123def456
```

### Pillar 3: Traces (Distributed Tracing)

**What**: The journey of a single request across multiple services/components.
**When to use**: Understanding latency, finding bottlenecks, debugging cross-service failures.
**Example**: "Where did the 5-second delay come from in this payout?"

```
Trace: abc123def456
├─ [200ms] KafkaConsumer.receive (PayoutsTransactionsWorker)
│  ├─ [5ms]   MongoDB.findById (payouts_transactions)
│  ├─ [3200ms] HTTP POST card-host.internal/debit  ← BOTTLENECK
│  ├─ [15ms]  MongoDB.save (transaction_history)
│  └─ [8ms]   KafkaProducer.send (salary-transfer-success-topic)
```

### How They Work Together

| Question | Pillar |
|----------|--------|
| "Are payouts healthy right now?" | Metrics (success rate counter) |
| "Why did this specific payout fail?" | Logs (error message + context) |
| "Where is the latency coming from?" | Traces (span breakdown) |
| "Is the card host degraded?" | Metrics (p99 latency) + Traces (span duration) |

---

## 3. Metrics Deep Dive

### What is a Metric?

A metric is a **numeric measurement** captured at a point in time, typically aggregated into time series. Unlike logs (which are individual events), metrics are designed for aggregation and statistical analysis.

### Anatomy of a Metric

```
┌─────────────────────────────────────────────────────────────┐
│  Metric Name: bharat.sme.salary.transfer.debit.duration     │
│  Type: Timer (histogram)                                     │
│  Value: 245ms                                                │
│  Timestamp: 2026-05-06T10:23:45Z                            │
│  Tags/Labels:                                                │
│    env=production                                            │
│    corporate_id=CORP-789                                     │
│    status=success                                            │
│    transfer_type=NEFT                                        │
└─────────────────────────────────────────────────────────────┘
```

Components:
- **Name**: Hierarchical identifier (use dots for namespacing)
- **Type**: Counter, Gauge, Timer, Distribution Summary, etc.
- **Value**: The numeric measurement
- **Timestamp**: When it was recorded
- **Tags/Labels**: Key-value pairs for dimensional filtering (this is what makes metrics powerful)

### Tags/Labels — The Secret Weapon

Tags let you slice and dice metrics without creating separate metric names:

```java
// BAD: Separate metric per status (cardinality explosion waiting to happen)
meterRegistry.counter("salary.transfer.debit.success").increment();
meterRegistry.counter("salary.transfer.debit.failed").increment();
meterRegistry.counter("salary.transfer.debit.pending").increment();

// GOOD: Single metric with a "status" tag
meterRegistry.counter("salary.transfer.debit", "status", "success").increment();
meterRegistry.counter("salary.transfer.debit", "status", "failed").increment();
meterRegistry.counter("salary.transfer.debit", "status", "pending").increment();
```

With the tag approach, you can:
- Query total debits: `sum(salary_transfer_debit_total)`
- Query failures only: `salary_transfer_debit_total{status="failed"}`
- Calculate success rate: `salary_transfer_debit_total{status="success"} / salary_transfer_debit_total`

### Cardinality — The #1 Metrics Pitfall

**Cardinality** = the number of unique time series a metric produces.

```
metric name × tag1 values × tag2 values × ... = total time series
```

**Dangerous example:**
```java
// DON'T DO THIS — userId has millions of values
meterRegistry.counter("api.request", "userId", userId).increment();
// If you have 1M users = 1M time series for ONE metric
```

**Safe example:**
```java
// DO THIS — corporate_id has bounded values (hundreds, not millions)
meterRegistry.counter("api.request", "corporate_id", corporateId, "endpoint", endpoint).increment();
// 500 corporates × 20 endpoints = 10,000 time series (manageable)
```

**Rule of thumb**: Never use unbounded values (user IDs, transaction IDs, timestamps) as tag values. Use them in logs/traces instead.

---

## 4. Metric Types Explained

### 4.1 Counter

**What**: A value that only goes up (monotonically increasing). Resets to zero on application restart.
**Use for**: Counting events — requests, errors, messages processed, bytes transferred.

```java
import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;

@Service
public class PayoutsMetrics {

    private final Counter debitSuccessCounter;
    private final Counter debitFailedCounter;

    public PayoutsMetrics(MeterRegistry registry) {
        // Option 1: Pre-register counters (preferred for known metrics)
        this.debitSuccessCounter = Counter.builder("bharat.sme.salary.transfer.debit")
            .description("Number of salary debit transactions")
            .tag("status", "success")
            .register(registry);

        this.debitFailedCounter = Counter.builder("bharat.sme.salary.transfer.debit")
            .description("Number of salary debit transactions")
            .tag("status", "failed")
            .register(registry);
    }

    public void recordDebitSuccess() {
        debitSuccessCounter.increment();
    }

    public void recordDebitFailed() {
        debitFailedCounter.increment();
    }
}
```

```java
// Option 2: Inline (simpler, good for dynamic tags)
meterRegistry.counter("bharat.sme.salary.transfer.debit", 
    "status", "success",
    "transfer_type", transferType.name()
).increment();
```

**What you can query:**
- Rate: "How many debits per second?" → `rate(salary_transfer_debit_total[5m])`
- Total: "How many debits today?" → `increase(salary_transfer_debit_total[24h])`

### 4.2 Gauge

**What**: A value that can go up AND down. Represents a current snapshot.
**Use for**: Current state — queue depth, active connections, memory usage, thread pool size.

```java
import io.micrometer.core.instrument.Gauge;
import java.util.concurrent.atomic.AtomicInteger;

@Service
public class QueueMetrics {

    private final AtomicInteger pendingPayouts = new AtomicInteger(0);

    public QueueMetrics(MeterRegistry registry) {
        // Gauge tracks the current value of pendingPayouts
        Gauge.builder("bharat.sme.payouts.queue.pending", pendingPayouts, AtomicInteger::get)
            .description("Number of payouts currently waiting to be processed")
            .register(registry);
    }

    public void payoutEnqueued() {
        pendingPayouts.incrementAndGet();
    }

    public void payoutProcessed() {
        pendingPayouts.decrementAndGet();
    }
}
```

```java
// Gauge from a method reference (tracks a live value)
Gauge.builder("bharat.sme.kafka.consumer.lag", kafkaConsumer, 
        consumer -> consumer.getLag("payouts-topic"))
    .description("Kafka consumer lag for payouts topic")
    .register(registry);

// Gauge from a collection size
Gauge.builder("bharat.sme.cache.size", corporateCache, Cache::size)
    .description("Number of entries in corporate cache")
    .register(registry);
```

**Counter vs Gauge — When to use which:**

| Scenario | Type | Why |
|----------|------|-----|
| Requests processed | Counter | Only goes up, you want rate |
| Active HTTP connections | Gauge | Goes up and down |
| Errors encountered | Counter | Only goes up, you want rate |
| Thread pool active threads | Gauge | Goes up and down |
| Kafka messages consumed | Counter | Only goes up |
| Kafka consumer lag | Gauge | Goes up and down |
| JVM heap memory used | Gauge | Goes up and down |
| Bytes transferred | Counter | Only goes up |

### 4.3 Timer

**What**: Measures the duration of events AND counts them. Essentially a counter + a distribution summary combined.
**Use for**: Latency measurement — API response times, DB query duration, external call duration.

```java
import io.micrometer.core.instrument.Timer;
import java.time.Duration;

@Service
public class CardHostMetrics {

    private final Timer debitTimer;
    private final MeterRegistry registry;

    public CardHostMetrics(MeterRegistry registry) {
        this.registry = registry;
        this.debitTimer = Timer.builder("bharat.sme.cardhost.debit.duration")
            .description("Time taken for card host debit API call")
            .publishPercentiles(0.5, 0.95, 0.99)  // p50, p95, p99
            .publishPercentileHistogram()           // enables histogram queries
            .minimumExpectedValue(Duration.ofMillis(10))
            .maximumExpectedValue(Duration.ofSeconds(30))
            .register(registry);
    }

    // Option 1: Wrap a Runnable
    public void timeDebitCall(Runnable debitCall) {
        debitTimer.record(debitCall);
    }

    // Option 2: Wrap a Supplier (returns a value)
    public <T> T timeDebitCall(Supplier<T> debitCall) {
        return debitTimer.record(debitCall);
    }

    // Option 3: Manual start/stop (for async or complex flows)
    public Timer.Sample startTimer() {
        return Timer.start(registry);
    }

    public void stopTimer(Timer.Sample sample, String status) {
        sample.stop(Timer.builder("bharat.sme.cardhost.debit.duration")
            .tag("status", status)
            .register(registry));
    }

    // Option 4: Record a known duration
    public void recordDuration(Duration duration) {
        debitTimer.record(duration);
    }
}
```

**Usage in your worker:**
```java
@Service
public class PayoutsTransactionsWorker {

    @Inject private MeterRegistry meterRegistry;

    public void processDebit(PayoutsTransaction txn) {
        Timer.Sample sample = Timer.start(meterRegistry);
        
        try {
            DebitResponse response = cardHostClient.debit(request);
            sample.stop(Timer.builder("bharat.sme.cardhost.debit.duration")
                .tag("status", response.getStatus().name())
                .tag("transfer_type", txn.getTransferType().name())
                .register(meterRegistry));
        } catch (Exception e) {
            sample.stop(Timer.builder("bharat.sme.cardhost.debit.duration")
                .tag("status", "error")
                .tag("error_type", e.getClass().getSimpleName())
                .register(meterRegistry));
            throw e;
        }
    }
}
```

**What a Timer gives you automatically:**
- `count` — how many times the event occurred
- `total_time` — sum of all durations (for calculating averages)
- `max` — maximum duration observed
- Percentiles (if configured): p50, p95, p99

### 4.4 Distribution Summary

**What**: Like a Timer but for non-time values. Tracks the distribution of values.
**Use for**: Request sizes, payload sizes, batch sizes, monetary amounts.

```java
import io.micrometer.core.instrument.DistributionSummary;

@Service
public class PayoutsDistributionMetrics {

    private final DistributionSummary payoutAmountSummary;
    private final DistributionSummary batchSizeSummary;

    public PayoutsDistributionMetrics(MeterRegistry registry) {
        this.payoutAmountSummary = DistributionSummary.builder("bharat.sme.payout.amount")
            .description("Distribution of payout amounts in INR")
            .baseUnit("INR")
            .publishPercentiles(0.5, 0.75, 0.95, 0.99)
            .minimumExpectedValue(100.0)
            .maximumExpectedValue(10_000_000.0)
            .register(registry);

        this.batchSizeSummary = DistributionSummary.builder("bharat.sme.payout.batch.size")
            .description("Number of transactions per payout batch")
            .publishPercentiles(0.5, 0.95)
            .register(registry);
    }

    public void recordPayoutAmount(double amountInRupees) {
        payoutAmountSummary.record(amountInRupees);
    }

    public void recordBatchSize(int size) {
        batchSizeSummary.record(size);
    }
}
```

### 4.5 Long Task Timer

**What**: Tracks the duration of currently active long-running tasks.
**Use for**: Tasks that take minutes/hours — file processing, batch jobs, report generation.

```java
import io.micrometer.core.instrument.LongTaskTimer;

@Service
public class FileProcessingMetrics {

    private final LongTaskTimer fileProcessingTimer;

    public FileProcessingMetrics(MeterRegistry registry) {
        this.fileProcessingTimer = LongTaskTimer.builder("bharat.sme.file.processing.active")
            .description("Currently active file processing tasks")
            .register(registry);
    }

    public void processFile(Payouts payouts) {
        LongTaskTimer.Sample sample = fileProcessingTimer.start();
        try {
            // ... process the salary file (could take minutes)
            doFileProcessing(payouts);
        } finally {
            sample.stop();
        }
    }
}
```

**Difference from regular Timer:**
- Regular Timer: Records duration AFTER the task completes
- Long Task Timer: Shows how many tasks are IN PROGRESS and how long they've been running

### 4.6 Function Counter & Function Timer

**What**: Wraps an existing object that already tracks counts/timing (e.g., a connection pool, a cache).
**Use for**: Exposing metrics from third-party libraries without modifying them.

```java
import io.micrometer.core.instrument.FunctionCounter;

// Example: Track Kafka consumer committed offsets
FunctionCounter.builder("bharat.sme.kafka.messages.consumed", kafkaConsumer,
        consumer -> consumer.getCommittedOffset("payouts-topic"))
    .description("Total Kafka messages consumed")
    .register(registry);
```

### Summary Table of Metric Types

| Type | Goes Up? | Goes Down? | Use Case | Example |
|------|----------|------------|----------|---------|
| Counter | ✅ | ❌ | Events, totals | Requests, errors, messages |
| Gauge | ✅ | ✅ | Current state | Queue depth, connections, memory |
| Timer | ✅ | ❌ | Duration + count | API latency, DB query time |
| Distribution Summary | ✅ | ❌ | Value distribution | Payload size, amounts |
| Long Task Timer | ✅ | ✅ | Active long tasks | File processing, batch jobs |

---

## 5. Custom Metrics vs Auto-Instrumented Metrics

### Auto-Instrumented Metrics (Free, out of the box)

Spring Boot Actuator + Micrometer automatically gives you:

| Category | Metrics | What They Tell You |
|----------|---------|-------------------|
| JVM Memory | `jvm.memory.used`, `jvm.memory.max`, `jvm.buffer.memory.used` | Memory pressure, leak detection |
| JVM GC | `jvm.gc.pause`, `jvm.gc.memory.promoted` | GC impact on latency |
| JVM Threads | `jvm.threads.live`, `jvm.threads.peak`, `jvm.threads.states` | Thread pool exhaustion |
| HTTP Server | `http.server.requests` (count, duration, status) | API performance |
| Connection Pools | `hikaricp.connections.active`, `hikaricp.connections.idle` | DB pool saturation |
| Kafka Consumer | `kafka.consumer.records.consumed`, `kafka.consumer.fetch.latency` | Consumer health |
| Kafka Producer | `kafka.producer.record.send.total`, `kafka.producer.request.latency` | Producer health |
| MongoDB | `mongodb.driver.commands` (if using Spring Data Mongo) | DB performance |
| Cache | `cache.gets`, `cache.puts`, `cache.evictions` | Cache effectiveness |
| System | `system.cpu.usage`, `process.cpu.usage`, `disk.free` | Resource utilization |

**You get ALL of these for free** just by having `spring-boot-starter-actuator` + a Micrometer registry on the classpath. No code needed.

### Custom Metrics (What you write yourself)

Custom metrics capture **business-specific** events that the framework can't know about:

```java
// The framework doesn't know what a "salary debit" is — you have to tell it
meterRegistry.counter("bharat.sme.salary.transfer.debit", "status", "success").increment();

// The framework doesn't know your payout batch sizes
meterRegistry.summary("bharat.sme.payout.batch.size").record(batchSize);

// The framework doesn't know about your card host API latency breakdown
Timer.Sample sample = Timer.start(meterRegistry);
// ... call card host ...
sample.stop(meterRegistry.timer("bharat.sme.cardhost.debit.duration", "status", status));
```

### When to Add Custom Metrics

Ask yourself: "Can I answer this question with auto-instrumented metrics?"

| Question | Auto-instrumented? | Custom needed? |
|----------|-------------------|----------------|
| "What's our API p99 latency?" | ✅ `http.server.requests` | ❌ |
| "How many salary payouts failed?" | ❌ | ✅ |
| "Is the DB connection pool saturated?" | ✅ `hikaricp.*` | ❌ |
| "What's the average payout batch size?" | ❌ | ✅ |
| "How long does the card host take?" | Partially (HTTP client metrics) | ✅ (with business context) |
| "What's our Kafka consumer lag?" | ✅ `kafka.consumer.*` | ❌ |
| "How many OTP verifications failed?" | ❌ | ✅ |

---

## 6. Micrometer — The Java Metrics Facade

### What is Micrometer?

Micrometer is to metrics what SLF4J is to logging — a **facade** (abstraction layer) that lets you write metrics code once and ship to any backend.

```
┌─────────────────────────────────────────────────────┐
│              Your Application Code                    │
│                                                      │
│   meterRegistry.counter("my.metric").increment()     │
└──────────────────────────┬───────────────────────────┘
                           │
                    ┌──────▼──────┐
                    │  Micrometer │  ← Facade/Abstraction
                    │  Core API   │
                    └──────┬──────┘
                           │
          ┌────────────────┼────────────────┐
          │                │                │
   ┌──────▼──────┐ ┌──────▼──────┐ ┌──────▼──────┐
   │  Prometheus │ │    OTLP     │ │   StatsD    │
   │  Registry   │ │  Registry   │ │  Registry   │
   └──────┬──────┘ └──────┬──────┘ └──────┬──────┘
          │                │                │
          ▼                ▼                ▼
   Prometheus        OTel Collector    Datadog/Graphite
```

### Why Micrometer (not raw OpenTelemetry SDK)?

1. **Spring Boot native** — first-class integration, auto-configuration
2. **Simpler API** — less boilerplate than raw OTel SDK
3. **Multiple registries** — ship to Prometheus AND OTLP simultaneously
4. **Auto-instrumentation** — JVM, HTTP, Kafka, DB metrics for free
5. **Battle-tested** — used by Netflix, VMware, and most Spring shops

### Core Concepts

#### MeterRegistry

The central component. All metrics are registered here. Spring Boot auto-configures one for you.

```java
@Service
public class MyService {
    
    private final MeterRegistry meterRegistry;
    
    // Spring injects the composite registry (which delegates to all configured registries)
    public MyService(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }
}
```

#### Composite Registry

When you have multiple registries (Prometheus + OTLP), Spring Boot creates a `CompositeMeterRegistry` that fans out to all of them:

```java
// You don't need to do this — Spring Boot does it automatically
// But conceptually:
CompositeMeterRegistry composite = new CompositeMeterRegistry();
composite.add(new PrometheusMeterRegistry(...));
composite.add(new OtlpMeterRegistry(...));

// When you record a metric, it goes to BOTH
composite.counter("my.metric").increment(); // → Prometheus AND OTLP
```

#### Meter Binders

Auto-register groups of related metrics:

```java
import io.micrometer.core.instrument.binder.jvm.*;
import io.micrometer.core.instrument.binder.system.*;

// These are auto-configured by Spring Boot Actuator, but you can add custom ones:
@Configuration
public class MetricsConfig {

    @Bean
    public MeterBinder kafkaLagBinder(KafkaConsumer consumer) {
        return registry -> {
            Gauge.builder("bharat.sme.kafka.lag", consumer, c -> c.getLag())
                .register(registry);
        };
    }
}
```

#### Common Tags

Apply tags to ALL metrics globally (great for environment, service name, version):

```java
@Configuration
public class MetricsConfig {

    @Bean
    public MeterRegistryCustomizer<MeterRegistry> commonTags(
            @Value("${spring.application.name}") String appName,
            @Value("${app.metrics.environment:local}") String env) {
        return registry -> registry.config()
            .commonTags(
                "application", appName,
                "env", env,
                "version", BuildInfo.getVersion()
            );
    }
}
```

Now every metric automatically gets `application=sme-service-workers, env=production, version=2.3.1`.

#### Metric Naming Conventions

Micrometer uses **dot notation** and automatically converts to the backend's convention:

| Your Code | Prometheus | Datadog/StatsD |
|-----------|-----------|----------------|
| `http.server.requests` | `http_server_requests_seconds_total` | `http.server.requests` |
| `jvm.memory.used` | `jvm_memory_used_bytes` | `jvm.memory.used` |

**Best practice for your service:**
```
bharat.sme.<domain>.<action>.<detail>

Examples:
bharat.sme.salary.transfer.debit          (counter)
bharat.sme.salary.transfer.debit.duration  (timer)
bharat.sme.payout.batch.size              (distribution summary)
bharat.sme.kafka.consumer.lag             (gauge)
```

---

## 7. OpenTelemetry — The Vendor-Neutral Standard

### What is OpenTelemetry (OTel)?

OpenTelemetry is a **CNCF project** that provides a single set of APIs, SDKs, and tools for collecting telemetry (metrics, logs, traces). It's the industry standard for vendor-neutral observability.

### OpenTelemetry vs Micrometer

| Aspect | Micrometer | OpenTelemetry |
|--------|-----------|---------------|
| Focus | Metrics (primarily) | Metrics + Traces + Logs (all three) |
| Spring Integration | Native, first-class | Good (via micrometer-tracing bridge) |
| API Complexity | Simple, Java-idiomatic | More verbose, language-agnostic |
| Auto-instrumentation | Via Spring Boot Actuator | Via Java Agent (zero-code) |
| Ecosystem | Java/Spring focused | Language-agnostic (Go, Python, JS, etc.) |
| Maturity (Java) | Very mature | Metrics stable, improving rapidly |

### How They Work Together (Recommended Approach)

You don't have to choose one or the other. The modern Spring Boot approach:

```
┌─────────────────────────────────────────────────────────────┐
│                    Your Application                           │
├─────────────────────────────────────────────────────────────┤
│  Metrics: Micrometer API                                     │
│    → micrometer-registry-otlp (exports via OTLP protocol)   │
│    → micrometer-registry-prometheus (for /actuator/prometheus)│
├─────────────────────────────────────────────────────────────┤
│  Tracing: Micrometer Tracing (Observation API)               │
│    → OpenTelemetry bridge (exports spans via OTLP)           │
├─────────────────────────────────────────────────────────────┤
│  Logs: SLF4J + Logback                                       │
│    → Structured JSON with traceId/spanId correlation         │
└─────────────────────────────────────────────────────────────┘
         │                    │                    │
         ▼                    ▼                    ▼
┌─────────────────────────────────────────────────────────────┐
│              OpenTelemetry Collector (or Groundcover)         │
│              Receives OTLP, exports to backends              │
└─────────────────────────────────────────────────────────────┘
```

### The OTLP Protocol

OTLP (OpenTelemetry Protocol) is the wire format for sending telemetry. It supports:
- **gRPC** (port 4317) — binary, efficient, bidirectional
- **HTTP/protobuf** (port 4318) — simpler, firewall-friendly

```yaml
# application.yml
management:
  otlp:
    metrics:
      export:
        enabled: true
        url: ${OTEL_EXPORTER_OTLP_ENDPOINT:http://localhost:4318}/v1/metrics
        step: 30s
    tracing:
      endpoint: ${OTEL_EXPORTER_OTLP_ENDPOINT:http://localhost:4318}/v1/traces
```

### OpenTelemetry Java Agent (Zero-Code Instrumentation)

The OTel Java Agent is a JAR you attach to your JVM that automatically instruments:
- HTTP clients/servers (RestTemplate, WebClient, Servlet)
- Database drivers (JDBC, MongoDB)
- Messaging (Kafka, RabbitMQ)
- gRPC, Redis, and 100+ libraries

```dockerfile
# In your Dockerfile
FROM eclipse-temurin:21-jre
COPY opentelemetry-javaagent.jar /opt/otel-agent.jar
COPY your-app.jar /opt/app.jar

ENV JAVA_TOOL_OPTIONS="-javaagent:/opt/otel-agent.jar"
ENV OTEL_SERVICE_NAME=sme-service-workers
ENV OTEL_EXPORTER_OTLP_ENDPOINT=http://otel-collector:4318

ENTRYPOINT ["java", "-jar", "/opt/app.jar"]
```

This gives you distributed tracing across all your services with ZERO code changes. The agent intercepts calls and creates spans automatically.

---

## 8. Micrometer vs OpenTelemetry — When to Use What

### Decision Matrix for Your SME Service

| Scenario | Recommendation | Why |
|----------|---------------|-----|
| Custom business metrics (counters, timers) | Micrometer API | Simpler, Spring-native |
| Exporting metrics to backends | micrometer-registry-otlp | Vendor-neutral wire format |
| Distributed tracing | Micrometer Tracing + OTel bridge | Best Spring integration |
| Auto-instrumentation (HTTP, Kafka, DB) | OTel Java Agent OR Spring Boot Actuator | Both work, agent is zero-code |
| Prometheus scraping (Groundcover) | micrometer-registry-prometheus | Already working for you |
| Structured logging with trace correlation | SLF4J + MDC + OTel context | Standard approach |

### The Recommended Stack for Your Service

```xml
<!-- pom.xml dependencies -->

<!-- Metrics API (you write code against this) -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-core</artifactId>
</dependency>

<!-- Export metrics via OTLP (vendor-neutral) -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-otlp</artifactId>
</dependency>

<!-- Export metrics via Prometheus (for Groundcover scraping) -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>

<!-- Distributed tracing via Micrometer Observation API -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-otel</artifactId>
</dependency>

<!-- OTLP span exporter -->
<dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-exporter-otlp</artifactId>
</dependency>

<!-- Spring Boot Actuator (auto-instrumentation) -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

---

## 9. Spring Boot Actuator & Auto-Configuration

### What Actuator Does for Observability

Spring Boot Actuator is the bridge between your app and observability backends:

```yaml
# application.yml — Full observability configuration
management:
  endpoints:
    web:
      exposure:
        include: health, info, prometheus, metrics
  
  # Metrics configuration
  metrics:
    distribution:
      percentiles-histogram:
        http.server.requests: true    # Enable histogram for HTTP metrics
      percentiles:
        http.server.requests: 0.5, 0.95, 0.99
      minimum-expected-value:
        http.server.requests: 10ms
      maximum-expected-value:
        http.server.requests: 30s
    tags:
      application: ${spring.application.name}
      env: ${APP_ENV:local}
    
    # OTLP export (vendor-neutral)
    export:
      otlp:
        enabled: true
        url: ${OTEL_EXPORTER_OTLP_METRICS_ENDPOINT:http://localhost:4318/v1/metrics}
        step: 30s
        resource-attributes:
          service.name: ${spring.application.name}
          service.version: ${app.version:unknown}
          deployment.environment: ${APP_ENV:local}
      
      # Prometheus export (for scraping)
      prometheus:
        enabled: true
  
  # Tracing configuration
  tracing:
    enabled: true
    sampling:
      probability: 1.0  # 100% in dev, reduce in prod (e.g., 0.1 = 10%)
    propagation:
      type: w3c  # W3C Trace Context (standard)

  # Health probes for Kubernetes
  endpoint:
    health:
      probes:
        enabled: true
      show-details: when-authorized
```

### The Observation API (Spring Boot 3+)

Spring Boot 3 introduced the **Observation API** — a single abstraction that produces BOTH metrics AND traces from the same instrumentation point:

```java
import io.micrometer.observation.Observation;
import io.micrometer.observation.ObservationRegistry;

@Service
public class CardHostService {

    private final ObservationRegistry observationRegistry;
    private final CardHostClient cardHostClient;

    public CardHostService(ObservationRegistry observationRegistry, CardHostClient cardHostClient) {
        this.observationRegistry = observationRegistry;
        this.cardHostClient = cardHostClient;
    }

    public DebitResponse debit(DebitRequest request) {
        // This single call creates:
        // 1. A Timer metric: cardhost.debit (with duration, count)
        // 2. A Trace span: "cardhost.debit" (with start/end time, tags)
        return Observation.createNotStarted("cardhost.debit", observationRegistry)
            .lowCardinalityKeyValue("transfer.type", request.getTransferType().name())
            .lowCardinalityKeyValue("corporate.id", request.getCorporateId())
            .highCardinalityKeyValue("transaction.id", request.getTxnId()) // only in traces, not metrics
            .observe(() -> cardHostClient.debit(request));
    }
}
```

**Key insight**: `lowCardinalityKeyValue` → goes into metrics tags (bounded values).
`highCardinalityKeyValue` → goes into trace span attributes only (unbounded values like IDs).

This is the future of Spring observability — one instrumentation point, two signals.

### @Observed Annotation (Even Simpler)

```java
import io.micrometer.observation.annotation.Observed;

@Service
public class PayoutsService {

    @Observed(
        name = "bharat.sme.payouts.process",
        contextualName = "process-payout-file",
        lowCardinalityKeyValues = {"payout.type", "salary"}
    )
    public void processPayoutFile(Payouts payouts) {
        // Automatically creates a timer metric + trace span
        // Duration is measured from method entry to exit
        // ...
    }
}
```

To enable `@Observed`, register the aspect:

```java
@Configuration
public class ObservationConfig {

    @Bean
    public ObservedAspect observedAspect(ObservationRegistry registry) {
        return new ObservedAspect(registry);
    }
}
```

---

## 10. Metric Registries & Exporters

### What is a Registry?

A registry is the **bridge** between your Micrometer code and a specific backend. Each backend has its own registry implementation that knows how to format and ship metrics.

### Available Registries

| Registry | Artifact | Backend | Protocol |
|----------|----------|---------|----------|
| Prometheus | `micrometer-registry-prometheus` | Prometheus/Grafana/Groundcover | Pull (scrape /actuator/prometheus) |
| OTLP | `micrometer-registry-otlp` | Any OTLP-compatible (Grafana, Datadog, New Relic) | Push (HTTP/gRPC) |
| StatsD | `micrometer-registry-statsd` | Datadog, Graphite, Telegraf | Push (UDP/TCP) |
| Datadog | `micrometer-registry-datadog` | Datadog only | Push (HTTP API) |
| New Relic | `micrometer-registry-new-relic` | New Relic only | Push (HTTP API) |
| CloudWatch | `micrometer-registry-cloudwatch2` | AWS CloudWatch | Push (AWS SDK) |

### Push vs Pull

**Pull (Prometheus model):**
```
Your App ←── scrape ←── Prometheus/Groundcover
  (exposes /actuator/prometheus endpoint)
  (backend pulls metrics every 15-30s)
```

**Push (OTLP/StatsD model):**
```
Your App ──── push ───→ OTel Collector / StatsD Agent
  (app sends metrics every 30-60s)
  (app is responsible for delivery)
```

**For your setup (Groundcover)**: You already have Prometheus scraping working. Adding OTLP push gives you flexibility to send to additional backends without changing your app.

### Multiple Registries (Recommended)

You can have BOTH Prometheus and OTLP active simultaneously:

```yaml
management:
  metrics:
    export:
      prometheus:
        enabled: true          # Groundcover scrapes this
      otlp:
        enabled: true          # Push to OTel Collector (future flexibility)
        url: http://otel-collector:4318/v1/metrics
```

Your code doesn't change — Micrometer's `CompositeMeterRegistry` fans out to both.

---

## 11. Distributed Tracing

### What is a Trace?

A trace represents the entire journey of a request through your system. It's composed of **spans**.

### What is a Span?

A span represents a single unit of work within a trace. Spans have:
- A name (e.g., "kafka.consume", "mongodb.find", "http.post")
- A start time and duration
- A parent span (forming a tree)
- Attributes/tags (metadata)
- Status (OK, ERROR)
- Events (timestamped annotations within the span)

### Trace Context Propagation

When Service A calls Service B, the trace context (traceId + spanId) must be propagated:

```
Service A (sme-service-apis)          Service B (sme-service-workers)
┌─────────────────────────┐          ┌─────────────────────────┐
│ Span: POST /payouts     │          │ Span: kafka.consume     │
│ traceId: abc123         │──Kafka──→│ traceId: abc123         │
│ spanId: span-1          │  header  │ parentSpanId: span-1    │
└─────────────────────────┘          │ spanId: span-2          │
                                     └─────────────────────────┘
```

**W3C Trace Context** (the standard) propagates via HTTP headers:
```
traceparent: 00-abc123def456789-span1234-01
tracestate: vendor=value
```

For Kafka, it's propagated via record headers.

### Setting Up Tracing in Spring Boot 3+

```xml
<!-- pom.xml -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-otel</artifactId>
</dependency>
<dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-exporter-otlp</artifactId>
</dependency>
```

```yaml
# application.yml
management:
  tracing:
    enabled: true
    sampling:
      probability: 0.1  # Sample 10% of traces in production
    propagation:
      type: w3c
  otlp:
    tracing:
      endpoint: http://otel-collector:4318/v1/traces
```

### Creating Custom Spans

```java
import io.micrometer.tracing.Tracer;
import io.micrometer.tracing.Span;

@Service
public class PayoutsTransactionsWorker {

    private final Tracer tracer;

    public PayoutsTransactionsWorker(Tracer tracer) {
        this.tracer = tracer;
    }

    public void processTransaction(PayoutsTransaction txn) {
        // Create a child span for the card host call
        Span cardHostSpan = tracer.nextSpan()
            .name("cardhost.debit")
            .tag("transaction.id", txn.getId())
            .tag("corporate.id", txn.getCorporateId())
            .tag("transfer.type", txn.getTransferType().name())
            .start();

        try (Tracer.SpanInScope ws = tracer.withSpan(cardHostSpan)) {
            DebitResponse response = cardHostClient.debit(request);
            cardHostSpan.tag("response.status", response.getStatus().name());
            cardHostSpan.tag("response.code", response.getResponseCode());
        } catch (Exception e) {
            cardHostSpan.error(e);
            throw e;
        } finally {
            cardHostSpan.end();
        }
    }
}
```

### Correlating Logs with Traces

The magic of observability: linking logs to traces so you can jump from a log line to the full trace.

```java
// Spring Boot 3 automatically puts traceId/spanId in MDC
// Your log pattern should include them:
```

```xml
<!-- logback-spring.xml -->
<configuration>
    <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <!-- Structured JSON logging with trace correlation -->
            <pattern>
                {"timestamp":"%d{ISO8601}","level":"%level","logger":"%logger{36}",
                 "traceId":"%X{traceId:-none}","spanId":"%X{spanId:-none}",
                 "message":"%msg"}%n
            </pattern>
        </encoder>
    </appender>
</configuration>
```

Now when you see a log line:
```json
{"timestamp":"2026-05-06T10:23:45","level":"ERROR","logger":"PayoutsTransactionsWorker",
 "traceId":"abc123def456","spanId":"span789",
 "message":"Debit failed for txnId=TX-12345, responseCode=51"}
```

You can search for `traceId=abc123def456` in your tracing backend (Tempo/Jaeger) and see the full request journey.

---

## 12. Structured Logging

### Why Structured Logging?

Traditional logs:
```
2026-05-06 10:23:45 ERROR PayoutsWorker - Debit failed for transaction TX-12345
```

Structured logs (JSON):
```json
{
  "timestamp": "2026-05-06T10:23:45.123Z",
  "level": "ERROR",
  "logger": "c.n.s.w.PayoutsTransactionsWorker",
  "traceId": "abc123def456",
  "spanId": "span789",
  "message": "Debit failed",
  "context": {
    "transactionId": "TX-12345",
    "corporateId": "CORP-789",
    "responseCode": "51",
    "reason": "Insufficient funds",
    "transferType": "NEFT",
    "amount": 50000.00
  }
}
```

Structured logs are **queryable**. You can search: `context.responseCode=51 AND context.corporateId=CORP-789`.

### Setting Up Structured Logging (Spring Boot 3.4+)

Spring Boot 3.4 introduced native structured logging support:

```yaml
# application.yml
logging:
  structured:
    format:
      console: ecs  # Elastic Common Schema (or "logstash" or "gelf")
```

For Spring Boot 3.x (before 3.4), use Logback with a JSON encoder:

```xml
<!-- logback-spring.xml -->
<configuration>
    <springProfile name="!local">
        <appender name="JSON" class="ch.qos.logback.core.ConsoleAppender">
            <encoder class="net.logstash.logback.encoder.LogstashEncoder">
                <includeMdcKeyName>traceId</includeMdcKeyName>
                <includeMdcKeyName>spanId</includeMdcKeyName>
                <customFields>{"service":"sme-service-workers"}</customFields>
            </encoder>
        </appender>
        <root level="INFO">
            <appender-ref ref="JSON" />
        </root>
    </springProfile>
    
    <springProfile name="local">
        <appender name="CONSOLE" class="ch.qos.logback.core.ConsoleAppender">
            <encoder>
                <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} [%X{traceId}] - %msg%n</pattern>
            </encoder>
        </appender>
        <root level="INFO">
            <appender-ref ref="CONSOLE" />
        </root>
    </springProfile>
</configuration>
```

### Adding Context to Logs (MDC)

```java
import org.slf4j.MDC;

public void processTransaction(PayoutsTransaction txn) {
    // Add business context to all logs within this scope
    MDC.put("transactionId", txn.getId());
    MDC.put("corporateId", txn.getCorporateId());
    MDC.put("transferType", txn.getTransferType().name());
    
    try {
        log.info("Processing debit transaction");
        // All log lines in this block will include the MDC context
        DebitResponse response = cardHostClient.debit(request);
        log.info("Debit completed with status: {}", response.getStatus());
    } finally {
        MDC.clear(); // Always clean up!
    }
}
```

---

## 13. Practical Implementation in Your SME Service

### Step 1: Create a Vendor-Neutral MetricsService

Replace the DD-specific abstraction with a clean, Micrometer-native service:

```java
package com.niyo.sme.commons.service.metrics;

import io.micrometer.core.instrument.Counter;
import io.micrometer.core.instrument.MeterRegistry;
import io.micrometer.core.instrument.Timer;
import io.micrometer.core.instrument.DistributionSummary;
import lombok.extern.slf4j.Slf4j;
import org.springframework.stereotype.Service;

import java.time.Duration;
import java.util.function.Supplier;

/**
 * Vendor-neutral metrics service built on Micrometer.
 * Ships to whatever backend is configured (Prometheus, OTLP, StatsD, etc.)
 */
@Service
@Slf4j
public class MetricsService {

    private static final String METRIC_PREFIX = "bharat.sme.";

    private final MeterRegistry registry;

    public MetricsService(MeterRegistry registry) {
        this.registry = registry;
    }

    /**
     * Increment a counter metric.
     * Use for: events that only go up (requests, errors, messages processed).
     *
     * @param metricName the metric name (will be prefixed with "bharat.sme.")
     * @param tags       key-value pairs (must be even number of strings)
     */
    public void increment(String metricName, String... tags) {
        try {
            registry.counter(METRIC_PREFIX + metricName, tags).increment();
        } catch (Exception e) {
            log.warn("Failed to record metric: {}", metricName, e);
        }
    }

    /**
     * Increment a counter with a specific amount.
     */
    public void incrementBy(String metricName, double amount, String... tags) {
        try {
            registry.counter(METRIC_PREFIX + metricName, tags).increment(amount);
        } catch (Exception e) {
            log.warn("Failed to record metric: {}", metricName, e);
        }
    }

    /**
     * Record a timed operation.
     * Use for: measuring how long something takes.
     *
     * @return Timer.Sample that you stop when the operation completes
     */
    public Timer.Sample startTimer() {
        return Timer.start(registry);
    }

    /**
     * Stop a timer and record the duration.
     */
    public void stopTimer(Timer.Sample sample, String metricName, String... tags) {
        try {
            sample.stop(registry.timer(METRIC_PREFIX + metricName, tags));
        } catch (Exception e) {
            log.warn("Failed to record timer: {}", metricName, e);
        }
    }

    /**
     * Time a supplier (returns a value).
     */
    public <T> T time(String metricName, Supplier<T> operation, String... tags) {
        Timer.Sample sample = Timer.start(registry);
        try {
            T result = operation.get();
            sample.stop(registry.timer(METRIC_PREFIX + metricName, tags));
            return result;
        } catch (Exception e) {
            sample.stop(registry.timer(METRIC_PREFIX + metricName, 
                mergeTags(tags, "error", e.getClass().getSimpleName())));
            throw e;
        }
    }

    /**
     * Record a duration directly (when you already know the elapsed time).
     */
    public void recordDuration(String metricName, Duration duration, String... tags) {
        try {
            registry.timer(METRIC_PREFIX + metricName, tags).record(duration);
        } catch (Exception e) {
            log.warn("Failed to record duration: {}", metricName, e);
        }
    }

    /**
     * Record a distribution value (non-time measurements like amounts, sizes).
     */
    public void recordValue(String metricName, double value, String... tags) {
        try {
            registry.summary(METRIC_PREFIX + metricName, tags).record(value);
        } catch (Exception e) {
            log.warn("Failed to record value: {}", metricName, e);
        }
    }

    /**
     * Get the underlying registry for advanced use cases.
     */
    public MeterRegistry getRegistry() {
        return registry;
    }

    private String[] mergeTags(String[] base, String... additional) {
        String[] merged = new String[base.length + additional.length];
        System.arraycopy(base, 0, merged, 0, base.length);
        System.arraycopy(additional, 0, merged, base.length, additional.length);
        return merged;
    }
}
```

### Step 2: Rename Constants (Drop DD Prefix)

```java
package com.niyo.sme.commons.constant;

/**
 * Centralized metric names for the SME service.
 * Convention: <domain>.<action>.<detail>
 * Full metric name will be: bharat.sme.<constant value>
 */
public final class MetricsConstants {

    private MetricsConstants() {}

    // ─── Salary / Payouts ────────────────────────────────────────────
    public static final String SALARY_FILE_UPLOAD_INITIATE = "salary.file.upload.initiate";
    public static final String SALARY_FILE_UPLOAD_STAGING = "salary.file.upload.staging";
    public static final String SALARY_FILE_VALIDATION = "salary.file.validation";
    public static final String SALARY_FILE_UPLOAD_SUCCESS = "salary.file.upload.success";
    public static final String SALARY_FILE_UPLOAD_FAILED = "salary.file.upload.failed";

    public static final String SALARY_OTP_TRIGGERED = "salary.otp.triggered";
    public static final String SALARY_OTP_VERIFY_SUCCESS = "salary.otp.verify.success";
    public static final String SALARY_OTP_VERIFY_FAILED = "salary.otp.verify.failed";

    public static final String SALARY_TRANSFER_DEBIT = "salary.transfer.debit";
    public static final String SALARY_TRANSFER_BANK = "salary.transfer.bank";
    public static final String SALARY_TRANSFER_REFUND = "salary.transfer.refund";
    public static final String SALARY_INTERNAL_TRANSACTION = "salary.internal_transaction";

    // ─── Collect Request ─────────────────────────────────────────────
    public static final String COLLECT_REQUEST_P2P = "collect_request.wallet_p2p";
    public static final String COLLECT_REQUEST_WALLET_DEBIT = "collect_request.wallet_debit";
    public static final String COLLECT_REQUEST_WALLET_CREDIT = "collect_request.wallet_credit";
    public static final String COLLECT_REQUEST_REVERSE_DEBIT = "collect_request.reverse_debit";

    // ─── Insurance ───────────────────────────────────────────────────
    public static final String INSURANCE_WALLET_DEBIT = "insurance.wallet.debit";

    // ─── Inward Remittance ───────────────────────────────────────────
    public static final String INWARD_TRANSFER = "inward.transfer";
    public static final String INWARD_REVERSAL = "inward.reversal";

    // ─── Corporate Lien ──────────────────────────────────────────────
    public static final String CORPORATE_LIEN_WALLET_TRANSFER = "corporate_lien.wallet_transfer";

    // ─── Bill Pay ────────────────────────────────────────────────────
    public static final String BILL_PAY_WALLET_DEBIT = "billpay.wallet_debit";
    public static final String BILL_PAY_WALLET_CREDIT = "billpay.wallet_credit";

    // ─── Revoke ──────────────────────────────────────────────────────
    public static final String REVOKE_CREDIT = "revoke.credit";

    // ─── Refund to Source ────────────────────────────────────────────
    public static final String REFUND_TO_SOURCE_CREDIT = "refund_to_source.credit";

    // ─── ICICI Banking ───────────────────────────────────────────────
    public static final String ICICI_BANKING_TXN_INITIATE = "icici_banking.transaction.initiate";

    // ─── Common Status Tags ──────────────────────────────────────────
    public static final String TAG_STATUS = "status";
    public static final String STATUS_SUCCESS = "success";
    public static final String STATUS_FAILED = "failed";
    public static final String STATUS_PENDING = "pending";
    public static final String STATUS_UNKNOWN = "unknown";
    public static final String STATUS_ERROR = "error";
    public static final String STATUS_INVALID = "invalid";
}
```

### Step 3: Usage in Workers (Before → After)

**Before (current code):**
```java
// Scattered, no tags, DD-specific naming
dataDogMetricsService.setMetrics(DatadogMetricsConstants.DD_SALARY_TRANSFER_DEBIT_SUCCESS);
dataDogMetricsService.setMetrics(DatadogMetricsConstants.DD_SALARY_TRANSFER_DEBIT_FAILED);
dataDogMetricsService.setMetrics(DatadogMetricsConstants.DD_SALARY_TRANSFER_DEBIT_PENDING);
```

**After (vendor-neutral with tags):**
```java
// Single metric name, status as a tag — much more powerful for querying
metricsService.increment(MetricsConstants.SALARY_TRANSFER_DEBIT,
    MetricsConstants.TAG_STATUS, MetricsConstants.STATUS_SUCCESS,
    "transfer_type", txn.getTransferType().name());

metricsService.increment(MetricsConstants.SALARY_TRANSFER_DEBIT,
    MetricsConstants.TAG_STATUS, MetricsConstants.STATUS_FAILED,
    "transfer_type", txn.getTransferType().name(),
    "error_code", response.getResponseCode());

metricsService.increment(MetricsConstants.SALARY_TRANSFER_DEBIT,
    MetricsConstants.TAG_STATUS, MetricsConstants.STATUS_PENDING,
    "transfer_type", txn.getTransferType().name());
```

**Even better — with timing:**
```java
public void processDebit(PayoutsTransaction txn) {
    Timer.Sample timer = metricsService.startTimer();
    
    try {
        DebitResponse response = cardHostClient.debit(request);
        
        String status = switch (response.getStatus()) {
            case SUCCESS -> MetricsConstants.STATUS_SUCCESS;
            case ERROR, INVALID -> MetricsConstants.STATUS_FAILED;
            default -> MetricsConstants.STATUS_UNKNOWN;
        };
        
        // Records both the count AND the duration
        metricsService.stopTimer(timer, "salary.transfer.debit.duration",
            MetricsConstants.TAG_STATUS, status,
            "transfer_type", txn.getTransferType().name());
        
        // Also increment the counter for simple dashboards
        metricsService.increment(MetricsConstants.SALARY_TRANSFER_DEBIT,
            MetricsConstants.TAG_STATUS, status,
            "transfer_type", txn.getTransferType().name());
            
    } catch (Exception e) {
        metricsService.stopTimer(timer, "salary.transfer.debit.duration",
            MetricsConstants.TAG_STATUS, MetricsConstants.STATUS_ERROR,
            "error_type", e.getClass().getSimpleName());
        throw e;
    }
}
```

### Step 4: Configuration

```yaml
# application.yml (base — all environments)
management:
  endpoints:
    web:
      exposure:
        include: health, info, prometheus, metrics
  endpoint:
    health:
      probes:
        enabled: true
  metrics:
    tags:
      application: ${spring.application.name}
    distribution:
      percentiles-histogram:
        bharat.sme: true
      percentiles:
        bharat.sme: 0.5, 0.95, 0.99
    export:
      prometheus:
        enabled: true
      otlp:
        enabled: ${OTEL_METRICS_ENABLED:false}
        url: ${OTEL_EXPORTER_OTLP_ENDPOINT:http://localhost:4318}/v1/metrics
        step: 30s
        resource-attributes:
          service.name: ${spring.application.name}
          deployment.environment: ${APP_ENV:local}
  tracing:
    enabled: ${OTEL_TRACING_ENABLED:false}
    sampling:
      probability: ${OTEL_SAMPLING_PROBABILITY:1.0}
    propagation:
      type: w3c
  otlp:
    tracing:
      endpoint: ${OTEL_EXPORTER_OTLP_ENDPOINT:http://localhost:4318}/v1/traces
```

```yaml
# application-prod.yml
management:
  metrics:
    export:
      otlp:
        enabled: true
      prometheus:
        enabled: true
  tracing:
    enabled: true
    sampling:
      probability: 0.1  # 10% sampling in production
```

```yaml
# application-dev.yml
management:
  metrics:
    export:
      otlp:
        enabled: true
      prometheus:
        enabled: true
  tracing:
    enabled: true
    sampling:
      probability: 1.0  # 100% sampling in dev
```

### Step 5: Dependencies (pom.xml)

```xml
<!-- REMOVE these -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-statsd</artifactId>
</dependency>
<dependency>
    <groupId>io.opentracing</groupId>
    <artifactId>opentracing-util</artifactId>
    <version>0.33.0</version>
</dependency>

<!-- KEEP this -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>

<!-- ADD these -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-otlp</artifactId>
</dependency>

<!-- For distributed tracing -->
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-otel</artifactId>
</dependency>
<dependency>
    <groupId>io.opentelemetry</groupId>
    <artifactId>opentelemetry-exporter-otlp</artifactId>
</dependency>

<!-- For @Observed annotation support -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>
```

---

## 14. Observability Backends & Collectors

### The OpenTelemetry Collector

The OTel Collector is a vendor-neutral proxy that receives, processes, and exports telemetry:

```
┌──────────────┐     ┌─────────────────────────────────────┐     ┌──────────────┐
│ Your App     │────→│       OTel Collector                 │────→│ Prometheus   │
│ (OTLP push)  │     │                                     │────→│ Tempo/Jaeger │
└──────────────┘     │  Receivers → Processors → Exporters │────→│ Loki         │
                     └─────────────────────────────────────┘     └──────────────┘
```

**Why use a collector instead of sending directly to backends?**
1. **Decoupling** — change backends without changing app config
2. **Batching & retry** — collector handles network failures
3. **Processing** — sampling, filtering, enriching telemetry
4. **Fan-out** — send to multiple backends from one source

### Groundcover (Your Current Backend)

Groundcover is an eBPF-based observability platform that:
- Scrapes Prometheus metrics from `/actuator/prometheus` (already working for you)
- Can receive OTLP data
- Provides metrics, logs, and traces in one platform
- Runs as a DaemonSet in your Kubernetes cluster

Your current setup sends metrics to `groundcover-vector.groundcover.svc.cluster.local:8125` via StatsD. The cleaner path is to let Groundcover scrape Prometheus (which it already does) and optionally push via OTLP.

### Common Backend Stacks

| Stack | Metrics | Traces | Logs | Notes |
|-------|---------|--------|------|-------|
| Groundcover | ✅ (Prometheus scrape) | ✅ (OTLP) | ✅ (eBPF) | Your current setup |
| Grafana Cloud | Mimir (Prometheus) | Tempo | Loki | Popular managed option |
| Datadog | ✅ (OTLP or StatsD) | ✅ (OTLP) | ✅ | Expensive but full-featured |
| AWS CloudWatch | ✅ (OTLP via ADOT) | X-Ray | CloudWatch Logs | AWS-native |
| Self-hosted | Prometheus + Grafana | Jaeger/Tempo | OpenSearch/Loki | Full control |

---

## 15. Best Practices & Anti-Patterns

### DO ✅

1. **Use tags for dimensions, not metric names**
   ```java
   // GOOD
   metricsService.increment("salary.transfer", "status", "success", "type", "NEFT");
   metricsService.increment("salary.transfer", "status", "failed", "type", "NEFT");
   
   // BAD
   metricsService.increment("salary.transfer.neft.success");
   metricsService.increment("salary.transfer.neft.failed");
   ```

2. **Keep tag cardinality bounded**
   ```java
   // GOOD — corporate_id has ~500 values
   "corporate_id", corporateId
   
   // BAD — transaction_id has millions of values
   "transaction_id", txnId  // Use this in traces/logs, not metrics
   ```

3. **Name metrics hierarchically**
   ```
   bharat.sme.salary.transfer.debit          ← domain.subdomain.action
   bharat.sme.salary.transfer.debit.duration  ← add .duration for timers
   bharat.sme.kafka.consumer.lag              ← infrastructure metrics
   ```

4. **Always add a description**
   ```java
   Counter.builder("bharat.sme.salary.transfer.debit")
       .description("Number of salary debit transactions attempted")
       .register(registry);
   ```

5. **Use Timer for anything with latency**
   ```java
   // Every external call should be timed
   Timer.Sample sample = Timer.start(registry);
   try {
       result = externalService.call();
       sample.stop(registry.timer("external.call.duration", "status", "success"));
   } catch (Exception e) {
       sample.stop(registry.timer("external.call.duration", "status", "error"));
   }
   ```

6. **Fail silently on metrics errors** — metrics should never break business logic
   ```java
   try {
       registry.counter("my.metric").increment();
   } catch (Exception e) {
       log.warn("Metric recording failed", e); // Don't throw!
   }
   ```

### DON'T ❌

1. **Don't use high-cardinality values as tags**
   ```java
   // NEVER — creates millions of time series
   .tag("user_id", userId)
   .tag("transaction_id", txnId)
   .tag("timestamp", Instant.now().toString())
   .tag("request_body", body)
   ```

2. **Don't create metrics in hot loops without caching**
   ```java
   // BAD — creates a new Counter lookup every iteration
   for (Transaction txn : transactions) {
       meterRegistry.counter("txn.processed").increment();
   }
   
   // GOOD — cache the counter reference
   Counter counter = meterRegistry.counter("txn.processed");
   for (Transaction txn : transactions) {
       counter.increment();
   }
   ```

3. **Don't use metrics for debugging** — use logs/traces for that
   ```java
   // BAD — this is a log, not a metric
   metricsService.increment("debug.reached.line.245");
   
   // GOOD — this is a meaningful business event
   metricsService.increment("salary.transfer.debit", "status", "failed");
   ```

4. **Don't forget to handle the "unknown" state**
   ```java
   // BAD — only tracks success/failure, misses pending/unknown
   if (success) increment("success");
   else increment("failed");
   
   // GOOD — tracks all states
   String status = switch (response.getStatus()) {
       case SUCCESS -> "success";
       case ERROR, INVALID -> "failed";
       case PENDING -> "pending";
       default -> "unknown";
   };
   metricsService.increment("salary.transfer", "status", status);
   ```

5. **Don't mix counter and gauge semantics**
   ```java
   // BAD — using a counter for something that goes down
   counter("active.connections").increment();  // When does it decrement?
   
   // GOOD — use a gauge for values that go up and down
   Gauge.builder("active.connections", pool, Pool::getActiveCount).register(registry);
   ```

### Alerting Best Practices

Define alerts on **rates and ratios**, not absolute values:

```yaml
# Good alerts (PromQL examples)
# Error rate > 5% over 5 minutes
rate(bharat_sme_salary_transfer_debit_total{status="failed"}[5m]) 
  / rate(bharat_sme_salary_transfer_debit_total[5m]) > 0.05

# p99 latency > 5 seconds
histogram_quantile(0.99, rate(bharat_sme_cardhost_debit_duration_seconds_bucket[5m])) > 5

# No successful payouts in 10 minutes (dead service)
rate(bharat_sme_salary_transfer_debit_total{status="success"}[10m]) == 0
```

---

## 16. Spring Boot 4 & Java 21/25 Considerations

### Spring Boot 4 (Expected 2025-2026)

Spring Boot 4 will bring:

1. **Jakarta EE 11 baseline** — you're already on Jakarta (good, since you migrated from `javax.inject` to `jakarta.inject`)

2. **Java 21 minimum** — virtual threads become the default for I/O-bound work

3. **Micrometer 2.x** — potential API changes (but likely backward-compatible for basic usage)

4. **Native OpenTelemetry support** — deeper integration without bridges

5. **Structured logging built-in** — no more third-party JSON encoders needed

### Java 21 Features Relevant to Observability

#### Virtual Threads (Project Loom)

Virtual threads change how you think about thread pool metrics:

```java
// Traditional threads — you monitor thread pool saturation
Gauge.builder("thread.pool.active", executor, ThreadPoolExecutor::getActiveCount)
    .register(registry);

// Virtual threads — thread count is less meaningful
// Instead, monitor: request concurrency, queue depth, latency
Gauge.builder("requests.active", this, self -> self.activeRequests.get())
    .register(registry);
```

With virtual threads enabled in Spring Boot:
```yaml
spring:
  threads:
    virtual:
      enabled: true  # Spring Boot 3.2+
```

The auto-instrumented metrics adapt automatically — `http.server.requests` still works, but the underlying thread model is different.

#### Structured Concurrency (Preview in Java 21, stable path to 25)

```java
// Java 21+ structured concurrency with observability
try (var scope = new StructuredTaskScope.ShutdownOnFailure()) {
    // Each subtask gets its own span (if tracing is configured)
    Subtask<DebitResponse> debit = scope.fork(() -> cardHostClient.debit(request));
    Subtask<NotificationResult> notify = scope.fork(() -> notificationService.send(notification));
    
    scope.join().throwIfFailed();
    
    // Both complete or both fail — clean observability
    metricsService.increment("salary.transfer.debit", "status", "success");
}
```

#### Scoped Values (Java 21 Preview → Java 25 Stable)

Scoped values replace ThreadLocal for passing context (like trace IDs) through virtual threads:

```java
// Java 25 — ScopedValue for observability context
private static final ScopedValue<String> TRACE_ID = ScopedValue.newInstance();
private static final ScopedValue<String> CORPORATE_ID = ScopedValue.newInstance();

public void processRequest(Request request) {
    ScopedValue.where(TRACE_ID, request.getTraceId())
               .where(CORPORATE_ID, request.getCorporateId())
               .run(() -> {
                   // All code in this scope (including virtual thread forks)
                   // can access TRACE_ID.get() and CORPORATE_ID.get()
                   processDebit(request);
               });
}
```

This is relevant because Micrometer Tracing currently uses ThreadLocal for context propagation. Virtual threads + ScopedValues will be the future-proof approach.

#### Pattern Matching & Records (Java 21+)

Cleaner metric recording with modern Java:

```java
// Java 21+ pattern matching for status mapping
String status = switch (response) {
    case DebitResponse r when r.getStatus() == Status.SUCCESS -> "success";
    case DebitResponse r when r.getStatus() == Status.ERROR -> "failed";
    case DebitResponse r when r.getStatus() == Status.PENDING -> "pending";
    case null -> "error";
    default -> "unknown";
};

metricsService.increment(MetricsConstants.SALARY_TRANSFER_DEBIT,
    MetricsConstants.TAG_STATUS, status);
```

```java
// Records for metric events (immutable, concise)
public record MetricEvent(String name, String status, Map<String, String> tags) {
    public MetricEvent(String name, String status) {
        this(name, status, Map.of());
    }
}
```

### Java 25 (Expected September 2025)

Key features for observability:
- **Stable Scoped Values** — better context propagation than ThreadLocal
- **Stable Structured Concurrency** — clean parent-child span relationships
- **Stable Stream Gatherers** — useful for metric aggregation pipelines
- **Compact Object Headers** — reduced memory overhead (better JVM metrics)

### Migration Path: Spring Boot 3 → 4

Your observability code migration will be minimal if you follow this guide:

1. **Micrometer API stays the same** — `MeterRegistry`, `Counter`, `Timer` etc. won't change
2. **Configuration keys may shift** — watch for `management.metrics.*` → potential restructuring
3. **OpenTelemetry becomes first-class** — the bridge dependency may become unnecessary
4. **`javax.inject` → fully gone** — you'll need `jakarta.inject` everywhere (you're already migrating)

---

## 17. Glossary

| Term | Definition |
|------|-----------|
| **Metric** | A numeric measurement aggregated over time (counter, gauge, timer, etc.) |
| **Custom Metric** | A metric you define in code for business-specific events (vs auto-instrumented) |
| **Counter** | A metric that only increases (events, totals). Reset on restart. |
| **Gauge** | A metric that can increase or decrease (current state snapshots). |
| **Timer** | Measures duration AND count of events. Combines counter + histogram. |
| **Histogram** | Distribution of values across configurable buckets. Enables percentile queries. |
| **Distribution Summary** | Like a Timer but for non-time values (sizes, amounts). |
| **Tag / Label** | Key-value metadata attached to a metric for dimensional filtering. |
| **Cardinality** | The number of unique time series a metric produces (name × tag combinations). |
| **Time Series** | A unique combination of metric name + tag values, tracked over time. |
| **Percentile (p50/p95/p99)** | The value below which X% of observations fall. p99 = 99th percentile. |
| **SLO** | Service Level Objective — a target for metric values (e.g., "p99 latency < 500ms"). |
| **SLI** | Service Level Indicator — the metric that measures an SLO. |
| **Trace** | The full journey of a request across services, composed of spans. |
| **Span** | A single unit of work within a trace (e.g., one HTTP call, one DB query). |
| **Trace Context** | The traceId + spanId propagated between services (W3C standard). |
| **MDC** | Mapped Diagnostic Context — thread-local key-value store for log enrichment. |
| **OTLP** | OpenTelemetry Protocol — the standard wire format for telemetry data. |
| **Micrometer** | Java metrics facade (like SLF4J for logging). Write once, export anywhere. |
| **OpenTelemetry** | CNCF standard for vendor-neutral observability (metrics + traces + logs). |
| **Registry** | Micrometer component that bridges your code to a specific backend. |
| **Exporter** | Component that ships telemetry to a backend (push model). |
| **Scrape** | Backend pulls metrics from your app's endpoint (pull model, Prometheus). |
| **Collector** | Proxy that receives, processes, and forwards telemetry (OTel Collector). |
| **Instrumentation** | Code that produces telemetry (manual or automatic). |
| **Auto-instrumentation** | Telemetry produced without writing code (Spring Actuator, OTel Agent). |
| **Observation API** | Spring Boot 3+ unified API that produces both metrics AND traces. |
| **Sampling** | Only recording a percentage of traces to reduce volume/cost. |
| **Fan-out** | Sending the same telemetry to multiple backends simultaneously. |
| **StatsD** | UDP-based protocol for pushing metrics (used by Datadog, Graphite). |
| **Prometheus** | Pull-based metrics system. Scrapes /metrics endpoints periodically. |
| **PromQL** | Prometheus Query Language for querying and alerting on metrics. |
| **Groundcover** | eBPF-based observability platform (your current backend). |
| **eBPF** | Linux kernel technology for observing system calls without app changes. |
| **Virtual Threads** | Java 21+ lightweight threads for I/O-bound work (Project Loom). |
| **Scoped Values** | Java 21+ replacement for ThreadLocal, works with virtual threads. |

---

## Quick Reference: What to Use When

```
┌─────────────────────────────────────────────────────────────────────┐
│ "How many X happened?"              → Counter                        │
│ "What is the current X?"            → Gauge                          │
│ "How long does X take?"             → Timer                          │
│ "What's the distribution of X?"     → Distribution Summary           │
│ "How many X are in progress?"       → Long Task Timer                │
│ "Why did X fail?"                   → Logs (with traceId)            │
│ "Where is the bottleneck in X?"     → Distributed Trace              │
│ "Is X degraded right now?"          → Gauge + Timer p99 + Alert      │
└─────────────────────────────────────────────────────────────────────┘
```

---

## Further Reading

- [Micrometer Documentation](https://micrometer.io/docs)
- [Spring Boot Actuator Metrics](https://docs.spring.io/spring-boot/docs/current/reference/html/actuator.html#actuator.metrics)
- [OpenTelemetry Java SDK](https://opentelemetry.io/docs/languages/java/)
- [Micrometer Tracing](https://micrometer.io/docs/tracing)
- [Spring Boot Observability (Official Guide)](https://spring.io/blog/2022/10/12/observability-with-spring-boot-3)
- [Prometheus Best Practices: Naming](https://prometheus.io/docs/practices/naming/)
- [OpenTelemetry Collector](https://opentelemetry.io/docs/collector/)
