---
title: The System Design Interview Framework
tags: [system-design, interview]
type: note
status: evergreen
created: 2026-10-01
updated: 2026-10-14
summary: The 4-phase method for running a 45-minute system design interview — time budgets, what interviewers actually grade, the classic failure patterns, and how to drill with this vault.
related: ["[[Back of the Envelope Estimation]]", "[[Design a URL Shortener]]", "[[What Is System Design]]"]
---

# The System Design Interview Framework

A system design interview is not a quiz with a hidden correct answer — it is a 45-minute simulation of working with you on an ambiguous problem. The interviewer is grading *how you drive*: whether you scope before you build, whether you quantify before you choose, and whether your decisions follow from trade-offs you can articulate. This note gives you a repeatable 4-phase structure with a time budget, so that under pressure you always know what to do next. Every problem note in this module ([[Design a URL Shortener]] onward) is written in exactly this structure, so drilling them *is* drilling the method.

## Why this matters

- **The format punishes improvisation.** 45 minutes is brutally short. Candidates who wing it typically spend 25 minutes on a detail the interviewer didn't care about and never produce a complete design. A fixed phase structure guarantees you cover requirements, numbers, architecture, and depth — in that order.
- **Interviewers grade process, not trivia.** Two candidates can land on the same architecture and get opposite outcomes. The one who asked what's out of scope, estimated the write rate, and said *why* a cache goes here passes; the one who drew the same boxes silently does not.
- **The method transfers to real work.** Requirements → estimation → high-level → deep dive is also how a competent design review or RFC at work proceeds ([[What Is System Design]]). Practicing for interviews makes you better at your job, which is the honest reason to invest in it.

## How it works

The 45-minute round splits into four phases. The minute counts are a budget, not a law — but if you're still clarifying requirements at minute 15, you are failing, and you should know you are failing.

| Phase | Budget | Output |
|---|---|---|
| 1. Requirements clarification | 5–10 min | Agreed scope: functional list, non-functional targets, explicit OUT-of-scope list |
| 2. Back-of-envelope estimation | ~5 min | RPS, storage, bandwidth numbers that justify later choices |
| 3. High-level design | 10–15 min | API sketch, data model, one end-to-end diagram, interviewer buy-in |
| 4. Deep dives | 15–20 min | 2–4 hard sub-problems solved with trade-offs, steered by the interviewer |

### Phase 1 — Requirements clarification (5–10 min)

Never start drawing. The prompt ("design WhatsApp") is deliberately underspecified; your first job is to shrink it to something designable in 40 minutes. Cover three lists:

1. **Functional requirements** — what the system does. Propose a short list and ask the interviewer to confirm: "For a chat system I'd cover 1:1 messages, group chat, and delivery status — should I also handle media, or park it?" Proposing-then-confirming is stronger than open-ended "what should it do?", which wastes your own time budget.
2. **Non-functional requirements** — the qualities: scale (users, traffic), latency targets, availability vs consistency preference, durability. These are the requirements that actually shape the architecture; two systems with identical features and 1000x different traffic are different designs ([[Scalability]]).
3. **Explicitly out of scope** — say it out loud and write it in a corner: "Out of scope: registration, spam detection, analytics." This is the highest-leverage sentence in the interview: it protects your time and signals senior judgment. Interviewers almost never object; they usually look relieved.

Ask for numbers directly: "How many daily active users? Read-heavy or write-heavy?" If the interviewer says "you tell me," they are testing [[Back of the Envelope Estimation]] — pick defensible numbers ("assume 100M DAU, roughly Twitter-scale") and move on.

### Phase 2 — Estimation (~5 min)

Turn the agreed scale into three numbers, showing arithmetic out loud:

- **Throughput**: requests/day ÷ 86,400 ≈ requests/sec (use ~100K sec/day for mental math), then multiply by a peak factor — 2–3x for smooth traffic, 5–10x for spiky or event-driven traffic ([[Back of the Envelope Estimation]]). Separate reads from writes using the read:write ratio.
- **Storage**: items/day × bytes/item × retention period. Sanity-check against intuition ("~18 TB over 5 years — a few large disks, fits a modest sharded cluster").
- **Bandwidth**: throughput × payload size, for ingress and egress.

The point is not precision — it is that the numbers *drive decisions you make later*. "500 writes/sec fits a single primary; 50K reads/sec means a cache tier" is the sentence that earns the points. The full technique, with the memorized constants, is in [[Back of the Envelope Estimation]] and [[Latency Numbers Every Engineer Should Know]].

