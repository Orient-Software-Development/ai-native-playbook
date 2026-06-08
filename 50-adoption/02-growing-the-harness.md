# Growing the Harness

> A harness is grown, not designed — one real failure at a time. When the agent goes wrong, don't re-prompt and hope; ask what control was missing, encode it once so it can't recur, and bank the harnessability that makes the next control cheaper. And when a new model lands, prune the scaffolding it no longer needs.

Breadcrumb: [Playbook](../README.md) › Adoption

## The principle

The [last page](01-minimum-viable-harness.md) got you a small-but-whole harness in week one: one
guide, a few sensors, a gate, one behaviour test. This page is what you do for every week after — how
that minimum grows into the full harness the rest of this playbook describes, without ever stopping to
"design" it.

The engine is the **steering loop**, introduced back in
[harness engineering](../00-foundations/03-harness-engineering.md): the agent acts, a sensor fires (or a
human spots a miss), you look at the failure, you add a guide or a sensor, the agent acts again. You don't
sit down and predict every way the agent will go wrong — you *can't*. You watch it go wrong once, then
encode the fix as a durable control so that exact failure can't recur. A mature harness looks designed but
was actually grown, one failure at a time.

Growing has two motions, and the second is the one teams forget:

- **Grow** — add a guide or sensor in response to a real, observed failure.
- **Prune** — remove a control once the model has grown into the job it was scaffolding. A harness that
  only ever accumulates turns into the [skill-library tax](../00-foundations/02-context-engineering.md):
  bloat that costs budget and attention for safety it no longer buys.

Underneath both runs a slower investment: **harnessability** — the property (from
[harness engineering](../00-foundations/03-harness-engineering.md)) that strong types, clear module
boundaries, established frameworks, and real test coverage make a codebase *cheaper to control*. Every
unit of harnessability you bank makes every future guide and sensor easier to write. You grow controls in
response to failures; you grow harnessability on purpose, because it compounds under everything else.

## Why it works

**"Try harder" doesn't change anything.** The most common reaction to an agent failure is to re-prompt —
say it again, more firmly. But the next run starts from the same harness that produced the failure, so it
reproduces the failure. Re-prompting fixes *this* instance, by hand, at the cost of your attention, and
buys nothing for the next one. Encoding a control changes the *system*: the rule now applies on every run,
to every change, automatically. A fix you make once and that holds forever is worth incomparably more than
a fix you re-make every session.

**You grow from failures because you can't predict them.** A complete harness designed up front would be
guessing at which controls the codebase needs — and it would be wrong, because the failures that actually
happen depend on this codebase, this domain, this model. The repo *teaches* you its weak spots by failing
in them. The steering loop is just the discipline of listening: treat every recurring failure as a
specification for the control that was missing.

**The two investments compound on top of each other.** Each control makes the next run more reliable; each
unit of harnessability makes the next control cheaper to add. Bank a type system early and a whole class of
future bugs is caught for free, before you ever write a sensor for it. That's why a harness gets *easier*
to grow over time rather than harder — the opposite of how un-harnessed codebases age.

**And the harness has to track the model, not just the codebase.** Every control you add encodes an
assumption: *the model can't be trusted to do this on its own.* Models improve, and those assumptions go
stale — the verbose step-by-step guide you wrote for a task the new model now handles natively isn't
neutral, it's dead weight crowding the context and masking what's actually load-bearing. Pruning isn't
tidying; it's keeping the harness honest about what the current model genuinely needs.

## How to apply it

- **On a failure, ask "what control was missing?" — not "how do I re-prompt?"** That question is the whole
  loop. The answer is always a durable control, never a sharper one-off prompt.
- **Match the fix to the failure.** If the agent *didn't know* the expectation, it needs a
  [guide](../20-harness/01-guides-feedforward.md) (feedforward). If it knew but did it wrong, or could
  regress later, it needs a [sensor](../20-harness/03-sensors-feedback.md) (feedback). Usually you want
  both — the guide so it gets it right next time, the sensor so a slip can't merge — and you
  [reach for the cheap computational check first](../00-foundations/03-harness-engineering.md), a lint or a
  type rule before an AI reviewer.
- **Encode it once so it applies everywhere.** The payoff of a control is that it covers the whole
  codebase at once. Don't fix the inlined query in this one diff and move on — write the rule that fails the
  build for *any* such diff, forever. Hand-fixing each instance is doing a sensor's job by hand.
