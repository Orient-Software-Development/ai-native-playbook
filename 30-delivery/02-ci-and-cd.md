# CI and CD

> Keep two jobs separate: **continuous integration** verifies every change and is the authoritative gate that decides what may *merge*; **continuous delivery/deployment** takes the already-verified result and *releases* it to environments. Conflate them and you get the worst of both — deploy trouble blocking good merges, or unverified code shipping. Make the merge gate strict *and* fast with draft-aware checks, so iteration stays cheap.

Breadcrumb: [Playbook](../README.md) › Delivery

## The principle

[Trunk-based development](01-trunk-based-development.md) only works because something stands at the trunk and
refuses to let a broken change in. That something is the **CI gate**, and this page is about what it is, what
it is *not*, and how to keep it both trustworthy and quick.

Two distinct jobs hide under the single phrase "CI/CD," and the whole lesson is to keep them apart:

- **CI — continuous integration.** On every push and pull request, an automated pipeline *verifies the
  change*: build it, lint it, type-check it, run the [layered test suite](../20-harness/03-sensors-feedback.md).
  CI answers one question — *is this change safe to merge?* — and its verdict is the authoritative entry
  condition for the [trunk](01-trunk-based-development.md). It is the
  [un-bypassable wall](../20-harness/04-keep-quality-left.md) the keep-quality-left rule sends the heavy
  checks to.
- **CD — continuous delivery/deployment.** Once a verified change is on the trunk, CD takes *that exact
  artifact* and moves it out to environments — staging, then production. **Delivery** means the verified
  artifact is always *ready* to release at the push of a button; **deployment** means it goes out
  *automatically*. Either way, CD's input is something CI already blessed; it never re-decides correctness,
  it ships what was decided.

The rule is a clean seam between them: **CI gates the merge; CD gates the release.** They run on different
triggers, answer different questions, and fail for different reasons — and you keep them that way on purpose.

## Why it works

The seam matters because conflating the two breaks in *both* directions, and each direction is a familiar
outage.

**Bolt deployment onto the merge gate, and deploy problems block good code.** If "deploy to staging" is a
*required* step of the same pipeline that decides whether a PR can merge, then the day staging is down — a
cloud hiccup, an expired credential, a full disk — *nothing merges*, even though every change is perfectly
correct and verified. You've coupled "is this code good?" to "is the deploy target healthy?", and the second
question has nothing to do with whether the code should land. The trunk freezes for a reason that isn't about
the trunk.

**Bolt verification onto the release path — or skip it to unfreeze — and unverified code ships.** The usual
sequel to the freeze above is someone making the deploy step non-blocking under pressure, and quietly
loosening the test gate along with it to "get things moving." Now the wall has a hole: a change can reach
production without CI's verdict behind it. The gate that the whole [trunk discipline](01-trunk-based-development.md)
depends on is exactly as strong as its strictest enforced check, and a release path that can bypass CI is a
release path that ships bugs with a green badge.

**The gate is also what makes agent throughput safe.** When agents are opening many small PRs a day, no
human can be the merge gate — there's too much, too fast, and "looks fine" is
[the least reliable signal on agent code](../10-lifecycle/06-review-and-convergence.md). The *automated* gate
is the thing that scales: it decides merge-readiness deterministically, every time, no matter who or what
wrote the change. Take the gate away and throughput doesn't speed you up, it just floods the trunk faster
than anyone can check it.

**But a strict gate that's slow gets routed around.** Here's the tension this page has to resolve. An agent
iterating on a PR pushes often — and if every push triggers the full integration-and-e2e suite, the CI queue
clogs, feedback takes ages, and people start merging around the slow gate or disabling checks "just this
once." A gate everyone learns to dodge is [no gate at all](../20-harness/04-keep-quality-left.md), the same
failure the heavyweight git hook causes one stage earlier. So the gate has to be *both* strict and fast,
which is what draft-aware gating buys you.

## How to apply it

- **Draw the seam: CI decides merge, CD decides release.** Put build, lint, type-check, and the test pyramid
  in the pipeline that gates the PR. Put deploy-to-environment in a *separate* pipeline that triggers off a
  merged trunk. A deploy failure should page whoever owns the environment — it should never block an
  unrelated, correct change from merging.
- **Make CI the one authoritative, required, un-bypassable gate.** Every must-not-merge check lives here as a
  *required* status, not in a [skippable local hook](../20-harness/04-keep-quality-left.md). If it isn't
  enforced in CI, it isn't enforced.