### Phase 3 — High-level design (10–15 min)

Build the skeleton end to end before going deep anywhere:

1. **API** — 3–6 endpoints with verbs, key parameters, and responses. One line each; no exhaustive schemas.
2. **Data model** — the main entities, their key fields, and the access patterns, because access patterns decide the database ([[Choosing a Database]]).
3. **Diagram** — clients → edge ([[Load Balancers]], [[API Gateway]]) → services → cache → datastore → async paths ([[Message Queues vs Event Logs]]). Draw left to right, narrate every box as you draw it: a silent diagram earns nothing.
4. **Get buy-in** — end the phase with: "That's the high-level shape. Which part would you like me to go deep on — the fan-out, the storage, or the delivery path?" This question hands the interviewer the steering wheel, which is exactly what they want, and it guarantees your deep dives are the ones being graded.

Resist detail here. If you catch yourself discussing Redis eviction policy before the diagram is complete, pull up.

### Phase 4 — Deep dives (15–20 min)

Now solve the 2–4 genuinely hard sub-problems — the ones each problem note in this module lists under *Deep dives*. For each: state the problem, give two or three candidate approaches, compare them with the estimation numbers, pick one, and name what you gave up. That five-step beat — problem, options, numbers, choice, cost — is the core graded behavior of the whole interview.

Follow the interviewer's steering. If they ask "what happens when a node dies?", that is not a trap — it is them telling you where the points are. Thinking aloud matters most here: a wrong path reasoned through aloud scores better than a right answer delivered silently.

Save the last 2 minutes to close: walk the diagram once, name the main bottleneck and what you'd evolve first. A deliberate ending beats being cut off mid-sentence.

### Worked example — the first ten minutes

Phases 1 and 2 are the ones candidates under-rehearse, because they feel like preamble rather than content. They are not: they are where the interviewer forms their first judgement, and they are the only part of the round you can script. Here is what good sounds like, for the prompt behind [[Design a Notification Service]]. The numbers below are that note's own estimation, so you can use this as a literal rehearsal.

**Interviewer —** "Design a notification service."

**You —** "I'll scope it first, then put numbers on it, then draw. Can I propose a functional list and have you correct me? I'd cover multi-channel delivery — push, SMS, email, in-app; templates, so callers send a template ID plus variables rather than raw copy; per-user preferences and opt-outs; and delivery status tracking, so support can answer 'did my OTP go out'. Does that match what you had in mind?"

**Interviewer —** "That's right. Anything you'd deliberately leave out?"

**You —** "Yes, and I'd like to say it explicitly so we don't spend time there: out of scope are the campaign-builder UI, send-time or content recommendation, and client-side rendering of the in-app inbox." *— writes OUT OF SCOPE in the corner of the board.*

**You —** "One more functional question, because it changes the architecture rather than the feature list: are there priority tiers? Concretely, does a five-million-user marketing campaign share infrastructure with a login OTP?"

**Interviewer —** "Assume both exist. You tell me what to do about it."

**You —** "Then I'll treat separation as a hard requirement. A campaign that takes forty minutes to drain must not delay an OTP by forty minutes, so critical and bulk can never share a queue's fate. That's the most load-bearing constraint so far and it will show up directly in the design as separate queues per tier."

**You —** "Four non-functional questions. What delivery guarantee — may we ever drop one, may we ever send one twice? What latency target for the critical tier? What daily volume? And is anything about opt-out legally binding?"

**Interviewer —** "Never silently drop. Duplicates are bad, not fatal. Critical should reach the provider in a second or two. Ten million notifications a day. And yes, unsubscribe is legally binding."

**You —** "So: at-least-once attempt plus idempotent suppression of duplicates — exactly-once to a handset isn't achievable end to end, since a provider can time out after it has already sent. Critical target of one to two seconds measured at *provider handoff*, not at delivery, because the last mile isn't ours. And opt-out enforcement is a correctness requirement, not a filter we're allowed to skip under load. Let me put numbers on this."

**You —** "Ten million a day over 86,400 seconds is about **116 per second** average. That's nothing — one modest service handles it. So the average is not what designs this system.

Peak is. A campaign to five million users enqueued over ten minutes is 5,000,000 ÷ 600 ≈ **8,300 per second** sustained, roughly **70x** the average. That ratio *is* the design: ingest rate has to be decoupled from provider send rate, which is the argument for queues instead of synchronous sends, and the bulk path has to be rate-limited independently of the critical path.

