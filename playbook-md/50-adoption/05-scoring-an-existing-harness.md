# Scoring an Existing Harness

> Before you grow a harness you should know where it stands — and the foundations left "how do you grade the harness?" as an open question. This is the working answer: a five-layer rubric an agent can score *any* repo against, in one pass, with evidence attached to every number. It won't tell you the harness is *good*; it tells you, layer by layer, what's present, what's missing, and what to build next.

Breadcrumb: [Playbook](../README.md) › Adoption

## Story so far

Before you grow a harness — or grow an existing one — it helps to know where it actually stands. Foundations left that question dangling on purpose: there's no settled way to grade a harness the way coverage grades code. This final chapter is the pragmatic stand-in. Point an agent at any repo and it scores five layers, each number backed by a command you can re-run. It won't hand you a gold star that says the harness is *good*; it'll tell you, plainly and with receipts, where it's blind — which is the only thing you can actually act on.

## The principle

[Harness engineering](../00-foundations/03-harness-engineering.md) ended on an honest caveat: you can
grade your *code* — coverage tells you what ran, mutation testing tells you whether the tests have teeth —
but there's no settled way to grade the *harness itself*. A quiet harness is ambiguous; silence could mean
clean work or a blind sensor. *Harness coverage* is still an open frontier.

This page is the pragmatic stand-in until the frontier closes: not a coverage metric, but a **structured
assessment**. You point an agent at a repository, and it scores five layers — the three regulation
categories from the foundations plus the two control layers that sit around them — each on a 1–10 scale,
each line backed by a concrete, reproducible measurement. The output is a **scorecard**: where this repo is
strong, where it is blind, and a ranked list of what to build next. It is the diagnostic you run *before*
[the minimum viable harness](01-minimum-viable-harness.md) on a greenfield repo, and the health check you
re-run periodically on a mature one to catch [drift](../30-delivery/04-drift-and-health-sensors.md).

The five layers:

| Layer | What it grades | Mostly… |
|---|---|---|
| **Feedforward (Guides)** | instruction files, conventions, blueprints, repo legibility — what steers the agent *before* it acts | guides |
| **Maintainability harness** | linters, formatters, type checks, dead-code and complexity sensors | computational sensors |
| **Architecture fitness** | boundary rules, dependency-direction checks, fitness functions, performance budgets | computational sensors |
| **Behaviour harness** | the spec-as-guide plus the tests that prove the code does what the business wanted | guides + sensors |
| **Inferential sensors** | AI reviewers and spec-to-code validators — semantic checks computation can't reach | inferential sensors |

The rubric and the fillable report live next to this page: the
[scorecard template](../templates/harness-scorecard.md) is what the agent fills in; the
[scoring prompt](../templates/harness-scoring-prompt.md) is what you hand the agent to run it. This page is
the *why* and the *how to read it* — the discipline that keeps the number honest.

## Why it works

**A rubric makes a vague question answerable.** "Is our harness any good?" has no answer; "score these five
layers 1–10 against these band anchors, and cite the command behind each score" does. The
[anchors in the template](../templates/harness-scorecard.md) — what makes Maintainability a 3 versus a 7 —
are what let two different repos, or two different models, produce *comparable* numbers instead of two
people's gut feelings. Without anchors a score is a vibe; with them it's a measurement.

**Evidence-per-line is what separates an audit from an opinion.** The rule the rubric enforces is that every
score cites the grep, the file, the config line, or the GitHub setting behind it. That discipline is not
bureaucracy — it is the thing that makes the audit *re-runnable* and the score *defensible*. A finding
without a command behind it can't be checked, can't be diffed next quarter, and quietly rots.

**It answers the open question well enough to act on.** This isn't mutation testing for harnesses — it
doesn't prove a sensor *can fire*. But it does reliably surface the blunt facts that matter most: a
layer scoring 1 is a layer that is *absent*, and an absent layer is a decision you want to have made on
purpose, not by omission. Most of the value is in the zeros and ones the scan makes impossible to ignore.

## How to apply it

- **Discover the repo's signals — never assume them.** The rubric grades *capabilities*, not specific
  tools. "Multi-tenancy is enforced by a checkable rule" is the capability; whether that rule is a
  `withTenant` wrapper, a Postgres row-level-security policy, or a Rails `default_scope` is for the agent to
  *find*. A template that greps for one repo's idioms in another repo's tree scores noise. The
  [prompt](../templates/harness-scoring-prompt.md) makes discovery the first phase for exactly this reason.
- **Measure tracked source only, and emit the command.** The single most important integrity rule, learned
  the hard way: a suppression count that includes `node_modules` can read 1,700 when the real figure is
  zero. Scope every measurement to version-controlled source (`git grep`, not a store-walking `grep -r`),
  and write the exact command into the report so the next run can reproduce it. A number you can't reproduce
  is worse than no number — it looks like signal.
