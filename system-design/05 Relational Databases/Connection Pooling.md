---
title: Connection Pooling
tags: [system-design, databases]
type: note
status: evergreen
created: 2026-10-01
updated: 2026-10-03
summary: Why opening a database connection per request doesn't survive production, how to size a pool (small is usually faster), HikariCP defaults, PgBouncer/ProxySQL, and pool exhaustion as a classic outage shape.
related: ["[[How Databases Store Data]]", "[[Timeouts Retries and Exponential Backoff]]", "[[Load Shedding and Graceful Degradation]]"]
---

# Connection Pooling

A connection pool keeps a small set of long-lived database connections open and lends them to requests for the duration of a query or transaction. It exists because database connections are shockingly expensive to create and surprisingly expensive to merely *hold* — and because beyond a modest number, more connections make a database slower, not faster. Pooling is boring until the day the pool empties, at which point it becomes the whole incident.

## The problem that created it

A young service opens a connection per request: connect, query, close. It works in dev. Under production load, three walls appear in sequence.

First, **connection setup cost**: a TCP handshake, a TLS handshake, authentication, and server-side session setup — several network round trips plus real server work before the first byte of your actual query. For a 2 ms query, tens of milliseconds of ceremony.

Second, **per-connection server cost**: PostgreSQL forks an **operating-system process per connection**, each with its own memory (and `work_mem` multiplies per sort/hash operation within it). MySQL uses a thread per connection — lighter, but far from free. Either way the server spends RAM and scheduler effort just *keeping* connections, idle or not. Postgres ships with `max_connections = 100` by default for a reason.

Third — and least intuitive — **more concurrency makes the database slower**. A database machine has a fixed number of CPU cores and a fixed disk bandwidth. A few dozen active queries keep every core busy; thousands of active connections just multiply context switching, lock contention, and buffer-pool thrashing while throughput *drops*. The system got slower by being given more ways to be busy.

The fix inverts the model: open N connections once, reuse them forever, and make requests queue briefly for a connection instead of stampeding the database.

## What it solves

- **Amortizes setup**: handshake and auth paid once per connection lifetime, not per request.
- **Caps database-side concurrency**: the pool is a deliberate admission-control valve — the DB sees at most N sessions regardless of incoming traffic.
- **Smooths bursts**: short queue waits at the pool absorb spikes that would otherwise become DB overload.
- **Gives you one place to measure**: pool wait time is the cleanest early-warning signal that the database is the bottleneck.

## How it works

Mechanics in one paragraph: at startup the pool opens its connections. A request borrows one, runs its statements/transaction, and returns it (in JDBC, `close()` on a pooled connection *returns* it, not closes it). If all are busy, the request waits up to a timeout (`connectionTimeout`), then fails. The pool health-checks, retires connections past `maxLifetime` (defeating stale NAT/firewall state and letting the DB recycle memory), and replaces dead ones.

### Sizing: connections ≠ throughput

The counterintuitive core: **the correct pool size is small — usually around ten per instance, not hundreds.**

Throughput is bounded by what the database can *actively do in parallel*: roughly its core count while queries are CPU-bound, plus some allowance for time spent waiting on disk or network. HikariCP's "About Pool Sizing" wiki gives the classic starting formula, which it credits to the PostgreSQL wiki's "Number of Database Connections":

```
pool_size ≈ cores × 2 + effective_spindle_count
```

— i.e., for a 16-core database server with SSDs, think "a few dozen", not "a few thousand". The same wiki popularized an Oracle Real World Performance demonstration in which slashing a pool from thousands of connections to under a hundred *increased* throughput and collapsed latency: the queries stopped fighting each other.

What a bigger pool actually buys you above that point: nothing but memory burn and contention. Requests are better off waiting a few milliseconds in the pool queue (cheap, in your process) than "executing" on an oversubscribed database (expensive, everyone pays). This is queueing theory applied kindly: admit what the server can serve; queue the rest; shed what won't meet its deadline ([[Load Shedding and Graceful Degradation]]).

The fleet-level trap: pool size is **per instance**. 30 microservices × 3 replicas × 20 connections = 1,800 connections at a Postgres that comfortably runs 100–300. Kubernetes autoscaling multiplies this silently ([[Autoscaling]]). Budget: `instances × pool_size` per service, summed across services, must fit the database's connection budget with headroom for admins and failover churn.

### HikariCP defaults (the JVM standard, and Spring Boot's default pool)

| Setting | Default | Meaning |
|---|---|---|
| `maximumPoolSize` | **10** | total connections — the deliberate, small cap |
| `minimumIdle` | = max | i.e., a **fixed-size pool** by default (recommended) |
| `connectionTimeout` | 30 000 ms | max wait to borrow before Hikari throws `SQLTransientConnectionException` |
| `idleTimeout` | 600 000 ms | idle retirement (only when minimumIdle < max) |
| `maxLifetime` | 1 800 000 ms | retire every connection after 30 min |
| `leakDetectionThreshold` | 0 (off) | log a warning if a borrow is held longer than this — **turn this on** |