Storage: one status record per notification, about 1 KB once you include IDs, channel, state transitions, timestamps and provider receipts. 10M × 1 KB = **10 GB/day**; 90 days hot ≈ **900 GB**. One collection, archived to object storage after that — storage isn't a problem here, so I won't spend time on it.

One assumption I'd like to check: SMS vendors typically cap throughput per account or sender ID at tens to hundreds of messages per second. If that holds, the bottleneck isn't our infrastructure at all — it's the provider, which turns the problem from 'scale the workers' into 'rate-control per provider and keep more than one vendor'."

**Interviewer —** "That's the right instinct. Assume it holds."

**You —** "Then the sentence I'd write at the top of the board is: *116/sec average, 8,300/sec in campaign bursts, bottlenecked by third parties.* That buys queues, per-channel and per-tier isolation, and provider rate limiting — and it tells me sharding the status store is not a day-one concern. Let me draw the high level."

Ten minutes in, nothing designed yet, and the interviewer already has most of what they grade: scope proposed rather than extracted, an out-of-scope list volunteered, one functional question chosen because it changes the architecture, a guarantee stated in achievable terms, arithmetic performed aloud, and every number tied to a decision before a single box is drawn. Note also what is *absent*: no technology name has been spoken yet. "Queue" is a role; which queue is a phase-3 question.

### When the interviewer pushes back or moves the goalposts

Mid-round changes are not accidents. "Now assume 100x the traffic", "what if the regulator requires all data in-country", "actually the campaign is fifty million users" — these are scripted probes, and they test one thing: **do you know which parts of your own design are load-bearing?** Someone who built the design can answer in thirty seconds; someone who assembled it from a memorised template has to start over, and it shows.

A four-step protocol:

1. **Restate the change and name what it invalidates.** "Fifty million in ten minutes, not five — that's about 83,000 per second. My claim that one status-store shard suffices no longer holds, and neither does writing a status row per notification on the bulk path."
2. **Re-run the arithmetic aloud.** The change is usually quantitative, so the answer should be. It also buys you ten entirely legitimate seconds of thinking time.
3. **Find the seam; don't rebuild.** Most changes hit one or two components. Say which, and say which are untouched: "ingest and the critical path are unaffected; the bulk path needs batched enqueue, and the status store needs partitioning by user."
4. **Name the new cost.** "Batched status writes make per-notification status eventually consistent for a few seconds. Fine for marketing; I'd keep per-notification writes on the critical tier."

Distinguish two cases. A **requirement change** is new information — adapt, visibly. **Pushback on a decision you believe is sound** ("isn't a queue overkill here?") may be a genuine correction or a deliberate test of conviction, and you cannot tell which, so treat both the same way: restate the concern in your own words, reason about it aloud against your stated constraints, then adjust with a reason or hold with a reason. Both score. Only the reflexes score zero. And when the pushback is right, say so plainly — "you're correct, I had the read-write ratio backwards, which changes the caching decision". Updating visibly on evidence is a senior signal, not a concession.

### When they name a technology you don't know

It will happen, and it is not the disqualifier candidates fear. Bluffing is. Interviewers follow up on invented capabilities, and the second answer is always worse than the first.

The move is to **convert the product into a role**. Every piece of infrastructure occupies a slot in a design — an ordered log, a leader-elected coordinator, a columnar store, an edge cache — and you know the slots even when you don't know the brand:

> "I haven't operated Pulsar. My understanding is that it's a partitioned log with the storage layer separated from the brokers — if that's roughly right, the role it plays here is the one I was giving Kafka: a durable ordered log per key with independent consumer groups. Can I reason about the role and have you correct me on the product specifics?"

That gives the interviewer three things at once: an honest boundary, evidence that you classify systems by behaviour rather than by name, and an explicit invitation to collaborate. Three rules around it:

- **One sentence of not-knowing, then move on.** Dwelling turns a neutral moment into a negative one.
- **Reason from the family.** [[NoSQL Taxonomy]] and [[Choosing a Database]] exist partly for this: "I've never run ScyllaDB, but it's a wide-column store, so I'd expect query-first modelling and partition-key design to dominate the conversation" is a strong answer from someone who has never touched it.
- **Never invent specifics.** Version behaviour, throughput numbers, "I think it supports X" — this is where candidates lose rounds they were otherwise passing. "I'd have to check" is a complete sentence, and interviewers respect it far more than the alternative.

## Signals interviewers grade

