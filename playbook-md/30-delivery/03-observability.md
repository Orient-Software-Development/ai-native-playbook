# Observability

> Your sensors don't get to stop at the merge gate. **Observability** — structured logs, metrics, and traces from running code — is the feedback loop that extends *past merge into production*, so you can see what the system actually does for real users, not just that the deploy succeeded. Without it the harness goes blind the moment code leaves CI, and "it deployed" gets mistaken for "it works."

Breadcrumb: [Playbook](../README.md) › Delivery

## Story so far

Every sensor we've built so far fires *before* the code goes live — tests, type checks, evidence captured at verification. They prove the change is correct against the inputs you *thought of*, in an environment you *controlled*. Production is neither of those things: real volumes, real concurrency, real edge cases nobody specced. This page builds the sensor for that messier world — because the most dangerous sentence in all of delivery is "the deploy succeeded," said as though it meant the feature actually works.

## The principle

Every sensor in this playbook so far fires *before* the change is live —
[tests, type checks, fitness functions](../20-harness/03-sensors-feedback.md) in [CI](02-ci-and-cd.md),
[evidence captured at verification](../10-lifecycle/05-verify-proof-not-vibes.md). They prove the change is
correct against the inputs you *thought of*, in the environment you *controlled*. Production is neither: real
data volumes, real concurrency, real edge cases nobody specced. **Observability** is the sensor for *that*
world — the runtime feedback loop that tells you how the system behaves once it's actually running.

It's built from three kinds of signal, and a junior reader should keep all three straight:

