---
title: Kafka Operations
tags: [system-design, messaging, aws]
type: note
status: evergreen
created: 2026-10-01
updated: 2026-10-14
summary: What running Kafka actually costs — self-managed vs AWS MSK vs Confluent Cloud, the monitoring that matters, common incidents, and rough capacity rules.
related: ["[[Kafka Architecture]]", "[[Choosing a Message Broker]]", "[[Consumer Groups and Rebalancing]]", "[[Capacity Planning]]"]
---

# Kafka Operations

[[Kafka Architecture]] explains why Kafka is powerful; this note is about the bill. Kafka is one of the heaviest things a team can self-operate: a stateful, disk-bound, consensus-running distributed system where the worst failures are silent (data aging out unread) rather than loud. The managed offerings — AWS MSK and Confluent Cloud — remove different slices of that burden at different prices, and none of them removes the part that most often causes incidents: *your* partitions, configs, and clients.

## The problem that created it

(Here "it" = the managed-Kafka market, and this note's checklist.) Teams adopt Kafka for the architecture and discover the operations: a growing system starts with a happy 3-broker cluster, then hits its first disk-full at 3 a.m., its first stuck rebalance during a deploy, its first "the upgrade notes say roll brokers one at a time and wait for ISR to recover". A broker is not a stateless pod you can kill and reschedule — it owns terabytes of partition replicas that must re-replicate when it moves ([[Stateless vs Stateful Services]]). The question "who carries the pager for the broker layer?" is what MSK and Confluent exist to answer.

## What it solves

Operating Kafka *well* solves, concretely:

- **Durability promises actually holding** — RF/min.insync math is void if URP sits at 50 for a week unnoticed.
- **Capacity ahead of growth** — brokers, disks, and partition counts sized before the firehose grows, not after.
- **Safe change** — rolling upgrades, config changes, and partition reassignment without downtime or data loss.
- **Early warning** — lag and ISR monitoring that pages before data loss, not after.

## How it works

### Self-managed: the full job description

- **Broker sizing.** Kafka is disk-throughput- and network-bound before CPU-bound. You size for: write rate × replication factor (every byte in is written RF times and crosses the network RF−1 times), plus consumer fan-out reads, plus re-replication headroom — a dead broker's recovery traffic competes with live traffic. Page cache is the read path, so RAM beyond the heap matters; the JVM heap itself stays modest (commonly ~6 GiB) because the data lives in the OS cache.
- **Partition count planning.** Partitions are the unit of parallelism *and* of controller/metadata load. Too few: consumers can't scale out. Too many: slower leader elections and recovery, more open file handles, more replication fetchers. Changing counts later breaks key ordering ([[Kafka Architecture]]), so this is up-front [[Capacity Planning]].
- **Rebalances and reassignment.** Adding a broker moves nothing automatically (in vanilla Apache Kafka) — you run partition reassignment, which copies whole replicas and must be throttled or it starves production traffic. Cruise Control exists because humans are bad at this.
- **Upgrades.** Rolling, one broker at a time, waiting for ISR to fully recover between each; protocol/metadata version bumps are separate staged steps. A 6-broker upgrade is an afternoon of supervised work even when nothing goes wrong.
- **Monitoring that matters** (the short list):
  - **Under-replicated partitions (URP)** — should be 0; sustained >0 means degraded durability.
  - **Consumer lag** per group (Burrow, exported lag metrics) — trend, not just absolute value.
  - **ISR shrink/expand rate** — flapping ISRs mean a broker or network is marginal.
  - **Offline partitions / active controller count** (exactly 1) — cluster-level health.
  - Disk fill %, request handler idle %, produce/fetch p99 latency.
  - On the client side: producer `record-error-rate` and retry rate (min.insync violations surface here as `NotEnoughReplicas`), and consumer rebalance frequency per group.
- **Controlled shutdown and restarts.** A broker stopped gracefully hands off leadership first (controlled shutdown); a broker killed hard forces leader elections for everything it led and a longer log-recovery scan on restart. On Kubernetes this means generous termination grace periods and never treating brokers like cattle — relevant to any EKS-hosted cluster.

### AWS MSK: what AWS takes vs what stays yours

MSK runs the brokers (and, pre-KRaft versions, ZooKeeper) for you: provisioning, broker VM patching, broker-failure replacement, encryption, IAM integration, and tiered storage on Standard brokers — or Express brokers, whose storage is pay-as-you-go and never sized (3-AZ only; Kafka 3.6/3.8/3.9/4.2 as of 2026). **What stays yours is most of what actually pages:** topic design, partition counts, replication/min.insync configs, client tuning, consumer lag, rebalance storms, partition skew — MSK has no opinion about your partitions. You also still choose broker counts/sizes and trigger version upgrades (MSK orchestrates the roll). MSK Serverless shifts capacity management to AWS for spiky/small workloads at a per-throughput price. Honest summary: MSK converts "run the servers" to a line item; it does not convert Kafka into [[SQS and SNS|SQS]].

### Confluent Cloud: what the premium buys

Confluent sells Kafka-as-a-utility plus the ecosystem: elastic capacity (scale without thinking in brokers — their units abstract throughput), **Schema Registry** managed ([[Event Schema Evolution]]), a large catalog of **managed connectors** (no self-run Kafka Connect cluster), **tiered storage** by default (long retention without big disks), ksqlDB/Flink offerings, and strong multi-region/linking features. The premium is real — list prices run well above MSK for the same raw throughput — and what it buys is the *rest* of the platform plus near-zero broker ops. Client-side concerns (partition strategy, consumer lag, poison messages) remain yours, always, everywhere.

| | Self-managed | AWS MSK | Confluent Cloud |
|---|---|---|---|
| Broker provisioning/patching/failure | You | AWS | Confluent |
| Upgrades | You, manual roll | You trigger, AWS rolls | Invisible |
| Scaling | You + reassignment | You resize, AWS helps | Elastic |
| Schema registry, connectors | You run them | Mostly you (some managed) | Managed |
| Partition/topic design, client configs, lag | **You** | **You** | **You** |
| Cost shape | Instances + EBS + people-time | Broker-hours + storage + transfer | Throughput + storage + features premium |

### Tiered storage — how it changes the retention math

Classic Kafka couples retention to broker disks: 30-day retention means 30 days × write rate × RF sitting on expensive attached storage, and every broker replacement re-replicates all of it. **Tiered storage** (KIP-405 in Apache Kafka, marked production-ready in 3.9; long offered by Confluent, and available on MSK for supported versions) splits each partition into a hot local tail and a cold remote tier (S3-class object storage). Consequences: retention stops being a disk-sizing problem (the `disk = rate × RF × retention` rule below shrinks to the local hot window), historical reads come back slower but no longer evict page cache for tail readers, and broker recovery/reassignment only moves the local tail — the single biggest reduction in self-managed Kafka's operational pain in years. The catch: reads from the cold tier add latency and object-store request costs, and compacted topics are not tierable. This is a large part of what Confluent's "infinite retention" premium actually is.

### Common incidents (the ones you'll actually see)

1. **Unbalanced partitions / broker skew.** Some brokers lead far more (or hotter) partitions; they run hot while others idle, and the cluster's ceiling becomes one broker's ceiling. Caused by key skew ([[Hot Partition Problem]]), naive assignment after adding brokers, or leader clustering after restarts. Fix: preferred-leader election, reassignment, Cruise Control; redesign keys for the skew case.
2. **Consumer-lag storms.** A consumer slows (downstream DB degraded, GC, code regression) → lag grows → when it recovers it fetches huge batches, hammering the same downstream → crash/timeout → [[Consumer Groups and Rebalancing|rebalance]] → duplicates from redelivery → more load. Classic metastable [[Failure Modes in Distributed Systems|failure]]. Mitigate: max.poll.records caps, [[Backpressure and Flow Control|backpressure]] into the consumer, pause/resume, alert on lag *slope*.
3. **Retention-caused data loss.** Lag quietly crosses the retention window; segments age out unread; nothing errors. Weeks later reconciliation finds the hole. Mitigate: alert when `lag (in time) > retention × 0.5`; size retention against worst-case consumer outage, not disk comfort.
4. **Too many partitions.** Thousands of partitions per broker → longer leader elections, slow restarts/failover, metadata bloat, file-handle exhaustion. KRaft raised the ceilings dramatically vs ZooKeeper, but clients and brokers still pay per-partition costs. Symptom: controlled restart of one broker takes tens of minutes to clear URP.

### Capacity rules of thumb — **all rough (2026), verify against your own benchmarks**

- A partition sustains on the order of **single-digit to low-tens of MB/s** writes in typical setups — benchmark yours; message size and compression dominate.
- Partition count ≈ `max(target throughput ÷ measured per-partition producer rate, target consumer parallelism)`, then add ~2× headroom because raising it later breaks ordering.
- Keep per-broker partition replica counts in the **low thousands** at most (KRaft tolerates more than ZooKeeper-era ~4k guidance, but client metadata and recovery costs still accumulate).
- Disk = `write rate × RF × retention period`, plus ~30–40% free-space headroom (compaction and reassignment need scratch space).
- Network: every produced byte costs roughly `RF − 1` inter-broker copies **plus one per consumer group reading it** — fan-out, not produce rate, often saturates NICs first.

## Trade-offs

| Option | You gain | You pay |
|---|---|---|
| Self-managed | Full control, lowest infra cost at scale, any version/plugin | A real SRE workload; pager, upgrades, capacity engineering |
| MSK | Broker ops gone, AWS-native (IAM, VPC), moderate cost | Still own all Kafka-level design/tuning; version lag; less escape hatch when brokers misbehave |
| Confluent | Near-zero platform ops, ecosystem included, elastic | Highest cost; vendor coupling; still own client-side correctness |

The through-line: **money buys away broker ops, never client correctness.** Partitioning, idempotency, lag, and schema discipline are yours on every tier — which is why "we moved to managed Kafka" rarely reduces incident count as much as hoped.

## Failure modes

- **Silent durability erosion**: URP ignored → one failure from loss while dashboards look green.
- **The 3 a.m. disk-full**: retention sized to disk instead of disk to retention; brokers crash on full disks and recovery is slow and manual.
- **Upgrade-triggered instability**: rolling a broker before ISR recovered from the previous one → min.insync violations → produce failures platform-wide.
- **Cost surprise on managed tiers**: cross-AZ traffic can rival the broker line item — inter-broker replication on self-managed EC2 (MSK does not bill replication between brokers), and producer/consumer traffic crossing AZs to partition leaders on every AWS option, multiplied by consumer fan-out; throughput-priced tiers punish chatty, uncompressed producers.
- **"Managed" complacency**: teams stop watching lag because "AWS runs Kafka now" — but lag was never AWS's job.

## When NOT to use it

(When not to *run* Kafka at all — see [[Choosing a Message Broker]] for the full framework.)

- Throughput measured in hundreds of msgs/sec with no replay requirement: [[SQS and SNS|SQS]] deletes this entire note from your life.
- No one owns the pager: self-managed Kafka without an owning team is an outage subscription.
- Self-managing to "save money" at low sustained throughput: people-time swamps the infra savings — the cost shape of self-managing is dominated by humans, not instances.

## In your stack

- You've lived all three columns of the table — self-managed, MSK, Confluent — and then left for SQS. That journey is itself the lesson: each hop paid money or capability to shed ops ([[Choosing a Message Broker]] maps it).
- On EKS, remember a Kafka broker is the opposite of your stateless services: it can't be treated like a 1-replica pod that reschedules freely; every broker move triggers re-replication traffic.
- The lag-vs-retention alert is the one to carry forward regardless of broker: the equivalent failure exists in [[Redis Streams]] (MAXLEN trimming past the slowest group) and in SQS (14-day max retention, DLQ expiry).
- Fintech angle: retention-caused loss isn't an ops blemish, it's an audit finding. If an event stream feeds reconciliation or regulatory reporting, retention sizing is a compliance control, not a disk setting.
- Cost-shape summary for an infra recommendation: self-managed scales with instances + storage + engineer-hours; MSK with broker-hours + storage (+ per-GB written on Express) + client cross-AZ transfer (replication between brokers is not billed on MSK, but client traffic is — watch it, it's the sneaky line); Confluent with throughput units + retention. Fan-out multiplies egress on all three.
- If Kafka returns to your stack, the realistic default given team size is MSK (AWS-native IAM/VPC, brokers managed) with tiered storage enabled — Confluent's premium starts paying off when you'd otherwise run Connect clusters and Schema Registry yourselves.

## Interview angle

The 60-second version: *"Kafka's hard costs are operational. Self-managed means owning broker sizing — disk and network bound, write rate times replication factor — partition-count planning you can't easily undo, throttled rebalancing when topology changes, staged rolling upgrades, and monitoring URP, consumer lag, and ISR flapping. MSK removes the broker babysitting but leaves everything Kafka-shaped: your partitions, your configs, your clients, your lag. Confluent charges a premium for elastic capacity plus the ecosystem — schema registry, connectors, tiered storage. The invariant across all three: managed money buys away broker ops, never client-side correctness."*

Classic probes:

- "What do you monitor on a Kafka cluster?" — URP first, consumer lag (as time, not messages), ISR shrink rate, offline partitions, controller count, disk. Naming URP first signals you've operated it.
- "MSK vs Confluent vs self-managed?" — answer in terms of which failure classes each removes, and name the cost shape: instances+people vs broker-hours+transfer vs throughput-premium.
- "A consumer group is lagging badly — walk me through it." — is lag growing or draining? one partition (skew) or all (consumer capacity)? check rebalance frequency, downstream latency, then the retention clock — you're racing deletion.

## Check yourself

> [!question]- Q: Explain why "we moved to MSK" doesn't eliminate Kafka incidents. Which incident classes remain?
> MSK owns the broker layer: VM failures, patching, infrastructure. It has no control over topic/partition design, producer/consumer configs, or application behavior — so hot partitions, consumer-lag storms, rebalance churn, retention-vs-lag races, and schema breakage all remain. Those client-side classes cause most Kafka pages, which is why the incident count drops less than expected.

> [!question]- Q: What happens if consumer lag, measured in time, exceeds the topic's retention? How would you catch it before it bites?
> The broker deletes segments the consumer never read — permanent, errorless loss discovered only downstream (gaps in reconciliation). Catch it by alerting on lag expressed in *time* relative to retention (e.g., page at 50% of the window), and by sizing retention to survive your worst-case consumer outage plus recovery time, not to fit the disk.

> [!question]- Q: Why is adding a broker to a Kafka cluster not an instant capacity fix?
> A new broker starts empty and vanilla Kafka assigns it nothing automatically. You must run partition reassignment, which copies replica data over the network — throttled, or it starves live traffic. Until reassignment completes, the new broker carries no load. Contrast with stateless scale-out; this is the cost of [[Stateless vs Stateful Services|stateful]] scale-out.

> [!question]- Q: Mini-design — 20 MB/s sustained produce, RF=3, 7-day retention, 2 consumer groups. Rough the disk and network needs, labeling assumptions.
> Disk: 20 MB/s × 604,800 s ≈ 12.1 TB raw × RF 3 ≈ ~36 TB, plus ~35% headroom → ~50 TB cluster-wide (assumes post-compression rate is the 20 MB/s). Network: ingest 20 + inter-broker replication 40 (RF−1 = 2×) + consumer egress 40 (2 groups × 20) ≈ 100 MB/s cluster aggregate, before reassignment/recovery headroom — so per-broker NICs need comfortable margin above their share. All figures rough by construction; benchmark before buying.

## Sources

- Apache Kafka documentation — Operations section (monitoring, reassignment, upgrades)
- KIP-405 (Tiered Storage) and the Kafka 3.9 release notes
- AWS MSK documentation — shared responsibility, best practices, MSK Serverless
- Confluent Cloud documentation and pricing pages (elastic CKUs, tiered storage, managed connectors)
- Confluent blog / Apache docs on choosing partition counts (Jun Rao's classic post)
- LinkedIn Engineering — Cruise Control and Burrow project write-ups
- AWS Builders' Library — operational-readiness patterns (general, applies to broker fleets)
- Narkhede, Shapira, Palino — *Kafka: The Definitive Guide* (O'Reilly), chapters on reliability and monitoring