- **Use the agent to build the harness.** Encoding a control is itself one of the best agent tasks there
  is — it's mechanical and well-specified, and the agent that keeps tripping a rule is well placed to
  author the check for it. So hand it the work: *draft the [structural test](../20-harness/03-sensors-feedback.md)*
  that pins a boundary; *turn an observed pattern into a lint rule* ("you keep doing X — write me the rule
  that forbids it"); *scaffold a custom linter* for a project-specific smell no off-the-shelf tool knows;
  *write the [how-to guide](../20-harness/01-guides-feedforward.md)* from reading the code it just
  navigated. You describe the control and [review the result](../10-lifecycle/06-review-and-convergence.md);
  the agent writes it. The harness grows the harness — and because you still review every control before it
  lands, you get the leverage without ceding the judgement.
- **Bundle a recurring shape into a harness template.** Once you've grown a solid harness for one service
  of a given shape — a CRUD API, an event processor, a dashboard — don't hand-rebuild it for the next one.
  Bundle its guides and sensors (the instruction-file conventions, the lint and fitness-function config,
  the [scaffolding](../20-harness/01-guides-feedforward.md)) into a reusable *harness template* that the
  next service of that shape starts from already harnessed. The grown harness becomes the
  [blueprint](../20-harness/01-guides-feedforward.md) for its whole topology. The one cost to budget for: a
  template is a shared dependency, so the day you improve it the instances start drifting from it — plan the
  upkeep to pull fixes upstream and push them back out, or each copy quietly diverges into its own thing.
- **Bank harnessability as you go.** When you touch an untyped corner, type it; when a boundary is implicit,
  make it a checkable rule. These are the investments that [pay off twice](../00-foundations/03-harness-engineering.md)
  and make every later control cheaper — covered as a practice in
  [repo structure and legibility](../20-harness/02-repo-structure-and-legibility.md).
- **Prune when a new model lands.** Treat a model upgrade as a reason to *re-examine* the harness, not just
  to expect free wins. Remove one control at a time and watch what regresses: if output holds without it,
  it was scaffolding the model has outgrown — drop it and reclaim the budget. If something breaks, it was
  load-bearing — put it back. Stress-test your assumptions one at a time so you can tell which is which.
- **Widen autonomy as the harness earns it.** Don't hand the agent the whole lifecycle on day one. Start it
  on a narrow, well-harnessed slice where a miss is caught cheaply, and extend its scope only as the
  controls around a zone prove they catch the agent's mistakes. Autonomy is something the harness *earns*,
  zone by zone — the deliberate allocation of it is the [responsible team's job](03-responsible-team-and-ai-debt.md).
- **Don't:** re-prompt the same failure twice without adding a control (you're paying attention for nothing);
  hand-patch each instance of a rule the agent keeps breaking (encode it); only ever add and never prune
  (the harness bloats into a tax); grant end-to-end autonomy before the harness can catch the agent in that
  zone.

## In practice

A team is past week one. Their minimum viable harness is holding, and now they're letting the agent build
features against it.

**The loop in one turn.** The agent ships a feature, and a reviewer notices the API response includes an
internal field that should never leave the system — an email-verification token, say. They flag it, the
agent fixes that one response, done. A week later it happens again on a different endpoint. The instinct is
to add a firmer line to the prompt: *be careful not to leak internal fields.* That's the trap — the third
endpoint leaks too, because nothing in the system changed. The steering-loop move is different: ask *what
control was missing?* The answer is two controls. A **guide** — a convention naming which fields are
internal and never serialised — so the agent knows the expectation up front. And a **sensor** — a
computational check that fails the build if a response shape includes a field on the internal list — so a
slip can't merge even when the agent forgets. They have the agent write the check, review it, and merge it.
The leak never recurs, on any endpoint, ever. One failure, observed once, became a permanent property of
the codebase.

**The prune, months later.** Early on, the same team had written a long, careful guide walking the agent
step by step through wiring a new database migration, because the model kept getting the order wrong. A
model upgrade lands. On a hunch, they remove that guide for one migration and watch: the agent gets it
right on its own, every time. The scaffolding was for a weakness the new model doesn't have. They delete
the guide, the context budget gets a little leaner, and the harness gets a little more honest about what
the current model actually needs.

The lesson the example carries: you don't author a harness, you *steer* one. Every recurring failure is the
spec for a missing control; encode it once and it's gone for good, everywhere. And every control is a
standing bet on a model weakness — so as the model grows, you prune the bets that no longer pay, and the
harness stays the smallest set of controls that does the whole job. Grown from failures, pruned by
upgrades, compounding on harnessability underneath.

## Anti-patterns

- **Try harder.** Re-prompting the same failure instead of encoding a control. The harness is unchanged, so
  the failure recurs — you've spent attention and bought nothing durable.
- **Hand-fixing each instance.** Patching the same class of mistake in diff after diff instead of writing
  the one rule that catches all of them. You're doing a sensor's job by hand, and it still recurs the moment
  you look away.
- **The write-only harness.** Only ever adding controls, never pruning. Stale guides for weaknesses the
  model outgrew pile up into the [skill-library tax](../00-foundations/02-context-engineering.md) — budget
  and attention spent on safety no longer bought, masking which controls are actually
  [load-bearing](../40-anti-patterns/01-failure-modes.md).
- **Big-bang autonomy.** Granting the agent end-to-end scope before the harness around that zone has earned
  it. Autonomy is earned by controls that demonstrably catch the agent's mistakes, not granted on optimism.
- **Designing it all up front.** Trying to predict and build the complete harness in one sitting — the same
  mistake as the [big-bang minimum harness](01-minimum-viable-harness.md), a step later. You can't foresee
  the failures; let the repo show you.
- **The drifting template.** Bundling a harness template for a recurring service shape
  and then never reconciling the instances with it — so each copy diverges, a shared fix never propagates,
  and the "template" becomes three subtly different harnesses nobody can maintain. A template is a
  dependency; budget its upkeep or don't make one.

---
[← Previous: The Minimum Viable Harness](01-minimum-viable-harness.md) · [Contents](../README.md) · [Next → The Responsible Team and AI Debt](03-responsible-team-and-ai-debt.md)

Related: [Harness Engineering](../00-foundations/03-harness-engineering.md) · [The Minimum Viable Harness](01-minimum-viable-harness.md) · [Legacy and Brownfield](04-legacy-and-brownfield.md) · [Guides — Feedforward](../20-harness/01-guides-feedforward.md) · [Sensors — Feedback](../20-harness/03-sensors-feedback.md) · [Repo Structure and Legibility](../20-harness/02-repo-structure-and-legibility.md) · [Drift and Health Sensors](../30-delivery/04-drift-and-health-sensors.md) · [The Responsible Team and AI Debt](03-responsible-team-and-ai-debt.md)
