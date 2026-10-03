---
title: The Fallacies of Distributed Computing
tags: [system-design, distributed-theory]
type: note
status: evergreen
created: 2026-10-01
updated: 2026-10-02
summary: The eight false assumptions engineers make about networks, each paired with a modern production failure it causes.
related: ["[[Network Partitions and Timeouts]]", "[[CAP Theorem]]", "[[Timeouts Retries and Exponential Backoff]]", "[[Failure Modes in Distributed Systems]]"]
---

# The Fallacies of Distributed Computing

Every note in this module — [[CAP Theorem]], [[Consensus and Raft]], [[Idempotency]] — exists because a network call is not a method call. In the 1990s, engineers at Sun Microsystems (Peter Deutsch, with the eighth item attributed to James Gosling) catalogued the assumptions that programmers silently carry over from single-machine code into distributed code. Every one of them is false, and every one of them still takes down systems today. Read this note first: the rest of the module is the theory of what to do once you stop believing these eight things.

## Why this matters

On one machine, calling a function either runs it or throws. Across a network, there is a **third outcome: you don't know**. The request may have been lost, the response may have been lost, or the other side may be alive but slow. Distributed systems theory is largely the study of that third outcome. If you design as if any of the eight fallacies were true, your system works in the demo, works in staging, and fails in production at 2 a.m. during a network blip — which is exactly when you can least afford to debug it.

The fallacies are also the fastest sanity check in a design review: for each arrow in an architecture diagram, ask "what happens when this arrow is slow, lossy, or severed?" If the answer is "it can't be", a fallacy is hiding there.

## How it works

The eight fallacies, each with the modern failure it produces.

### 1. The network is reliable

Packets are dropped, switches reboot, cables get cut, cloud availability zones lose connectivity to each other. **Modern failure:** a payment service calls a partner bank API, the response is lost after the bank processed the debit, the service retries, and the customer is charged twice. The fix is not "a better network" — it is [[Idempotency]] and designing for [[Network Partitions and Timeouts]]. AWS, GCP and Azure all publish post-mortems of intra-region network partitions; they are routine events, not black swans.

### 2. Latency is zero

A call across the network costs milliseconds where a local call costs nanoseconds — four to six orders of magnitude (see [[Latency Numbers Every Engineer Should Know]]). **Modern failure:** a microservice migration turns one in-process call into a chain of 12 synchronous HTTP hops; p99 latency balloons because latencies add up along the chain and the slowest hop dominates. This is the core argument in [[Communication Patterns Between Services]] and a big reason [[Why Async Messaging Exists]].

### 3. Bandwidth is infinite

Links saturate. **Modern failure:** a service ships full entity snapshots on every change event; at 10x traffic the Kafka cluster's inter-broker replication saturates the NICs, replication lag grows, and consumers fall minutes behind. Pagination, compression, and sending deltas instead of snapshots all exist because of this fallacy.

### 4. The network is secure

Anything on the wire can be read or altered unless you prevent it. **Modern failure:** "it's internal traffic, we don't need TLS" — then a compromised pod in the same cluster sniffs service-to-service traffic carrying PAN or KYC data. This fallacy funds the entire contents of [[TLS and HTTPS]], [[API Security Essentials]] and mTLS in [[Service Mesh]]. Zero-trust networking is the industry formally renouncing fallacy 4.

### 5. Topology doesn't change

Machines move, scale out, scale in, get replaced. **Modern failure:** a client caches a resolved IP address of a service forever; Kubernetes reschedules the pod, the IP is reassigned, and the client sends traffic into the void (or to a different service). This is why [[Health Checks and Service Discovery]] exists, why DNS TTLs matter ([[How DNS Works]]), and why [[Consistent Hashing]] was invented — membership change is the normal case, not the exception.

### 6. There is one administrator