- **Logs** — a timestamped record of discrete things that happened ("write to deal 412 failed: constraint
  violation"). Most useful when they're **structured** (see below), not free-text prose.
- **Metrics** — numbers aggregated over time: request counts, error rates, latencies, queue depths. Metrics
  tell you *that* something changed — the error rate doubled after the 2pm deploy — and let you alert on it
  automatically.
- **Traces** — the path of a single request as it moves across functions and services, with timing at each
  hop. When a metric says "checkout got slow," the trace tells you *where* the time went.

Together they answer the question CI structurally cannot: *is this working in production, right now, for real
users?* CI proves *safe to ship*; observability proves *actually working once shipped* — and the two are not
the same claim.

## Why it works

**A successful deploy is not a working feature.** The deploy pipeline's job ends at "the new artifact is
running" — it has no idea whether the feature *does the right thing* under real load. A change can pass every
CI check, deploy green, and still be broken in production: a query that's instant on seed data crawls on a
million real rows; a third-party call that always succeeded in tests times out for real users; an edge case
the spec missed throws for one customer in fifty. Without observability, the *only* sensor left for these is a
user complaint — which means you find out late, from the worst possible source, with no signal pointing at
the cause. Observability turns that silent gap into a visible one: the error rate spikes, the latency graph
steps up right at the deploy, the failed-write log appears — and the failure announces itself instead of
festering.

**Structured signal is what makes diagnosis — human or agent — possible.** A log line that's prose
("something went wrong saving the deal") can be read by a person but not *queried*; you can't ask "how many
of these in the last hour, and for which tenant?" A **structured** log — emitted as machine-readable fields
(`event=deal_write_failed deal_id=412 tenant=acme reason=constraint`) instead of a sentence — can be
filtered, counted, and correlated. This matters doubly in an AI-native team: when you hand a production
problem to an agent, structured logs and metrics are *evidence it can actually consume and reason over* — it
can read the fields, find the pattern, and locate the fault, the same way
[evidence artifacts](../10-lifecycle/05-verify-proof-not-vibes.md) make verification checkable rather than a
vibe. Prose logs force the agent (and you) to guess; structured signal lets it diagnose.

**It closes the loop back to the spec.** Observability isn't only for firefighting. What production tells you
— this query is slow at real scale, this edge case happens more than anyone guessed, users abandon at this
step — is *input to the next change*. The runtime signal becomes a finding, the finding becomes a
[normative requirement](../10-lifecycle/01-spec-the-contract.md) or an acceptance criterion (a performance
budget, a new edge case to handle), and the loop runs again. A harness with no production sensor can only
ever react to complaints; a harness with one *learns from reality* and feeds it forward into the next spec.

## How to apply it

- **Emit structured logs, not prose.** Log events as machine-readable fields (event name, ids, the tenant or
  user, the outcome) so they can be filtered, counted, and correlated — not sentences a human must read one at
  a time. The test: could you answer "how many of these in the last hour, broken down by customer?" without
  grepping prose? If not, it isn't structured enough.
- **Instrument as you build, not after an incident.** Adding the log, the metric, and the trace span is part
  of the [slice](../10-lifecycle/03-task-slicing.md), the same way its test is — and an agent can be told to
  instrument the code it writes. Observability bolted on after a production fire is always missing the one
  signal you needed.
- **Measure what the spec cares about.** Turn the feature's real requirements into metrics: if a requirement
  is "the export must finish in reasonable time," there should be a latency metric on the export and an alert
  when it regresses. The [acceptance criteria](../10-lifecycle/01-spec-the-contract.md) tell you what to
  watch.
- **Alert on symptoms users feel.** Page on error rate, latency, and failed operations — the things that mean
  the product is broken for someone — not on every internal blip. An alert that fires constantly gets muted,
  and a [muted alert is no sensor](../20-harness/04-keep-quality-left.md), the same way a skipped hook is no
  gate.
- **Watch the deploy.** The most useful moment for a metric is right after a release: a graph that steps up
  exactly at the deploy time is your fastest, clearest signal that *this change* caused *that regression*.
  Make "what did the last deploy do to error rate and latency?" a question you can answer in seconds.
- **Feed findings back into specs.** When production surfaces a slow path or an unhandled case, write it into
  the next spec as a requirement or acceptance criterion — don't just hot-patch it and move on. The loop only
  compounds if the lesson lands in the durable contract.
- **Don't:** treat a green deploy as proof the feature works; log unqueryable prose; bolt on observability
  only after an outage; alert so noisily the alerts get ignored; or fix a production finding without folding
  it back into the spec.

## In practice

The *per-user deal priority* feature ships — fully [verified](../10-lifecycle/05-verify-proof-not-vibes.md),
[merged behind a green gate](02-ci-and-cd.md), deployed clean. The deploy pipeline reports success. Everyone
moves on.

**Without observability.** In production the deals-list query — instant against the handful of seed rows in
CI — has to left-join each user's priorities across hundreds of thousands of real deals, and it's slow.
There's no metric on the query and no structured log, so nothing fires. The page just feels sluggish. Two
weeks later a frustrated account manager mentions it in passing; by then the slow query is buried under a
dozen later changes, and finding it means re-deriving from scratch which change made the list slow. The
sensor of last resort — an annoyed human — caught it, late and vague.

**With observability.** The query slice shipped with a latency metric and a structured log on the list
endpoint. The afternoon of the deploy, the latency graph steps up sharply right at release time and an alert
fires: *deals-list latency regressed.* The structured logs show the slow endpoint and the join, the trace
shows the time going into the priority left-join, and that evidence — queryable, not prose — is handed to an
agent. With the fault localised it adds the missing index and the latency drops back. Then the loop closes:
the finding becomes a new acceptance criterion in the spec — *the deals list must stay under its latency
budget at production scale* — wired to the very metric that caught it, so the next change that regresses it
fails *before* a user ever feels it.

The lesson the example carries: the bug existed in both worlds — the difference was purely whether the system
could *see itself*. CI proved the feature safe to ship; only observability proved whether it actually worked
once shipped, turned a vague "it feels slow" into a precise, agent-readable signal, and fed the lesson back
into the contract. Without the runtime sensor the harness ends at the merge gate and production is a black
box; with it, the feedback loop runs all the way to the user and back.

## Anti-patterns

- **"It deployed, so it works."** Treating a green deploy as proof the feature is correct in production — when
  the deploy pipeline only knows the artifact is *running*, not that it does the right thing under real load.
  ([Failure modes](../40-anti-patterns/01-failure-modes.md))
- **Prose logs.** Free-text log lines a human can read but nobody can *query*, so you can't count, filter, or
  correlate them — and an agent handed the incident can only guess. The opposite of
  [evidence you can check](../10-lifecycle/05-verify-proof-not-vibes.md).
- **Observability-after-the-fire.** Adding the missing log or metric only once an outage has already
  happened — guaranteeing the one signal you needed wasn't there when it mattered. Instrument as part of the
  slice, not the postmortem.
- **The alert that cried wolf.** So many low-value alerts that the team mutes them, so the one real alert is
  ignored too — a [sensor everyone learns to skip](../20-harness/04-keep-quality-left.md).
- **The unclosed loop.** Hot-patching a production finding without writing it back into the spec, so the
  lesson never becomes a durable requirement and the same class of bug ships again.

> **Next up — [Drift and Health Sensors](04-drift-and-health-sensors.md):** observability watches production in real time. But some rot isn't caused by any single change at all — it accumulates quietly between them, or arrives from outside your repo entirely. The last delivery page builds the sensors that run on a *clock* to catch exactly that slow slide.

---
[← Previous: CI and CD](02-ci-and-cd.md) · [Contents](../README.md) · [Next → Drift and Health Sensors](04-drift-and-health-sensors.md)

Related: [Drift and Health Sensors](04-drift-and-health-sensors.md) · [Sensors — Feedback](../20-harness/03-sensors-feedback.md) · [Verify — Proof, Not Vibes](../10-lifecycle/05-verify-proof-not-vibes.md) · [CI and CD](02-ci-and-cd.md) · [Spec — The Contract](../10-lifecycle/01-spec-the-contract.md) · [Failure Modes](../40-anti-patterns/01-failure-modes.md)