- **Driving the conversation.** You propose scope, you announce phase transitions ("requirements look settled — let me put numbers on this"), you offer the deep-dive menu. Candidates who wait to be dragged through the interview read as junior regardless of technical content.
- **Trade-off articulation.** Every choice framed as *this over that because* — "SQL over document store here because the reconciliation queries are relational" — not as technology name-dropping. Naming what your choice costs ("we accept replica lag; here's where it shows") is the strongest single signal.
- **Quantified reasoning.** Decisions traced to the phase-2 numbers. "The math says 1,200 writes/sec, so a single leader with failover is fine — sharding would be premature" beats "let's shard for scale" every time.
- **Handling pushback.** Interviewers deliberately challenge sound designs to test whether you fold, defend blindly, or evaluate. The pass behavior: treat the pushback as new information, reason about it aloud, and either adjust with a reason or respectfully hold with a reason. Both outcomes can score; reflexive agreement scores zero.
- **Knowing failure modes.** Volunteering "this registry is a single point of failure — mitigated by running three replicas" before being asked. Senior engineers see failure first ([[Failure Modes in Distributed Systems]]).

The same rubric in scanning form. Most interviewers are working from a sheet that looks close to this, and the difference between the two right-hand columns is rarely knowledge — it is habit:

| Signal | Strong looks like | Weak looks like |
|---|---|---|
| Scoping | Proposes a functional list, confirms it, volunteers an out-of-scope list, all inside 8 minutes | Starts drawing from the prompt; or is still clarifying at minute 18 |
| Quantified reasoning | "1,200 writes/sec, so a single leader with failover — sharding would be premature" | "It'll be high scale, so we'll shard"; or computes numbers and never refers to them again |
| Trade-off articulation | Names the option not taken, and what the chosen one costs | Lists products: "we'll use Kafka because it's scalable" |
| Depth | Two or three sub-problems solved to the mechanism, with numbers behind the choice | Ten components touched at depth zero |
| Driving | Announces phase transitions; offers the deep-dive menu; manages the clock | Waits to be asked the next question |
| Failure thinking | Volunteers the single point of failure and its mitigation unprompted | Happy path only; "what if that node dies?" produces a long pause |
| Handling pushback | Restates the concern, re-reasons aloud, then adjusts or holds — with a reason either way | Folds instantly, or defends without engaging the concern |
| Honesty about unknowns | "I haven't run Pulsar; here's the role I need it to fill" | Confident, wrong specifics about a product they have never operated |
| Communication | Narrates while drawing; the diagram is legible; checks in at phase boundaries | Silent drawing; a board only the candidate can read |

## Common failure patterns

- **Jumping to technology names.** "We'll use Kafka, Cassandra, and Kubernetes" before requirements exist. Tools are answers; you haven't established the questions. Name the *role* first ("an append-heavy log store"), the product second.
- **The silent architect.** Drawing for three minutes without speaking. The interviewer can only grade what you externalize; silence is indistinguishable from confusion.
- **One pattern for everything.** Every problem answered with the same microservices-plus-Kafka-plus-cache template, whether it fits or not. The giveaway: the design doesn't change when the requirements do. [[Design a Stock Exchange]] and [[Design a URL Shortener]] should not look alike.
- **Ignoring the interviewer's hints.** They ask "how big is a message, roughly?" and you brush past it — they were steering you toward estimation and you drove by the exit. Every interviewer interjection is a signal; the interview is collaborative by design.
- **Uniform shallow coverage.** Touching ten components at depth zero. Breadth earns the skeleton points once; all remaining points live in the deep dives.
- **No numbers, or numbers abandoned.** Either skipping estimation entirely, or computing "50K reads/sec" and then never letting it influence a single decision. Unused estimation is worse than none — it shows you know the ritual but not the purpose.

## How to practice with these notes

1. **First pass — read in order.** Read each problem note top to bottom to absorb the structure and the canonical answers. Start with [[Design a URL Shortener]]; it is the smallest complete rehearsal of the method.
2. **Second pass — simulate before reading.** Take only a note's title. Set a 45-minute timer, speak aloud, and whiteboard the full four phases. Then open the note and diff: which deep dives did you miss? Which numbers were off by 100x? The gap list is your study queue.
3. **Drill estimation separately.** Do the *Back-of-envelope estimation* section of three different problem notes in one sitting until the arithmetic is automatic. Hesitation at the math costs you phase 3 and 4 time.
4. **Practice the transitions.** Rehearse the connective sentences — scoping, the estimation handoff, the buy-in question, the closing summary. They are what driving *sounds* like, and they come out fluently only if rehearsed.
5. **Cover the full spread.** Drill at least one read-heavy fan-out problem ([[Design a News Feed]]), one connection-heavy real-time problem ([[Design a Chat System]]), one correctness-critical problem ([[Design a Payment System]]), and one infrastructure build ([[Design a Key-Value Store]]) — they exercise different muscles, and the interview you get is not your choice.

