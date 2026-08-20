# Glossary

Plain-language definitions of the terms of art used across the playbook. Every page also defines its
terms in-line on first use — this is the backstop for when you forget one, and the single place each
definition lives so two pages never drift apart. Each entry links to the page that treats the term
most fully.

Breadcrumb: [Playbook](README.md)

---

### Acceptance criteria

The observable checks that decide whether a requirement is "done" — stated so another person can
make the call without interpreting. If you can't name the check that proves a requirement, the
requirement is too vague to build. → [Spec — The Contract](10-lifecycle/01-spec-the-contract.md)

### Agent instruction file

The short, checked-in file an agent reads first on every task (commonly `CLAUDE.md` or `AGENTS.md`):
repo layout, non-negotiable conventions, build/test commands, and pointers to deeper context. Keep
it lean and current — it's read every time, so length is a recurring tax and staleness teaches the
wrong thing with authority. → [Guides — Feedforward](20-harness/01-guides-feedforward.md)

### AI technical debt

The plausible-but-mediocre code an agent accumulates faster than anyone reviews it — stubs, shortcuts,
guessed shapes. It compounds *invisibly* because no one feels accountable for code they didn't write, so
it has no natural owner to pay it down. Treat it as a managed budget: make it visible in the
[system of record](#system-of-record) and pay it down continuously, like a high-interest loan.
→ [The Responsible Team and AI Debt](50-adoption/03-responsible-team-and-ai-debt.md)

### Anti-cheat constraint

An explicit feedforward rule that forbids the agent from faking success — "you may not weaken a
failing test to pass it," "you may not patch source merely to satisfy a test," "tests must assert
real state." The feedforward half of stopping reward hacking; only holds when a state-asserting
sensor enforces it. → [Guides — Feedforward](20-harness/01-guides-feedforward.md)

### Attention budget

The context window seen as a *budget* rather than storage: the agent has limited attention to spend,
so every token you load competes with the tokens that actually matter. The goal is the smallest
high-signal set the task needs, not the most you can fit.
→ [Context Engineering](00-foundations/02-context-engineering.md)

### Audit spec

A standing, end-to-end walk-through of a feature that you re-run *later* to catch drift — a feature
that worked at merge but quietly broke a few changes on. The runtime sibling of reconciling a spec
against its code: it makes a silent regression surface on purpose.
→ [Verify — Proof, Not Vibes](10-lifecycle/05-verify-proof-not-vibes.md)

### Autonomy ladder

The four named rungs an agent's autonomy climbs, per zone: **propose-only**, **merge-with-review**,
**merge-on-green**, and **self-merge with post-merge correction**. Promotion is per-zone and
evidence-based — the zone's sensors have *actually caught* the failure class that worries you — and
demotion is automatic on an escaped defect. The precise form of [levels of autonomy](#levels-of-autonomy).
→ [Growing the Harness](50-adoption/02-growing-the-harness.md)

### Behaviour harness

The hardest of the three regulation categories: the guides and sensors that prove the code does what the
*business* wanted — not just that it's clean (maintainability) or well-shaped (architecture fitness). Its
core lever is the [invariant](#invariant), bound to a named scenario so the test derives from the intent,
not the code. Still insufficient for high autonomy, so human verification stays.
→ [Behaviour Harness](20-harness/05-behaviour-harness.md)

### Blueprint

A curated, canonical example of a pattern — one correct implementation the agent is pointed at by
name ("new endpoints look like this one") instead of having the pattern described in prose. A worked
example removes the interpretation a rule leaves open. → [Guides — Feedforward](20-harness/01-guides-feedforward.md)

### Brownfield (legacy adoption)

An existing, often years-old codebase you adopt the harness *into* — typically large, sparsely tested,
and untyped where it matters, so the harness is hardest to build precisely where it's needed most. You
don't fix it in one rewrite; you earn it one high-risk *seam* (a place two responsibilities meet) at a
time, pinning current behaviour with [characterization tests](#characterization-test) and drawing
boundaries incrementally, highest-risk first. → [Legacy and Brownfield](50-adoption/04-legacy-and-brownfield.md)

### Ceremony mismatch

Full spec machinery aimed at a change whose intent fits in a sentence — four user stories and sixteen
acceptance criteria for a one-line fix. It produces review burden, not safety: the paperwork should grow
and shrink with the problem, while the discipline of stating expected behaviour before building never
goes away. → [Spec — The Contract](10-lifecycle/01-spec-the-contract.md)

### Change classification

Sorting a change to a [spec](#specification-spec) before it merges: **additive** (a new requirement),
**compatible** (clarifies without changing promised behaviour), **breaking** (changes what an existing
requirement promises), or **ambiguous** (can't tell). Additive and compatible flow through the normal
gate; breaking and ambiguous need a human sign-off, because they redefine the contract itself.
→ [Drift and Health Sensors](30-delivery/04-drift-and-health-sensors.md)

### Characterization test

A test that captures what a piece of code *currently does* — bugs and all — rather than what it
*should* do, so that when the code is refactored, any change in behaviour turns a test red instead of
slipping by silently. The safety net you raise *before* touching tangled [legacy
code](#brownfield-legacy-adoption): not a correctness test, a change-detector.
→ [Legacy and Brownfield](50-adoption/04-legacy-and-brownfield.md)

### Code graph (structural index)

A machine-generated index of a codebase's symbols and how they connect (files → functions → classes →
call chains) that an agent queries directly instead of grepping the tree file by file. Feedforward the
repo generates about *itself* — it cuts the tokens spent locating prior context and improves navigation
on a large codebase. → [Repo Structure and Legibility](20-harness/02-repo-structure-and-legibility.md)

### Codemod

A mechanical code transform applied by a tool rather than reasoned out by the agent token by token — a
sweeping rename or API migration performed exactly and identically across the whole codebase. A
[computational guide](#computational-guide): asking an agent to "update all the call sites" invites a
missed one; a codemod can't miss. → [Guides — Feedforward](20-harness/01-guides-feedforward.md)

### Compaction

Summarising a long session's history *in place* so it keeps fitting in the window. It carries the old
conversation's framing forward — which makes it the right move when you're **mid-task on one coherent
thread** and that framing is signal, but the wrong one before an independent-judgement step, where a
[reset](#reset) is needed to drop the bias. Compaction preserves continuity; a reset gives a clean
slate. → [Context Engineering](00-foundations/02-context-engineering.md)

### Computational guide

[Feedforward](#feedforward) that shapes what the agent can *produce* through tooling rather than words
it reads — a [language server](#language-server-lsp), a type system, a scaffolding script (a generator
that stamps out a correct module skeleton), a [codemod](#codemod). Its counterpart is the *inferential* guide
(instruction files, conventions, blueprints) the agent reasons over. Computational guides barely touch
the [attention budget](#attention-budget), so reach for one before a written rule whenever a tool can do
the job. → [Guides — Feedforward](20-harness/01-guides-feedforward.md)

### Computational sensor

A [sensor](#sensor) that checks a rule deterministically — a test, linter, type checker, formatter,
fitness function. Fast, cheap, and never falsely confident: it can't be argued out of a real finding.
Reach for one of these before an [inferential sensor](#inferential-sensor) whenever structure can express
the rule. → [Harness Engineering](00-foundations/03-harness-engineering.md)

### Context rot

The measured tendency for answer quality to drop as the input gets longer — models read the front of
a long input well and skim the tail — so quality frays *well before* the window is full. The reason a
big context window is not the same as usable context.
→ [Context Engineering](00-foundations/02-context-engineering.md)

### Context window

Everything the model can actually see at the instant it acts — the prompt, instructions, files, and
history loaded into this turn. Anything not in the window effectively doesn't exist to the agent.
→ [Context Engineering](00-foundations/02-context-engineering.md)

### Course-correction

Watching the agent work and redirecting it at the *first* wrong turn rather than at the finished
diff. Early redirects are cheap (a sentence); a wrong direction left to run compounds into a rebuild,
because everything the agent builds after it inherits the mistake.
→ [Code With the Agent](10-lifecycle/04-code-with-the-agent.md)

### Continuous delivery / deployment (CD)

The path that takes a change CI already verified and *releases* it to environments. **Delivery** keeps the
verified artifact always *ready* to ship at the push of a button; **deployment** ships it *automatically*.
Distinct from [continuous integration](#continuous-integration): CI gates the *merge*, CD gates the
*release* — keep them on separate triggers so a deploy outage can't freeze the trunk and a release path
can't bypass the gate. Promote the exact artifact CI blessed; never rebuild to ship.
→ [CI and CD](30-delivery/02-ci-and-cd.md)

### Continuous integration

The practice of integrating every change into the shared [trunk](#trunk-based-development) *continuously* —
many small verified merges a day — rather than saving work up for one big merge. In its original sense it
names the **habit**, not the server: "we have a CI server" is not continuous integration unless the team
actually merges continuously. The automated [CI gate](30-delivery/02-ci-and-cd.md) is what makes that habit
safe. → [CI and CD](30-delivery/02-ci-and-cd.md)

### Convergence

The discipline of making each follow-up pass leave the codebase *more coherent*, not just bolt more
on — the counter-force to entropy at agent throughput. Its core move is the **rule of three**: the
first time you write a pattern is fine, the second you note it, and before the *third* copy you
extract the shared piece into one named place — early enough that the common shape is clear, late
enough that you actually know what's shared. → [Review and Convergence](10-lifecycle/06-review-and-convergence.md)

### Decision record (ADR)

A short, durable note recording *why* a choice was made — the context, the options, the call — one
file per decision (commonly an Architecture Decision Record). As feedforward it stops the agent
unknowingly contradicting a settled decision or reopening a closed question.
→ [Guides — Feedforward](20-harness/01-guides-feedforward.md)

### Design guide

A checked-in `DESIGN.md` or file of design tokens (colours, spacing, type scale, component rules)
the agent reads before building any UI. Turns a subjective "make it look right," which the agent can
only guess at, into named values it can apply and a reviewer can grade against.
→ [Guides — Feedforward](20-harness/01-guides-feedforward.md)

### Draft-aware gating

Running only the fast checks (lint, types, unit tests) while a pull request is a *draft* still being
iterated — often by an agent pushing repeatedly — and running the full heavy suite (integration, e2e,
performance) only once the PR is marked *ready*, requiring it green to merge. Keeps the [CI
gate](#continuous-integration) both strict (nothing merges unverified) and fast (work-in-progress pushes
don't pay for the heavy suite), so nobody routes around a slow gate.
→ [CI and CD](30-delivery/02-ci-and-cd.md)

### Drift and health sensor

A [sensor](#sensor) that runs on a *schedule* (a clock) rather than on a change (a diff), to catch decay
no single change causes: dead code, a newly-disclosed dependency vulnerability, an eroding coverage or
[mutation-score](#mutation-testing) *trend*, or architecture drift. A scheduled "janitor" run can both
flag the drift and open the PR that fixes it. Distinct from [observability](#observability), which
watches production at runtime rather than the repo on a clock.
→ [Drift and Health Sensors](30-delivery/04-drift-and-health-sensors.md)

### Escalation ladder

The enforcement rungs a rule can live on, weakest to strongest: an in-prompt instruction → a
session-level goal re-checked each turn → a deterministic [hook](#hook) that blocks until the check
passes → an independent second-opinion agent. Climb only as high as the rule demands — each rung costs
more. Rule of thumb: instructions for judgment calls, hooks for invariants.
→ [Guides — Feedforward](20-harness/01-guides-feedforward.md)

### Evaluator calibration

Building judgment into a separated evaluator the way you'd train a new reviewer: anchor its verdicts
with worked, reasoned examples, tune it in a loop against your own judgments, stress-test what the
criteria wording *rewards*, and don't chase its every finding. Separation removes the author's bias;
calibration is what adds the skepticism. → [Verify — Proof, Not Vibes](10-lifecycle/05-verify-proof-not-vibes.md)

### Evidence artifact

The captured proof attached to a verified change so a reviewer can *see* it rather than trust a
claim — a screenshot of the real screen in the real state for UI, a saved log excerpt or state dump
of the real persisted result for backend. "Verified" with nothing attached is a vibe, not proof.
→ [Verify — Proof, Not Vibes](10-lifecycle/05-verify-proof-not-vibes.md)

### Explore → plan → code

The lifecycle step between spec and code: the agent first *reads* the relevant code (explore), then
proposes *how* it will build the feature in prose (plan), and only writes code once that plan is
approved. Catches a wrong approach while it's still a paragraph, not a diff.
→ [Plan Before Code](10-lifecycle/02-plan-before-code.md)

### Feature flag

A runtime switch that keeps incomplete or not-yet-announced code dark in production. It lets unfinished
work merge to the [trunk](#trunk-based-development) — staying integrated with everyone else's changes —
instead of diverging on a long-lived branch, while the trunk stays releasable because the flag is off.
→ [Trunk-Based Development](30-delivery/01-trunk-based-development.md)

### Feedforward

Everything you put in front of the agent *before* it acts — instruction files, conventions,
blueprints, design guides, constraints — to raise the odds of a right first attempt. The "before"
half of the [harness](00-foundations/03-harness-engineering.md); its partner is feedback (sensors),
which grades the work after. → [Guides — Feedforward](20-harness/01-guides-feedforward.md)

### Fitness function

A [sensor](#sensor) that asserts the system's *shape* rather than a single result — "this module may not
import that one," "p95 latency stays under budget," "no database key crosses a context boundary." The way
architecture rules become checks the agent is blocked from breaking. Push each one to the layer the rule
actually lives in, or it's invisible where it matters. → [Sensors — Feedback](20-harness/03-sensors-feedback.md)

### Flop zone

A part of the work where the agent *reliably* gets it wrong: subjective frontend/design, cross-cutting
refactors, ambiguous business rules, security-sensitive code. Named on purpose so the team can decide each
one's controls — what to automate, what to guard with a sensor, what stays human — rather than applying the
same trust everywhere. → [The Responsible Team and AI Debt](50-adoption/03-responsible-team-and-ai-debt.md)

### Goodhart's law

"When a measure becomes a target, it stops being a good measure." The reason
[reward hacking](#reward-hacking) happens: a test is a *proxy* for "the feature works," and once
passing it becomes the goal, the agent optimises the proxy and the real thing drifts away. Also called
*specification gaming*. → [Failure Modes](40-anti-patterns/01-failure-modes.md)

### Guide

A single piece of [feedforward](#feedforward): one instruction file, convention, blueprint, design
guide, skill, or constraint. A guide changes the *odds* of a correct attempt — it never enforces, so
every guide that matters is backed by a sensor. → [Guides — Feedforward](20-harness/01-guides-feedforward.md)

### Harnessability

How readily a codebase accepts guides and sensors. Strong static types, clear module boundaries,
established frameworks, and high test coverage all raise it — each turns into a cheaper, more
reliable check. The payoff is double: the same investments that make a codebase easier for humans
make it easier for agents. → [Harness Engineering](00-foundations/03-harness-engineering.md)

### Harness template

A reusable bundle of guides and sensors for a recurring service shape (a CRUD API, an event processor, a
dashboard), so the next service of that shape starts *already harnessed* instead of being hand-built. The
grown harness becomes the [blueprint](#blueprint) for its whole topology — but it's a shared dependency,
so budget the upkeep to keep instances from drifting from it.
→ [Growing the Harness](50-adoption/02-growing-the-harness.md)

### Hook

A script the harness itself runs on an event — before an edit, after an edit, at the end of a turn —
that can hard-block an action rather than suggest against it. The deterministic layer where everything
the agent *reads* is advisory: a hook never passes through the context window, so it fires whether or
not the agent remembered the rule. → [Guides — Feedforward](20-harness/01-guides-feedforward.md)

### Inferential sensor

A [sensor](#sensor) that judges *semantically* — an AI reviewer reading a diff for a misleading name, a
lying comment, or drift from the spec. Slower, costs tokens, non-deterministic, and can be talked out of a
real finding, so it's reserved for what a [computational sensor](#computational-sensor) structurally can't
reach. → [Harness Engineering](00-foundations/03-harness-engineering.md)

### Intent elicitation

Having the agent interview *you* before a word of spec exists — challenging assumptions, making
constraints explicit, pinning success criteria, deciding what should *not* be built — then writing the
[spec](#specification-spec) from the interview. The questions are cheap; the guesses they replace are
not. Execution then starts in a fresh session, leaving the interview's residue behind.
→ [Spec — The Contract](10-lifecycle/01-spec-the-contract.md)

### Invariant

A statement of what must *always* hold for a scenario, regardless of input or implementation — "after any
transfer, total balances are unchanged," "a sorted list sorted again is identical." Because it's a property
of the *problem*, a test built from it is bound to the business rule and can't silently encode the code's
current bug. The lever of the [behaviour harness](#behaviour-harness).
→ [Behaviour Harness](20-harness/05-behaviour-harness.md)

### Just-in-time retrieval

Keeping *references* in context — paths, queries, links — and fetching the content at the moment of
need, rather than pre-loading everything that might turn out relevant. Includes interrogating a large
output with small query tools instead of loading the whole artifact into the window.
→ [Context Engineering](00-foundations/02-context-engineering.md)

### Keep quality left

The rule for *placing* [sensors](#sensor) along the pipeline (pre-commit → pre-push → CI → staging): push
each check to the *earliest* stage whose speed it can afford, and no further. Fast cheap checks on every
commit so failures surface in seconds; slow expensive ones in CI so they don't tax iteration and sit where
they can't be bypassed. Not "run everything as early as possible" — overloading a git hook just teaches the
team to skip it. → [Keep Quality Left](20-harness/04-keep-quality-left.md)

### Language server (LSP)

The engine that gives an editor go-to-definition, find-references, and type-on-hover, exposed to the
agent so it can *look up* a symbol's definition and callers exactly instead of grepping the text and
guessing. A [computational guide](#computational-guide): real code intelligence at almost no
[context](#attention-budget) cost. → [Guides — Feedforward](20-harness/01-guides-feedforward.md)

### Legibility

How readily a person or agent can navigate a repo and reason about the system *directly from its
structure* — conventional layout, contracts in one named place, tests beside their code, boundaries
written where they can be read. The cheapest feedforward there is: arranged once, it steers every future
task for free. Distinct from enforcement — a boundary buried in a lint config is *enforced* but not
legible. → [Repo Structure and Legibility](20-harness/02-repo-structure-and-legibility.md)

### Levels of autonomy

How much of the lifecycle the agent is trusted to drive unsupervised — and the principle that it's *earned*,
not granted. A zone widens the agent's scope only as its [controls](#sensor) prove they catch the agent's
mistakes; a [flop zone](#flop-zone) keeps a human in the loop until it's harnessed, and some (money,
security) may stay human on purpose. → [Growing the Harness](50-adoption/02-growing-the-harness.md)

### LLM-as-judge

Using a separate language-model agent to grade a change against the spec — an attempt to *scale* the
skeptical-evaluator idea. The *concept* of a separate evaluator is sound; wiring an automated judge
into the gate is still **experimental** — measure its false-positive rate, cost, and run-to-run
consistency, and keep a human signing off until it earns trust.
→ [Verify — Proof, Not Vibes](10-lifecycle/05-verify-proof-not-vibes.md)

### Memory store

A semi-durable scratchpad an agent can write to and retrieve from across turns — longer-lived than the
[context window](#context-window) but not the canonical source of truth. An accelerator, not the
record. → [Context Engineering](00-foundations/02-context-engineering.md)

### Minimum viable harness

The smallest set of controls that still closes the loop on day one of a project: one
[agent instruction file](#agent-instruction-file), a formatter and linter, a type check, a gate that
*blocks* on red, and one real [behaviour](#behaviour-harness) test. Weak but *whole* — one of each control
type (feedforward, sensor, gate) wired end to end — so you have a loop to
[grow](50-adoption/02-growing-the-harness.md) instead of a grand plan you never finish.
→ [The Minimum Viable Harness](50-adoption/01-minimum-viable-harness.md)

### Mutation testing

Testing the tests: inject a small deliberate bug into the source (flip `<` to `<=`, delete a line) to make
a "mutant," then rerun the suite. If a test fails, the mutant is *killed*; if the suite still passes, the
mutant *survived* — concrete proof those tests wouldn't catch that bug. The empirical answer to the
[coverage illusion](#sensor): coverage says a line ran, a survived mutant says nothing checked the result.
→ [Behaviour Harness](20-harness/05-behaviour-harness.md)

### Normative requirement

A requirement stated as a binding obligation with a clear pass/fail line — "the export **must**
include every account for the current tenant" — rather than a loose wish. Atomic enough to verify
on its own, and written in business language. → [Spec — The Contract](10-lifecycle/01-spec-the-contract.md)

### Observability

The runtime feedback loop: [structured logs](#structured-logging), metrics (aggregated numbers — error
rates, latencies, counts), and traces (the timed path of one request across the system) that let you see
what running code actually does in production. The [sensor](#sensor) that extends *past the merge gate*,
where CI's checks stop — without it, "it deployed" gets mistaken for "it works." Its findings feed back into
the next [spec](#specification-spec). → [Observability](30-delivery/03-observability.md)

### Plan

A short written statement of *how* a feature will be built — the files it will touch, the approach
for each, the order of work, the tests that prove it, and the open questions hit along the way. The
[spec](#specification-spec) owns the *what*; the plan owns the *how*. You review and approve it
*before* any code is written. Unlike the spec, the plan can be **ephemeral** — it can be discarded
once the code is written and reviewed against it; the spec is the durable contract that stays.
→ [Plan Before Code](10-lifecycle/02-plan-before-code.md)

### Plan mode

A read-only agent mode: it can investigate the codebase and draft a [plan](#plan), but cannot edit
files. Keeps exploration from sprawling into half-built code and keeps the plan a proposal you
approve rather than a change already made. → [Plan Before Code](10-lifecycle/02-plan-before-code.md)

### Progressive disclosure

Structuring information so the agent loads it in stages: lightweight identifiers first — file paths,
skill names and one-line descriptions, an index — full content only when judged relevant, appendix
detail only when actually needed. The reason a [skill's](#skill) metadata and an instruction file's
index matter more than their bodies. → [Context Engineering](00-foundations/02-context-engineering.md)

### Property-based testing

A test that asserts an [invariant](#invariant) — "output is never negative," "round-trip leaves it
unchanged" — and lets a tool *generate* the inputs that try to break it, rather than hand-picking examples.
Because the property comes from the spec's meaning (docstring, types, names), the test is independent of the
function body and finds the edge case a hand-written example skips.
→ [Behaviour Harness](20-harness/05-behaviour-harness.md)

### Property-testing loop

The five-step loop an agent runs [property-based testing](#property-based-testing) as: comprehend the
target, propose properties grounded in that context, generate the tests, execute *with self-reflection*
(did the code fail or did my test?), and report only above a confidence bar — with findings ranked by a
scoring rubric before a human reads them. → [Behaviour Harness](20-harness/05-behaviour-harness.md)

### Quality gate

The layered set of [sensors](#sensor) a change must pass — static checks (lint, types, fitness functions)
under unit → integration → performance → end-to-end tests. A "really good" gate is this *pyramid*, where
each tier catches a class the cheaper one structurally can't, not one fat suite doing everything.
→ [Sensors — Feedback](20-harness/03-sensors-feedback.md)

### Reset

Starting the agent on a fresh, empty context and carrying forward only a small handoff of what
matters — a genuine clean slate. Distinct from [compaction](#compaction): a reset drops the old
conversation's bias entirely, which is what keeps later steps (like writing tests) honest.
→ [Context Engineering](00-foundations/02-context-engineering.md)

### Reward hacking

An agent cheating its way to a passing check instead of doing the work — either weakening the check
(editing a test down to "didn't throw") or gaming the code (hard-coding the test's expected output).
Green CI, a silent bug. Not malice but [Goodhart's law](#goodharts-law); an *unconstrained* agent does
it reliably. Prevented two-sided: [anti-cheat constraints](#anti-cheat-constraint) (feedforward) plus
[sensor integrity](#sensor-integrity) (feedback). → [Failure Modes](40-anti-patterns/01-failure-modes.md)

### Scenario (SCN)

A named unit of behaviour with a stable identifier (e.g. `SCN-LEDGER-014`) and an approved
[invariant](#invariant) describing what must hold for it. The test cites the *scenario*, not the function,
so the contract is anchored to intent; trace it both ways (every scenario has a test, every test cites a
real scenario) and a dropped contract becomes visible. → [Behaviour Harness](20-harness/05-behaviour-harness.md)

### Self-evaluation failure

An agent's bias toward judging its own work as correct: it optimises for *task-complete*, so the
same process that wrote the code reads it as finished *and* good. The reason the author never
self-certifies — grading must go to a separate, skeptical evaluator with the spec in hand.
→ [Verify — Proof, Not Vibes](10-lifecycle/05-verify-proof-not-vibes.md)

### Sensor

Anything that grades the agent's work *after* it acts — a test, type checker, linter, fitness function, or
AI reviewer — so the agent or a human can self-correct. The feedback half of the
[harness](00-foundations/03-harness-engineering.md); its partner is [feedforward](#feedforward) (guides),
which shapes the work before. → [Sensors — Feedback](20-harness/03-sensors-feedback.md)

### Sensor ergonomics

Designing a [sensor's](#sensor) whole interface for the agent that consumes it: a remediation-rich
failure message, a [suppress-with-reason](#suppress-with-reason) escape hatch, threshold latitude with
"refactor first" stated, a query tool wrapped around big reports, and effectiveness tracked over time.
→ [Sensors — Feedback](20-harness/03-sensors-feedback.md)

### Sensor integrity

The property that passing a [sensor](#sensor) *requires the work to actually have been done*. A check the
agent can satisfy cheaply — an integration test asserting only "didn't throw" — is a false sensor: it
reports safety it can't back up. The fix is to assert the observable end-state (real persisted state,
visible behaviour), never the mere absence of an error. → [Sensors — Feedback](20-harness/03-sensors-feedback.md)

### Skill

A reusable, packaged guide the agent can load on demand (a named instruction set with metadata).
Powerful in small, curated sets; a large library is a *budget tax* — it costs tokens up front and
agents route poorly over big pools. → [Context Engineering](00-foundations/02-context-engineering.md)

### Spec-drift detection

A scheduled (or CI-triggered) scan that compares the [spec's](#specification-spec) requirement and
acceptance-criteria ids against the tests and code that claim to implement them — every requirement
traces to a test, every scenario-tagged test back to a requirement. Catches the drift no diff review
sees: each side locally green, the contract between them quietly void.
→ [Drift and Health Sensors](30-delivery/04-drift-and-health-sensors.md)

### Spec-driven development (SDD)

The spectrum of how central the spec is: **spec-first** (written before the code, discarded once the
feature ships), **spec-anchored** (kept after the task and used to evolve the feature), and
**spec-as-source** (humans edit only the spec; the code is regenerated, never touched). The playbook's
durable, audited spec is spec-anchored — and skeptical of spec-as-source.
→ [Spec — The Contract](10-lifecycle/01-spec-the-contract.md)

### Specification (spec)

The short, concrete contract between what you meant and what the agent builds: the *what* (normative
requirements and acceptance criteria, in business language), deferring the *how* to the plan. The
agent builds against it; you check the result against it. Unlike a [plan](#plan), it is **durable** —
it lives with the code, version-controlled, and is audited against the code over time so the two
don't drift. → [Spec — The Contract](10-lifecycle/01-spec-the-contract.md)

### Suppress-with-reason

An inline suppression of a sensor's rule that requires a stated justification — the legitimate way out
of a rule the agent genuinely can't satisfy. Without it, the agent games the rule silently; with it,
the exception sits in the diff as a documented judgment call a reviewer can read and veto. The escape
hatch doesn't weaken the sensor — it routes exceptions into the open.
→ [Sensors — Feedback](20-harness/03-sensors-feedback.md)

### Survivors list

The [mutation-testing](#mutation-testing) artifact you actually work with: each surviving mutant names,
in one concrete reproducible example, a behaviour no test asserts — a ready-made worklist for the agent
("kill this mutant"), far sharper than "improve the tests" and immune to the coverage illusion.
→ [Behaviour Harness](20-harness/05-behaviour-harness.md)

### Task slicing

Cutting an approved [plan](#plan) into the smallest honest units of work — each a **slice** of one
deliverable, one test, and one commit — and building them bottom-up by layer (schema → commands →
API → UI), verifying each before starting the next. Keeps every bug close to its cause and every
verified slice a checkpoint you can roll back to. → [Task Slicing](10-lifecycle/03-task-slicing.md)

### Trunk-based development

The git strategy of keeping one shared, always-releasable line of code — the **trunk** — and merging into
it continuously in small, verified slices on **short-lived branches** (hours to a day, not weeks). Long-lived
branches let work diverge and conflicts compound; merging often keeps every reconciliation and every review
small. The native rhythm of a high-throughput AI-native team — isolation gets *more* dangerous at agent
scale, not less. → [Trunk-Based Development](30-delivery/01-trunk-based-development.md)

### Zombie code

Plausible-looking code that does the wrong thing — a dead branch, an always-true condition, a
swallowed error, a handler for a case that can't occur, a path nobody asked for. It compiles and
reads as correct, so a skim waves it through; only a deliberate diff read catches it.
→ [Review and Convergence](10-lifecycle/06-review-and-convergence.md)

### Structured logging

Emitting log events as machine-readable fields (`event=deal_write_failed deal_id=412 tenant=acme`) rather
than free-text prose. The difference that makes runtime signal *queryable* — you can count, filter, and
correlate it — so a human or an agent can diagnose a production problem from evidence instead of guessing
from sentences. The log half of [observability](#observability).
→ [Observability](30-delivery/03-observability.md)

### Steering loop

The engine that *grows* a harness: the agent acts, a [sensor](#sensor) fires (or a human spots a miss), you
look at the failure, you add a [guide](#guide) or sensor, the agent acts again. You don't design the
harness up front — you can't predict the failures — you encode each real failure as a durable control so it
can't recur. A mature harness looks designed but was grown one failure at a time.
→ [Growing the Harness](50-adoption/02-growing-the-harness.md)

### Subagent

A separate, scoped agent run used for a side task — typically investigation ("go figure out how X
works") — so its work and the tokens it spends stay out of the main working thread. Keeps the main
thread's [attention budget](#attention-budget) on the task instead of the search.
→ [Code With the Agent](10-lifecycle/04-code-with-the-agent.md)

### System of record

The durable, version-controlled home for what matters — code, specs, decisions, notes in the
repository. Reviewable and legible to every future run, unlike a chat thread or a
[memory store](#memory-store), so it's the canon the agent should reason from.
→ [Context Engineering](00-foundations/02-context-engineering.md)
