# LinkedIn Posts — Publication-Ready Drafts

> **Goal**: 4 polished LinkedIn posts you can publish as-is (or with minor tweaks). Each post is calibrated to LinkedIn's algorithm: punchy hook, scannable body, one closing question, 3-5 hashtags.
>
> **Cadence**: 1 post per week while job-searching. Don't dump all 4 at once — space them out 7-10 days apart.
>
> **Posting time**: Tuesday or Wednesday, 9-10 AM IST (highest engagement window for tech LinkedIn in India).

---

## Pre-publish Checklist

Before hitting "Post" on any of these:

- [ ] Replace any `<placeholder>` with the actual specific (numbers, service names — though most posts don't have placeholders)
- [ ] Verify nothing reveals confidential business detail (customer counts, internal architecture diagrams, etc.)
- [ ] Read aloud — if a sentence trips you up, rewrite it
- [ ] Check the first 2 lines fit "above the fold" on mobile (LinkedIn truncates after ~200 chars; the hook must land in the visible portion)
- [ ] Add a personal touch line if it feels too corporate (your voice matters)

---

## Post 1: Kafka Consumer Rebalancing War Story

**Topic**: Production debugging — Kafka consumer rebalancing on every deployment.
**Length**: ~1,100 characters
**Best for**: Showing production debugging chops + Kafka depth

```
Our Kafka consumers rebalanced on every single deployment.

Every. Single. One.

For weeks we accepted it as "just how Kafka works." Then we got tired
of the 30-second message lag after each deploy and went hunting.

What we found:

→ Our session.timeout.ms was 10s
→ Our K8s rolling deploy took ~12s per pod
→ Consumer group coordinator was timing out before the new pod was ready
→ Every pod replacement triggered a full group rebalance
→ Result: 30-60s of stalled processing on every release

The fix wasn't glamorous:

1. Bumped session.timeout.ms to 45s
2. Bumped max.poll.interval.ms to 300s (we had long-running tasks)
3. Added a JVM shutdown hook to commit offsets + leave the group cleanly
4. Set heartbeat.interval.ms to 1/3 of session timeout (15s)

Rebalance time dropped from ~30s to under 5s. Lag stopped.

The lesson: most "Kafka is flaky" problems are actually consumer config
problems. The defaults are fine for development. They're not fine for
production rolling deploys.

What's the most boring config change that saved your team's week?

#Kafka #DistributedSystems #BackendEngineering #SRE
```

---

## Post 2: Spring Boot 2→3 — What Nobody Talks About

**Topic**: The hidden cost of Spring Boot 2 → 3 migration.
**Length**: ~1,200 characters
**Best for**: Showing migration ownership + Java depth

```
"It's just javax → jakarta. How hard could it be?"

Famous last words.

We migrated 3 production services from Spring Boot 2 to 3 last quarter.
The official migration guide makes it sound clean. Reality was 5 issues
nobody documents:

1. Hibernate 6 changed default ID generation strategy.
   Sequences that used to auto-increment by 1 started jumping by 50.
   Caused order-ID collisions in dev/staging. Easy fix once you find it.

2. Spring Security 6 removed the WebSecurityConfigurerAdapter approach.
   All our auth filters needed a rewrite to the new lambda-based DSL.

3. Datadog APM agent compatibility broke until we upgraded the agent too.
   Two days of "where did all my traces go?"

4. Bean cycle detection became strict by default.
   Circular dependencies that worked in SB2 now refused to start.
   Real bug or not, you have to fix every single one.

5. Some Jackson deserialization defaults changed.
   Date/time formatting in API responses subtly shifted. Caught it in
   integration tests, would have shipped if we'd skipped them.

Zero downtime on all three migrations. But the prep was 70% of the work.

Migration takeaway: integration tests > unit tests. Always. The bugs hide
in glue code.

Anyone else hit issues the guide doesn't mention?

#Java #SpringBoot #SoftwareEngineering #Migration
```

---

## Post 3: Building a Banking Microservice Solo — 544 Commits Later

**Topic**: Owning an end-to-end service as a single developer.
**Length**: ~1,150 characters
**Best for**: Showing ownership + architecture thinking (key SDE2 signal)

```
I built an entire production banking microservice as the only engineer.

544 commits later, here's what changed how I work.

The service handles agent onboarding for a partner bank — login/OTP,
background verification with face match, risk scoring, business approvals,
event tracking. About 8 months from "nothing" to "in production."

Three things I'd do differently if I started again:

1. Multi-channel API architecture from day one.
   I bolted on /ag (agent), /app (mobile), and /wb (web backoffice)
   midway through. Should have designed for it from the first PR.
   Refactor cost: about 60 hours I won't get back.

2. RBAC as a first-class concern, not an afterthought.
   I added masked PAN handling and role checks after the BGV system
   was already shipping. Threading it through retroactively was painful.

3. Event-driven activity tracking from PR #1.
   We added Kafka activity events around PR #200. The historical gap
   (no events for the first 5 months of usage) became a real compliance
   conversation. Wire it in early — even if the consumer doesn't exist
   yet, the topic should.

The biggest unlock as a solo dev: stop being defensive about your code.
PRs reviewed by yourself are the same as no review. I started inviting
engineers from sister teams for second eyes — slower, but the bugs they
caught saved me from production incidents twice.

Solo ownership isn't a flex. It's a forcing function for taste.

What's the one thing you'd build differently if you started your current
service from scratch?

#BackendEngineering #SoftwareArchitecture #Microservices #Fintech
```

---

## Post 4: From Confluent to MSK — A ZSTD-Compressed Story

**Topic**: Kafka infrastructure migration.
**Length**: ~1,100 characters
**Best for**: Showing infra ownership + cost-awareness (hiring manager catnip)

```
We moved 10+ services from Confluent Cloud to AWS MSK.

The cost savings were obvious. The gotchas weren't.

Easy wins:

→ ZSTD compression cut storage and network by ~40% vs Snappy
→ Direct VPC peering removed cross-cloud egress costs
→ Single AWS bill instead of two vendor relationships

The gotchas nobody warned us about:

1. Connection limits matter at scale.
   MSK brokers have hard ulimit ceilings. We hit them when all our
   pods came up at once after a cluster upgrade. Lesson: stagger pod
   warm-up and tune broker file descriptors before you need to.

2. Schema registry migration is its own project.
   We were using Confluent's hosted registry. Migrating subjects, IDs,
   and compatibility settings to a self-hosted alternative took 2 weeks
   of careful work. Don't underestimate it.

3. Acks and idempotence configs need re-verification.
   Defaults differ subtly between Confluent Cloud and MSK. We had an
   "exactly once" pipeline that quietly became "at least once" after
   migration. Caught it in load testing.

4. Monitoring isn't free.
   Confluent Cloud bundles dashboards. With MSK you wire up CloudWatch +
   Prometheus + JMX exporters yourself. Budget a week.

End result: 35% lower Kafka bill, deeper control of the cluster, and
significantly better observability than we had before. Worth it. But not
a weekend project.

For anyone planning a similar move — what's your biggest gotcha?

#Kafka #AWS #DataInfrastructure #BackendEngineering #Fintech
```

---

## Bonus: 4 Short Comment Templates (For Engagement on Others' Posts)

LinkedIn rewards engagement, not just posting. **5 thoughtful comments per week** does as much for your visibility as 1 original post.

### Template 1 — Add a specific lesson
> "We hit this exact issue with our Kafka MSK migration. The fix for us was bumping session.timeout to 45s and adding a clean shutdown hook. Game-changer once it clicked."

### Template 2 — Counterpoint with evidence
> "Interesting take. In our fintech context we actually went the opposite direction — we kept it as a monolith for the auth domain and split everything else into services. Compliance audits became way easier with auth centralized."

### Template 3 — Question that invites depth
> "Curious — at what scale did you start feeling the pain that pushed you toward this design? We're around 50 QPS and still in monolith mode for similar reasons."

### Template 4 — Validation + your story
> "This matches our experience exactly. We saw a 40% drop in storage costs after switching to ZSTD. The migration itself was a weekend's work, but the savings compound monthly."

> **Why these work**: they show specific, concrete experience without being braggy. Avoid "great post!" / "thanks for sharing" — algorithms and humans both ignore those.

---

## Post Performance Tracking (Optional)

If you want to optimize, track:

| Post | Date | Impressions | Reactions | Comments | Top recruiter view? |
|---|---|---|---|---|---|
| #1 Kafka rebalancing | | | | | |
| #2 Spring Boot 2→3 | | | | | |
| #3 Solo service | | | | | |
| #4 MSK migration | | | | | |

Re-use the format of whichever one performs best.

---

*Source: hooks from `02_linkedin_content-refined.md` Section 6 expanded into full drafts.*