- **Record harnessability first, because it sets the ceiling.** An untyped, boundary-less, framework-less
  repo *cannot* score a 9 on Maintainability or Architecture no matter how disciplined the team — the cheap
  sensors that earn those points [aren't available to it](../00-foundations/03-harness-engineering.md).
  Capture the stack up front so a low score reads as "low ceiling" where that's the truth, not "lazy team."
- **First run is a baseline; the value compounds on the second.** A single snapshot tells you where you
  stand. The [Orient One audits](../../audit-reports/) show where the real signal lives: the *delta* against
  a prior scorecard — what moved, what regressed, which recommendation was actioned and which was skipped
  for the third audit running. Date and keep every scorecard so the next one can diff it.
- **Score the lived layer, not just the committed one.** A control that lives entirely in org settings — an
  AI reviewer toggled on outside the repo — can vanish without a trace in version control. The scan should
  note both what the tree enforces *and* what's actually running (a `gh` pass over recent PRs catches the
  AI-reviewer that silently went quiet), and flag any control that leaves no in-repo record as fragile.
- **Run it with either model, and treat a disagreement as a finding.** The rubric is model-agnostic by
  design — Claude or Codex (or both) can fill the same template. If you run both and the scores diverge on a
  layer, that gap is itself worth reading: usually one model found a signal the other missed, and the merge
  of their evidence is a better audit than either alone.
- **Don't:** let the agent score from impressions instead of commands; reuse one repo's grep patterns on a
  different stack; report a single snapshot as a trend; mistake an advisory check (`continue-on-error`) for
  a gate; or read a low harnessability-capped score as a moral failing rather than a structural ceiling.

## In practice

You inherit a service and want to know what you're standing on. You hand the agent the
[scoring prompt](../templates/harness-scoring-prompt.md) pointed at the repo. It runs discovery first —
language, framework, build system, where the boundaries and tests live — and records that the repo is
TypeScript with clear package boundaries: harnessability is high, so the ceiling is high. Then it walks the
five layers.

It finds a strong **Feedforward** layer (a concrete instruction file, a real spec corpus) but with a
*dangling* Definition-of-Done reference — a guide that points at a file that doesn't exist — so the layer
scores an 8, not a 9, with the broken link cited by path. **Maintainability** is a 9: linter, formatter,
strict types, a pre-commit gate, all cited by config line — and the `@ts-ignore` count is reported as
`git grep '@ts-ignore'` = 0, *not* the 1,700 a naïve `grep -r` would have invented. **Architecture
fitness** scores well because a boundary rule is enforced in CI, with the workflow line quoted. The
**Behaviour** harness is the soft spot: tests exist and trace to scenarios, but a green PR ships *no
rendered-UI evidence* a reviewer can inspect — flagged as the top recommendation, with the one-line config
change that would fix it. **Inferential** scores a 1: no in-repo, gating, spec-aware AI sensor exists, and
the one AI reviewer that *was* running lives in org settings and went quiet on the last twenty PRs — a
control with no version-controlled trace.

Five numbers, every one with a command behind it, and a ranked list of what to build first. You didn't get
a verdict on whether the harness is "good" — you got a map of where it's blind, which is the thing you can
actually act on. Three months later you re-run it, diff the scorecard, and see which gaps you actually
closed.

## Anti-patterns

- **Scoring from vibes.** Assigning a 7 because the repo "feels mature" instead of citing the checks that
  earn it. An unsourced score can't be reproduced, diffed, or defended — it's an opinion wearing a number.
- **The node_modules count.** Reporting a metric measured over vendored or generated files —
  the suppression count, the line count, the dependency CVEs — as if it were the source signal. One
  unscoped `grep -r` and the whole audit loses its credibility. Scope to tracked source, always.
- **Idiom transplant.** Grepping for *this* repo's conventions (`withTenant`, an `SCN-` id scheme, a
  particular ADR layout) in a repo that's never heard of them, and scoring the absence as a gap. The rubric
  grades capabilities; the agent must discover how *this* repo realises them.
- **Snapshot-as-trend.** Presenting a first-run number as if it showed movement. A score is a point; a
  trend needs a prior scorecard to diff against. Until the second run, you have a baseline, not a direction.
- **Advisory mistaken for gate.** Counting a `continue-on-error` job, an un-required check, or an unprotected
  branch as enforcement. A sensor that can't fail the merge is a report, not a gate — and the difference is
  the whole point of the [layered gate](../20-harness/03-sensors-feedback.md).
- **Ceiling blindness.** Scoring an untyped, untested legacy repo against the same bar as a typed, modular
  one and reading the low number as negligence. Record harnessability first so the score is read against the
  ceiling the codebase actually allows.

> **That's the playbook — for now.** You came in not sure what "AI-native" even meant, and you're leaving able to score a harness and rank what to build next. Fittingly, this book is itself a harness we're still growing: every new failure we hit and every model that lands teaches us something that belongs on these pages, so expect them to keep changing. From here, the [Glossary](../glossary.md) defines every term of art, and the [scorecard template](../templates/harness-scorecard.md) + [scoring prompt](../templates/harness-scoring-prompt.md) are ready to point at your own repo. Go turn the ratchet.

---
[← Previous: Legacy and Brownfield](04-legacy-and-brownfield.md) · [Contents](../README.md) · [Next → Glossary](../glossary.md)

Related: [Harness Engineering](../00-foundations/03-harness-engineering.md) · [The Minimum Viable Harness](01-minimum-viable-harness.md) · [Growing the Harness](02-growing-the-harness.md) · [Sensors — Feedback](../20-harness/03-sensors-feedback.md) · [Drift and Health Sensors](../30-delivery/04-drift-and-health-sensors.md) · [Scorecard Template](../templates/harness-scorecard.md) · [Scoring Prompt](../templates/harness-scoring-prompt.md)