In Spring Boot terms, the overrides that matter most often:

```properties
spring.datasource.hikari.maximum-pool-size=10
spring.datasource.hikari.connection-timeout=3000
spring.datasource.hikari.max-lifetime=900000
spring.datasource.hikari.leak-detection-threshold=10000
```

Two defaults worth overriding consciously: `connectionTimeout` of 30 s is far too long for a user-facing request path (fail in 1–3 s and degrade; see [[Timeouts Retries and Exponential Backoff]]), and `maxLifetime` should be set *below* any network/infrastructure idle cutoff between you and the DB (cloud LBs and firewalls silently kill long-lived TCP flows).

### External poolers: PgBouncer and ProxySQL

A client-side pool bounds *one process*. When many services (or many instance replicas, or serverless functions) share one database, you also want a server-side pooler in front of it:

- **PgBouncer** (PostgreSQL): thousands of client connections multiplexed onto a few dozen real backend connections. Modes: **session** (one backend per client session — safe, least sharing), **transaction** (backend lent per transaction — the high-leverage mode), **statement** (per statement — most aggressive, breaks multi-statement transactions). Transaction mode historically breaks session state — session-level prepared statements, `SET` parameters, advisory locks; PgBouncer added protocol-level prepared-statement support in v1.21 (2023), which removed the biggest sharp edge, but session-state assumptions must still be audited.
- **ProxySQL** (MySQL): connection multiplexing plus query routing (e.g. reads to replicas), query caching and firewalling.
- Managed equivalents exist (e.g. AWS RDS Proxy) with the same conceptual role.

The two layers compose: HikariCP in the service (bounds the service, amortizes handshakes) → PgBouncer at the DB (bounds the fleet).

## Trade-offs

| You gain | You pay |
|---|---|
| Handshake cost amortized to ~zero | Pool config is now a correctness surface (timeouts, lifetimes, sizing) |
| DB protected from concurrency collapse | Requests can queue/fail at the pool — a new failure mode to handle |
| Clean saturation signal (pool wait time) | Shared mutable session state risk if anything leaks (`SET`, temp tables) |
| Fleet-level multiplexing via PgBouncer | Transaction-mode poolers constrain session features; one more hop and component to operate |

### The four numbers to watch

A pool is observable with four metrics; alert on the first two:

- **Pool wait time** (time to borrow) — the earliest saturation signal; rises before latency dashboards notice.
- **Timeouts at the pool** (`connectionTimeout` exceeded) — users are already failing.
- **Active borrows vs max** — sustained at max means the pool is the constraint (or a leak is in progress).
- **Borrow hold time** — Little's law input; a creeping hold time predicts exhaustion before it happens.

## Failure modes

- **Pool exhaustion — the classic outage.** One slow dependency inside a transaction (a remote call, a lock wait, a slow query) makes every borrow last seconds instead of milliseconds. Little's law does the rest: at 200 req/s × 5 s hold time you'd need 1,000 connections; you have 10. Every request now waits `connectionTimeout` then fails; the service is "down" while the database sits nearly idle. Detection: pool-wait-time and active-borrows metrics (Hikari exposes them via Micrometer); the tell is *timeouts at the pool with a calm database*. Mitigations: never do remote I/O inside a transaction ([[N+1 Queries and ORM Pitfalls]]), short `connectionTimeout`, statement timeouts in the DB, leak detection on.
- **Connection leaks**: a code path that borrows and never returns (exception path skipping close, manual `getConnection()` without try-with-resources). The pool drains over hours — a slow-motion exhaustion. `leakDetectionThreshold` names the culprit stack trace.
- **Fleet oversubscription**: deploys/autoscale add replicas until `SUM(pools) > max_connections`; new pods crash-loop on "too many connections" — often *during* an incident, when you scale out in panic. This failure punishes the rescue attempt.
- **Stale connections after failover**: the pool holds TCP connections to a database that just died; every borrow hands out a corpse until validation/retirement catches up. Keep `maxLifetime` sane and validation on; expect a burst of errors at [[Failover]].
- **Thundering reconnect**: all connections retire or die simultaneously (pooler restart, DB restart) and every instance reconnects at once — a reconnection stampede cousin of [[Cache Stampede and Thundering Herd]]. Jitter on `maxLifetime` (Hikari applies some automatically) and in reconnect logic.

## When NOT to use it

- Rarely "never pool" — but **don't client-pool per invocation** in environments where processes are ephemeral (serverless functions): each cold instance brings its own pool and the fleet math explodes. That's exactly the case for an external pooler/proxy instead.
- Don't bother for a single-user CLI tool, batch job, or migration script — one connection, used start to finish, is the pool.
- Don't use pooling as a *substitute* for fixing held-too-long connections: a pool in front of 5-second transactions is a 5-second-latency service with extra steps.

## In your stack