Different teams, vendors, and cloud providers each control a piece of the path. **Modern failure:** your service is correct, but a partner's API gateway starts returning 429s because *their* ops team tightened a quota; or a corporate proxy between you and a SaaS dependency silently strips a header. You cannot fix what you do not administer — you can only detect it ([[Logs Metrics and Traces]]) and degrade gracefully ([[Load Shedding and Graceful Degradation]]).

### 7. Transport cost is zero

Serialization, TLS handshakes, connection establishment, and cross-AZ/cross-region data transfer all cost CPU and money. **Modern failure:** a chatty service pair deployed in different availability zones generates a five-figure monthly cross-AZ data transfer bill — the system is *correct* and the invoice is still an incident. Connection pooling ([[Connection Pooling]]), keep-alive, and batching exist because of this fallacy.

### 8. The network is homogeneous

Different links have different latency, loss, and MTU characteristics; clients arrive over everything from fiber to 2G. **Modern failure:** an app tested on office Wi-Fi ships a 4 MB payload per screen and is unusable on a congested or weak mobile link — which is how a large share of users outside metros reach you. Protocol choices ([[HTTP 1.1 vs HTTP 2 vs HTTP 3]], QUIC's design) and [[CDN]] placement are responses to heterogeneity.

### The list at a glance

| # | False assumption | Typical production failure | First-line countermeasure |
|---|---|---|---|
| 1 | The network is reliable | Lost requests and acks, ambiguous outcomes, duplicates on retry | Timeouts, retries with backoff, [[Idempotency]]; queues for async paths |
| 2 | Latency is zero | Deep synchronous call chains, p99 blow-ups | Latency budgets per hop, fewer hops, caching ([[Why Caching Works]]), async |
| 3 | Bandwidth is infinite | Saturated links, replication lag, bloated payloads | Pagination, compression, deltas, [[Capacity Planning]] |
| 4 | The network is secure | Sniffed or tampered internal traffic | TLS/mTLS everywhere, signed payloads ([[API Security Essentials]]) |
| 5 | Topology doesn't change | Stale cached IPs, broken connection pinning | [[Health Checks and Service Discovery]], DNS TTL discipline, [[Consistent Hashing]] |
| 6 | There is one administrator | Partner quotas, middlebox surprises, conflicting changes | Explicit contracts and SLOs, defensive clients, [[Logs Metrics and Traces]] |
| 7 | Transport cost is zero | CPU burned on serialization, cross-AZ transfer bills | Connection reuse, batching, locality-aware placement |
| 8 | The network is homogeneous | Works on office Wi-Fi, unusable on 2G | Test on worst-case links, adaptive payloads, [[CDN]] |

### The ninth fallacy: the network is either up or down

The original list is from the 1990s and holds up remarkably well, but one assumption it does not name explicitly has become the dominant failure mode of modern systems: **that a dependency is either working or failed.** The expensive case is neither. A node that is slow, a link that drops 2% of packets, a disk that services reads in 400 ms instead of 4 — these are **grey failures**, and they are worse than clean failures for a specific reason: every mechanism you built to handle failure is triggered by *absence*, and a grey failure produces presence.

Walk through what happens. Health checks pass, because the node answers — eventually. The load balancer keeps sending it traffic, because it is "healthy". Clients time out and retry, so the degraded node now receives *more* load than its healthy peers. Threads pile up waiting on it, and the pool exhausts; callers of those callers time out in turn. Within minutes the slow node has taken down services that do not depend on it, because they shared a thread pool or a connection pool with something that did ([[Bulkheads and Isolation]]).

This is also the territory of **metastable failure**: a system that was stable under load enters a state where its own retry and recovery behaviour sustains the overload, so the system stays down even after the original trigger is gone. Restarting things helps only if it breaks the feedback loop; otherwise the loop resumes. The defences are the ones that treat slow as a kind of failed — timeouts tight enough to free resources, circuit breakers that stop sending work to a degraded dependency, load shedding that refuses work rather than queueing it, and bounded retries with jitter ([[Circuit Breakers]], [[Load Shedding and Graceful Degradation]], [[Failure Modes in Distributed Systems]]).

The design-review version of this fallacy is one question: **"what does this call do when the dependency answers in 30 seconds instead of 30 milliseconds?"** If the answer involves an unbounded wait, an unbounded queue, or a shared pool, you have found the next incident.

### What this looks like at 3am

The page says one service is down. You open its dashboard and it is slow but alive. You open the dashboard of the service it calls, and that one is fine — latency normal, error rate zero. Nothing is obviously broken anywhere, and yet the top of the stack is failing.

That pattern — **failure at the top, health at every individual layer** — is fallacy 1 and 2 compounding. Somewhere down the chain, something is marginally slower than usual. Each hop's timeout is generous, so each hop waits; waits stack along the chain; the hop at the top exceeds its own timeout and retries; the retries double the load on the thing that was already slow. No single component is misbehaving by its own metrics, which is why asking "which service is broken?" produces no answer.

What actually locates it: the correlation ID. Pull one failing request and read its full trace end to end — where did the time go, which hop's duration grew, and did the same logical operation appear more than once because something retried it ([[Distributed Tracing]])? Two numbers decide the next move. If one hop's latency grew and the others merely waited, you have the culprit. If the request appears three times in the trace, retry amplification is now part of the problem and reducing retries will help more than scaling anything. The thing not to do is restart services top-down: it clears the symptom, the loop re-forms, and you have destroyed the evidence.

### Using the list in a design review

- Walk every arrow in the diagram against fallacies 1 and 2 first — they cause the majority of incidents: "what is the timeout, what happens on retry, is the operation idempotent?"
- For every cache of an address, name, or location (IPs, DNS, pod endpoints), ask what invalidates it — that is fallacy 5.
- For every dependency crossing a team or company boundary, ask who you page when it misbehaves — fallacy 6 has no technical fix, only contracts and detection.
- Price the chatty paths: payload size × call rate × cross-AZ rate — fallacies 3 and 7 show up on invoices before they show up in alerts.
- Ask the grey-failure question on every arrow too, not just the failure question: a dependency that is *slow* exercises completely different code paths than one that is *down*, and the slow path is the one nobody tested.
- Name an owner for each arrow that crosses a team boundary — not a service, a person or rota. Fallacy 6's only mitigation is knowing who to call, and discovering that during an incident costs more than any technical gap.
- [[Chaos Engineering]] is the empirical version of this review: inject latency and partitions and see which fallacy your system still believes. Injecting *latency* is more informative than injecting failure, for exactly the reason above.

A compact way to remember the whole list: fallacies 1 and 5 are about **things changing without telling you**; 2, 3 and 7 are about **costs you forgot to count**; 4 and 6 are about **control you do not have**; and 8 is about **users who are not you**. Four categories is easier to carry into a review than eight items.

## Common misconceptions

- **"The cloud fixed this."** Cloud providers made individual fallacies *less frequent*, not false. Managed networks still partition; managed services still throttle you (fallacy 6); cross-AZ traffic still costs money (fallacy 7).
- **"Retries fix fallacy 1."** Retries convert "maybe it failed" into "maybe it happened twice". Retries are only safe on top of [[Idempotency]], and naive retries create retry storms ([[Timeouts Retries and Exponential Backoff]]).
- **"These are about the network layer, so they're the platform team's problem."** Every fallacy surfaces as an *application* bug: double charges, stale caches, zombie leaders. The application owns the consequences.
- **"Fallacy 2 is solved by faster hardware."** The speed of light sets a floor: Mumbai to Virginia and back is roughly 180–250 ms no matter how good the hardware is. Latency budgets are physics, not engineering debt.
- **"A timeout means the call failed."** It means you stopped waiting. The work may have completed, may be running now, or may never have started — and the difference matters most where money moves. Treating a timeout as a failure and acting on that belief is how duplicate debits happen; the correct handling is to treat it as *unknown* and resolve it by query or reconciliation ([[Idempotency]]).
- **"We've never had a partition, so we're fine."** You have had many and resolved them without noticing, because the retry succeeded. The fallacies describe events whose frequency is high and whose visibility is low — which is exactly the combination that makes a system look robust right up until the retry does not succeed.

## In your stack

- Every synchronous Kong → service → MongoDB → partner-bank chain embeds fallacies 1 and 2. The `x-correlation-id` header on every inter-service request exists so that when a hop fails ambiguously, you can reconstruct what actually happened from logs.
- Your messaging journey (RabbitMQ → Kafka → SQS → Redis Streams) is a decade of responses to fallacy 1: queues turn "the callee must be up right now" into "the callee must be up eventually" — see [[Why Async Messaging Exists]].
- Services pinned at 1 replica make fallacy 5 sharper: a single pod reschedule is a visible outage window for that service, not a load-balancer blip. Be honest about that in designs that assume "the service is always reachable".
- In a money-movement flow, fallacy 1 is a compliance issue, not just a technical one: an ambiguous debit must fail closed and reconcile, never guess. That is why [[Idempotency]] is the most load-bearing note in this module for your work.
- Fallacy 6 has a name in your architecture: the partner bank. You do not administer their uptime, their rate limits, their maintenance windows, or their definition of a timeout — so the only things you control are your own timeout, your own retry policy, and whether an ambiguous outcome has somewhere safe to land. Every partner integration should have an explicit answer to "what state does a transaction sit in while we do not know?", and that state needs a sweeper, not a human.
- Fallacy 5 is sharper on EKS than on fixed infrastructure, because pods are *expected* to move. Anything that caches an address — a resolved IP, a connection held open across a reschedule, a client that resolved DNS once at startup — is a bug waiting for the next deploy. Reuse the shared HTTP client configuration rather than hand-rolling one, since connection and DNS lifetimes are exactly the settings easy to get wrong in a new client.
- Fallacy 7 is the one your invoice notices before your alerts do. Cross-AZ traffic between a chatty service pair, large payloads on high-frequency calls, and per-request TLS handshakes from a client without keep-alive are all correct systems that cost real money ([[Capacity Planning]]). Worth pricing once per design, in the same breath as latency.

## Interview angle

The fallacies rarely appear as a direct question; they appear as the *grading rubric*. When an interviewer asks "what happens if this call fails?" or "what if the response is lost?", they are checking whether you believe fallacy 1. A strong candidate volunteers failure handling before being asked.

The 60-second version: "There are eight classic false assumptions about networks — reliable, zero latency, infinite bandwidth, secure, static topology, one admin, free transport, homogeneous. The master key is the first one: a remote call has three outcomes, not two — success, failure, and *unknown*. Everything in distributed systems design — timeouts, retries, idempotency, consensus — is machinery for living with 'unknown'."

Classic follow-ups: "Which fallacy bites microservice migrations hardest?" (latency is zero — call-chain depth), and "How do you make retries safe?" (→ [[Idempotency]]).

## Check yourself

> [!question]- Q: Explain why a remote call has three outcomes instead of two, and why the third is the hard one.
> A local call either returns or throws — the caller always learns the outcome. A remote call can also end in *no answer*: the request or the response may have been lost, or the callee may be slow. The caller cannot distinguish "never executed" from "executed, response lost". It's hard because any action you take next — retry, give up, alert a human — risks being wrong in one of those two worlds. Idempotency makes retrying safe in both worlds, which is why it's the standard answer.

> [!question]- Q: Scenario — your service calls a bank API with a 3-second timeout. The call times out. The product manager says "just retry". What do you say?
> A timeout does not mean the debit didn't happen; it means we don't know. Retrying blindly risks a double charge. Safe options: (1) send the retry with the same idempotency key so the bank deduplicates it, (2) query the bank for the status of the original attempt before retrying, or (3) park the transaction in a reconciliation state and resolve it asynchronously. In a regulated money flow, fail closed and reconcile — never guess.

> [!question]- Q: Which fallacies does a move from a monolith to microservices make worse, and why?
> Mainly 1, 2 and 7. Calls that were in-process function invocations (reliable, nanoseconds, free) become network calls (lossy, milliseconds, costing serialization + transfer). Fallacy 5 also worsens: more moving parts means topology changes more often. This is why [[Monolith vs Microservices]] treats the network boundary as the main cost of the migration.

> [!question]- Q: Scenario — a downstream service is responding in 8 seconds instead of 80 milliseconds, but returning correct results. Your service's client timeout is 30 seconds, the thread pool holds 200 threads, and traffic is 100 requests per second. Work out what happens, and name the three changes that prevent it.
> Start with the arithmetic, because it settles the argument faster than any principle. At 100 requests per second with an 8-second response, the number of requests in flight at any moment is 100 × 8 = 800 — four times the 200 threads available. The pool exhausts within a couple of seconds, after which new requests queue or are rejected, and callers of *your* service start timing out even for endpoints that never touch the slow dependency. If those callers retry, offered load rises and the dependency gets slower, which lengthens the queue further: the system is now sustaining its own overload and will not recover when the original slowness passes — metastable failure. Note that no alarm fired on the dependency: it returned correct results with a zero error rate the entire time. Three changes, each addressing a different link. (1) *Timeout sized to the dependency's real behaviour*, not to a round number — a 30-second timeout on an 80-millisecond call is not patience, it is a commitment to hold a thread for 30 seconds during a failure; at a 1-second timeout the in-flight count is 100 and the pool holds. (2) *A circuit breaker*, so that after a run of slow calls the client stops calling and fails fast, releasing resources and letting the dependency recover instead of being held under load ([[Circuit Breakers]]). (3) *Bulkheading*, so the slow dependency has its own bounded pool and cannot consume the capacity that unrelated endpoints need ([[Bulkheads and Isolation]]). Add bounded, jittered retries to taste — but note that retries are the amplifier here, so the correct instinct under grey failure is fewer retries, not more.

> [!question]- Q: Mini design prompt — you're adding a webhook callback from your system to merchant systems. Name three fallacies that apply and one mitigation for each.
> Fallacy 1 (their endpoint will be down sometimes): retry with exponential backoff and a dead-letter store — see [[Dead Letter Queues and Poison Messages]]. Fallacy 6 (you don't administer their side): publish a delivery contract — timeout, retry schedule, signature scheme — and expose a redelivery API instead of promising delivery. Fallacy 4 (the network is not secure): sign webhook payloads (HMAC) so merchants can verify authenticity — see [[API Security Essentials]]. Bonus: require merchant endpoints to be idempotent, because your retries will produce duplicates.

## Sources

- L. Peter Deutsch (Sun Microsystems), "The Eight Fallacies of Distributed Computing" — the original list (1994; eighth fallacy attributed to James Gosling, ~1997).
- Arnon Rotem-Gal-Oz, "Fallacies of Distributed Computing Explained" — the most-cited walkthrough paper.
- Martin Kleppmann, *Designing Data-Intensive Applications* (DDIA), Chapter 8 "The Trouble with Distributed Systems" — faults, partial failure, unreliable networks.
- AWS Builders' Library, "Timeouts, retries, and backoff with jitter" and "Challenges with distributed systems" — fallacies 1 and 2 in production practice.
- Bronson et al., "Metastable Failures in Distributed Systems", HotOS 2021 — why systems stay down after the trigger is gone.
- Huang et al., "Gray Failure: The Achilles' Heel of Cloud-Scale Systems", HotOS 2017 — the slow-not-dead failure mode and why monitoring misses it.