### The schedule

The week-by-week plan lives in one place so it cannot drift: **[[Practice Drills]], Part 3** — an eleven-week pass that reads the modules forward as a dependency chain and drills each design problem the week its prerequisites land. Read it there; this note only adds what turns that reading list into interview preparation.

Two things make this a schedule rather than a reading list:

- **Book two mock interviews with a person, minimum.** Nothing you do alone reproduces an interviewer interrupting at minute 12, or asking the follow-up you were quietly hoping to avoid. Week 8 is late for the first one — move it earlier if you can.
- **Record your solo attempts and listen to one.** You will hear the silences, the hedging, and the stretches where you narrated nothing — the three habits that cost the most marks and the three you cannot notice while you are talking.

If the full schedule is not available, the index's weekend track is the honest minimum: estimation, latency numbers, replication, sharding, caching, queues versus logs, and this note.

## Check yourself

> [!question]- Q: Explain the four phases and their time budgets to a colleague in under a minute.
> Requirements clarification (5–10 min): agree functional and non-functional scope and say what's explicitly out. Estimation (~5 min): compute RPS, storage, and bandwidth from the agreed scale, aloud. High-level design (10–15 min): API, data model, one end-to-end diagram, then ask the interviewer which part to go deep on. Deep dives (15–20 min): solve 2–4 hard sub-problems with options, numbers, a choice, and its cost — steered by the interviewer. Close with a 2-minute walkthrough naming the main bottleneck.

> [!question]- Q: What happens if you skip the buy-in question at the end of the high-level phase?
> You pick deep dives blind. You may spend 15 minutes on storage detail when the interviewer's rubric wanted the fan-out path, and those minutes are unrecoverable in a 45-minute round. The buy-in question aligns your remaining time with what is actually being graded — and asking it is itself a graded signal of collaboration.

> [!question]- Q: The interviewer challenges a sound decision you made. What is the pass behavior, and what are the two fail behaviors?
> Pass: treat it as new information — restate the concern, reason about it aloud against your constraints, then either adjust with a stated reason or hold your position with a stated reason. Fail one: fold instantly and redesign without reasoning (signals no conviction). Fail two: defend reflexively without engaging the concern (signals ego over analysis). The interviewer is testing the *process* of handling disagreement, not hunting for a specific answer.

> [!question]- Q: Why is estimation you compute but never use worse than no estimation at all?
> It demonstrates that you know the interview ritual but not its purpose. Estimation exists to drive decisions — cache or no cache, shard or single node, push or pull. Numbers computed and abandoned prove you can divide by 86,400 but not that you can design from data, and the interviewer sees the disconnect clearly.

> [!question]- Q: Mini-drill — you get "design Google Docs" cold. Without designing anything, produce phase 1: a proposed functional scope, three non-functional questions you would ask, and three things you would rule out of scope.
> One defensible version — functional: create/edit documents, concurrent editing by multiple users, view others' changes in near-real time, basic sharing permissions. Non-functional questions: how many concurrent editors per document must we support; what edit-propagation latency is acceptable; is offline editing with later merge required? Out of scope: comments and suggestions, version history UI, exports and formats, search across documents. (Any similar partition is fine — the graded skill is producing a crisp, confirmed scope quickly.)

## Sources

- *System Design Interview — An Insider's Guide, Vol. 1* (Alex Xu) — chapter "A Framework for System Design Interviews", the basis of the 4-phase structure.
- *System Design Interview, Vol. 2* (Alex Xu, Sahn Lam) — the same framework applied to harder problems.
- *Designing Data-Intensive Applications* (Kleppmann), ch. 1 — the vocabulary of load parameters and non-functional requirements that phase 1 and 2 rely on.
- Jackson Gabbard, "How to Succeed in a System Design Interview" (YouTube) — an ex-Facebook interviewer on what graders actually look for.
- The *Grokking the System Design Interview* course (Design Gurus) — popularized the requirements → estimation → design → deep-dive sequencing.