- Spring Boot services get HikariCP by default — which means every service has these exact defaults unless someone changed them. An audit of `connectionTimeout`, `maxLifetime` vs infra idle timeouts, and pool metrics exposure is cheap and usually overdue.
- The same discipline applies to the MongoDB Java driver (`maxPoolSize` default 100, `waitQueueTimeoutMS`) and your Redis clients (Lettuce multiplexes on few connections; Jedis pools) — every datastore client in the stack has a pool with defaults someone chose for a different company.
- Services pinned at 1 replica make per-instance sizing easier (one pool per service) but concentrate the blast radius: one leaked-connection bug = that whole capability down. Leak detection and pool alarms matter *more* at 1 replica, not less.
- Remote calls inside `@Transactional` methods are the Niyo-shaped version of the exhaustion outage: a slow partner-bank API inside a transaction holds a connection for the full call. Keep transactions datastore-only; publish via [[The Outbox Pattern]] rather than calling brokers in-transaction.
- Kong/edge rate limits ([[Rate Limiting]]) are your upstream pressure valve: it is better to reject at the edge than to queue into an exhausted pool.

## Interview angle

The 60-second version: *"Connections are expensive to make — TCP, TLS, auth — and expensive to hold: Postgres spends a process per connection. Worse, databases have finite parallel capacity, so beyond roughly cores-times-two active sessions, extra concurrency reduces throughput. So you keep a small fixed pool — HikariCP defaults to ten — borrow per transaction, and let requests queue briefly. Sizing is about the database's capacity, not the app's traffic; the famous result is that shrinking pools from thousands to under a hundred raises throughput. At fleet scale you add PgBouncer in transaction mode so thousands of client connections share tens of backend ones. The classic outage is exhaustion: one slow call inside a transaction holds borrows for seconds and the service dies at the pool with the DB idle."*

Classic probes: "why does a small pool outperform a big one?" (fixed parallel capacity + context-switch/contention tax); "what's Little's law got to do with sizing?" (needed connections = arrival rate × hold time); "session vs transaction pooling in PgBouncer?"; "how do you detect exhaustion vs a slow database?" (pool wait high + DB metrics calm vs both hot).

## Check yourself

> [!question]- Q: Explain to a new engineer why giving the service 500 connections instead of 10 made p99 latency worse.
> The database can only actively execute roughly as many queries in parallel as it has cores plus some I/O-wait allowance. At 500 connections the same work arrives as 500 concurrent sessions: context switching, lock contention and buffer-pool competition rise, so every query takes longer — queueing moved *inside* the database where it's expensive, instead of at the pool where it's cheap. A 10-connection pool completes the same queries faster and the brief pool wait is smaller than the slowdown it prevents.

> [!question]- Q: What happens if a partner API inside a @Transactional method starts taking 8 seconds? Walk the cascade.
> Each request now holds a pooled connection ~8 s. By Little's law, even 2 req/s needs 16 connections — the 10-slot pool saturates. New requests wait `connectionTimeout` (30 s default) then throw; upstream callers time out sooner and retry, adding load ([[Failure Modes in Distributed Systems]] retry amplification); the service appears down while the database is idle. Fix afterwards: move the remote call out of the transaction, cut `connectionTimeout` to seconds, add a client timeout on the partner call shorter than the pool timeout, alert on pool wait time.

> [!question]- Q: Your Postgres shows "FATAL: sorry, too many clients already" during a scale-out event. Why did adding pods break the database, and what are the two structural fixes?
> Each new pod brings its own full pool; total fleet connections crossed `max_connections`. Fix one: budget — size per-instance pools so `services × replicas × pool_size` fits the DB with headroom, enforced as config review. Fix two: structural — put PgBouncer (or RDS Proxy) in front so client connections multiplex onto a bounded backend set, decoupling fleet size from DB session count.

> [!question]- Q: Mini design: sizing for a 16-core Postgres serving 3 services × 2 replicas, p95 query 10 ms, peak 600 req/s total. Sketch the arithmetic.
> Needed concurrent connections ≈ arrival × hold = 600/s × 0.010 s = 6 active on average; allow for bursts and transactions spanning a few queries → per-instance pools of 8–10 give 48–60 total, comfortably under a 16-core box's healthy range (formula ≈ 16 × 2 = 32 *active*; pools slightly above that are fine since not all are active simultaneously) and under max_connections with headroom. If p95 hold time ever grows 10x, the math breaks — which is the argument for alerting on pool wait, not just query time.

## Sources

- HikariCP wiki: "About Pool Sizing" (the formula — credited there to the PostgreSQL wiki's "Number of Database Connections" — and the Oracle Real World Performance demonstration) and "Configuration" (defaults quoted above).
- PostgreSQL documentation: "Connections and Authentication" (`max_connections`, per-backend process model).
- PgBouncer documentation: pooling modes and feature matrix; release notes for v1.21 prepared-statement support.
- ProxySQL documentation: multiplexing and query routing.
- *Designing Data-Intensive Applications*, Kleppmann — Chapter 1's load-parameters framing applies directly to "connections are not throughput".