- **Gate on drafts cheaply, gate fully on ready.** Use **draft-aware gating**: while a PR is a *draft* (still
  being iterated, often by an agent pushing repeatedly), run only the fast checks — lint, types, unit tests —
  so feedback is near-instant. Run the heavy suite — full integration, e2e, performance — when the PR is
  marked *ready for review*, and require it green before merge. The strict checks still gate the merge; you
  just stop paying for them on every work-in-progress push.
- **Promote the same artifact; never rebuild to ship.** CD should deploy the exact build CI verified, not
  compile a fresh one on the way to production. Rebuilding reintroduces the question CI already answered — "is
  this artifact good?" — and a different build is a different, unverified thing.
- **Scope the gate to the change.** A docs-only PR shouldn't drag the whole e2e suite; let the pipeline skip
  the executable gates when there's nothing for them to catch — the same scoping the
  [keep-quality-left](../20-harness/04-keep-quality-left.md) page applies to hooks.
- **Don't:** make deploy a required step of the merge gate; loosen the test gate to work around a deploy
  outage; run the full heavy suite on every draft push until the queue clogs and people route around it; or
  rebuild the artifact for production instead of promoting the verified one.

## In practice

A team sets up "a CI/CD pipeline": one job, triggered on every PR, that runs *build → test →
deploy-to-staging*, and the PR can't merge until the whole job is green.

**What actually happens — direction one.** A Tuesday, staging's database is briefly down for maintenance.
The deploy step fails, so the pipeline is red, so *no PR can merge* — including three small, fully-correct
slices that have nothing to do with staging. The trunk is frozen by a problem that isn't a code problem at
all. The team is now blocked on an environment outage while perfectly good, verified changes pile up behind a
red badge.

**Direction two.** To unfreeze, someone marks the deploy step non-blocking — and, in the same harried edit,
flips the test job to non-required so "the pipeline goes green." The trunk unfreezes. It also now has a hole:
that afternoon an agent's PR with a real failing integration test merges anyway, because the gate that would
have caught it is no longer required. A bug ships behind a green checkmark, and nobody notices until it's in
production.

**The fix.** Split the one job along the seam. **CI** — build, lint, type-check, test pyramid — runs on every
PR and is the single *required* gate to merge; it never touches an environment, so a staging outage can't
freeze the trunk. **CD** — deploy to staging, then production — runs *after* merge as its own pipeline; if it
fails, it pages the environment owner and the verified artifact simply waits, but merges keep flowing. Then
they add **draft-aware gating**: while an agent iterates on a draft PR, only lint/types/unit run, so each
push gets feedback in seconds; the full integration-and-e2e suite runs the moment the PR is marked ready, and
merge requires it green. Now the gate is strict (nothing merges unverified), fast (drafts don't pay the heavy
suite), and decoupled (deploy trouble never blocks a good merge).

The lesson the example carries: "CI/CD" is two jobs, not one. CI is the authoritative answer to *may this
merge?* and must be strict, required, and fast enough that nobody dodges it; CD is the path that ships what CI
already blessed and must be free to fail without freezing the trunk. Keep the seam and the trunk stays both
releasable and unblocked; blur it and you get frozen merges, shipped bugs, or both.

## Anti-patterns

- **The merge gate that deploys.** Deployment wired as a required step of the gate that decides merge, so an
  environment outage freezes the trunk for changes that are perfectly correct.
  ([Failure modes](../40-anti-patterns/01-failure-modes.md))
- **The release path that skips CI.** A deploy route — or a loosened gate under pressure — that lets code
  reach production without CI's verdict behind it. The wall with a hole ships bugs with a green badge.
- **The slow universal gate.** Running the full heavy suite on every draft push until the queue clogs and the
  team starts merging around it or disabling checks — a
  [strict gate made worthless by being slow](../20-harness/04-keep-quality-left.md).
- **The rebuild-to-ship.** Compiling a fresh artifact on the way to production instead of promoting the one
  CI verified — shipping a different, unverified thing than the one that passed the gate.

---
[← Previous: Trunk-Based Development](01-trunk-based-development.md) · [Contents](../README.md) · [Next → Observability](03-observability.md)

Related: [Keep Quality Left](../20-harness/04-keep-quality-left.md) · [Sensors — Feedback](../20-harness/03-sensors-feedback.md) · [Trunk-Based Development](01-trunk-based-development.md) · [Observability](03-observability.md) · [Failure Modes](../40-anti-patterns/01-failure-modes.md)
