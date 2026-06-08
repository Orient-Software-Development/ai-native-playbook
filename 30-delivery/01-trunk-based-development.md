# Trunk-Based Development

> Keep one shared line of code — the **trunk** — and merge into it constantly, in small verified slices, behind a green gate. Long-lived branches are where work goes stale and conflicts compound; at agent throughput they rot faster than ever. A high-performing AI-native team's velocity comes from *integrating continuously*, not from working in isolation.

Breadcrumb: [Playbook](../README.md) › Delivery

## The principle

The earlier chapters built the work down to a single verified
[slice](../10-lifecycle/03-task-slicing.md): one deliverable, one test, one commit. This chapter is about
what happens to that slice next — how it ships, and how it ships when many people and many agents are
producing slices at once. The first decision is the git strategy, and the answer is **trunk-based
development**.

The **trunk** is the one shared, always-releasable line of code everyone integrates into (often `main`,
sometimes called `staging` or an integration branch). Trunk-based development is the discipline of keeping
that line healthy by merging into it *continuously* and in *small pieces*:

- **Short-lived branches.** A branch lives hours to a day or two, not weeks. You cut it from the trunk,
  land one slice, merge it back, and delete it. It never has time to drift far from the trunk it came from.
- **Small slices, merged behind a green gate.** Each merge is one verified slice that passed the
  [CI gate](02-ci-and-cd.md) — the automated checks that decide whether a change may land. Nothing merges
  on a promise; it merges because the gate proved it.
- **The trunk stays releasable.** Because every merge is small and verified, the trunk is always in a
  shippable state. Work that's too big to finish in one slice is hidden behind a **feature flag** — a
  runtime switch that keeps the half-built code dark in production — rather than parked on a branch off to
  the side.

The alternative — a long-lived **feature branch** that absorbs a whole feature over days or weeks before one
big merge — is the thing this practice exists to avoid. **Continuous integration** in its original sense is
exactly this: not "we have a CI server," but *the team actually integrates its work continuously* instead of
saving it up.

## Why it works

Two costs grow non-linearly with how long a branch lives, and trunk-based development exists to keep both
small.

**Merge conflicts compound.** While your branch sits open, the trunk moves underneath it — other people's
merges change the very files you're editing. The longer you wait, the more the two diverge, and a **merge
conflict** (two changes to the same lines that git can't reconcile automatically) stops being a one-line fix
and becomes an archaeology project: you're re-reading a week of someone else's changes to figure out how to
combine them with yours. Merge daily and each reconciliation is trivial; merge monthly and the merge itself
is a risky, untested change nobody planned for. Small and frequent isn't just tidier — it's *cheaper per
unit of code*, because conflict cost scales with divergence, and divergence scales with time.

**Unmerged work goes stale.** Code that isn't on the trunk hasn't really been integrated — it hasn't met
the other changes it will have to live with. A branch that looks finished in isolation can be subtly broken
against the trunk it'll merge into, and you won't find out until the big merge, which is the worst possible
moment: late, large, and all at once. Integrating continuously means every slice meets reality the day it's
written, while the context is fresh and the fix is small.

**Agents amplify both forces — in both directions.** This is the AI-native turn. On the upside, agent
throughput means slices land faster and more often, which is *exactly* the cadence trunk-based development is
built for — many small, verified merges a day is its native rhythm, not a strain on it. On the downside, the
same throughput makes a long-lived branch far more dangerous: a branch where an agent has been generating
volume for a week is a large pile of [plausible-but-unreviewed code](../10-lifecycle/06-review-and-convergence.md)
diverging from a trunk that other agents have been moving the whole time. The merge is enormous, the review
is hopeless, and the conflicts are everywhere. The faster your team produces code, the *more* it needs the
trunk to be the constant meeting point — isolation doesn't get safer at scale, it gets exponentially worse.

So the velocity of an AI-native team doesn't come from agents working alone on big branches. It comes from
many small, verified changes converging on one trunk continuously — the merge philosophy shifts from "a few
large PRs a sprint" to "many small PRs a day," and the trunk is what makes that throughput safe instead of
chaotic.

## How to apply it

- **Branch from the trunk, return to it fast.** Cut a short-lived branch, land one slice, merge it back the
  same day if you can, and delete the branch. If a branch has been open longer than a couple of days, that's
  a signal the work wasn't sliced small enough — go back and [cut it smaller](../10-lifecycle/03-task-slicing.md).
- **One slice per branch, one branch per slice.** The [task slice](../10-lifecycle/03-task-slicing.md) and
  the branch are the same unit. A branch that's accumulating "while I'm here" extras is on its way to
  becoming long-lived — keep it to the one deliverable it was cut for.
- **Never merge on a promise — merge behind the gate.** The trunk stays releasable only because every merge
  passed the [authoritative CI gate](02-ci-and-cd.md). A green gate is the entry condition for the trunk,
  full stop; no "I'll fix the test after merge."
- **Hide unfinished work behind a flag, not on a branch.** When a feature genuinely can't land in one slice,
  merge the incomplete pieces to the trunk dark — behind a feature flag — so they keep integrating with
  everyone else's work instead of diverging on a side branch. The trunk stays releasable because the flag is
  off; the work stays integrated because it's on the trunk.
- **Pull from the trunk often.** Rebase or merge the trunk into your short-lived branch frequently so you
  meet conflicts in ones and twos, while they're trivial, instead of in a wall at the end.
- **Review at the slice, not at the feature.** A small slice is a
  [reviewable diff](../10-lifecycle/06-review-and-convergence.md); a week-old branch is a wall nobody reads
  deliberately. Continuous merging is what keeps every review small enough to actually do.
- **Don't:** keep a "develop everything here" branch alive for a whole feature; batch a sprint of work into
  one heroic merge; let an agent run for days on an isolated branch and merge the result unread; or merge a
  red or skipped gate "to unblock."

## In practice

A team picks up the *per-user deal priority* feature — the same four-layer feature sliced in the lifecycle
chapter — and an agent is doing most of the typing.

**The long-lived-branch way.** They cut one `feature/deal-priority` branch and let the agent build the whole
thing on it: schema, commands, query, and UI, over four days, all merged at the end. Meanwhile two other
agents are landing changes on the trunk — one refactors the deals-list query the priority feature also
touches, another renames a column in a nearby table. None of that reaches the feature branch, because the
feature branch is off on its own. On day four the merge is opened: hundreds of lines of fluent agent code, a
stack of conflicts in the exact query both agents edited, and a column rename the branch never saw. The
review is a skim because the diff is too big to read deliberately — which is precisely where the
[privacy bug](../10-lifecycle/06-review-and-convergence.md) hides. The "merge" is now its own risky,
untested change, landing late and all at once.

**The trunk-based way.** The same four slices each get their own short-lived branch and merge the day
they're written, behind the green gate:

```text
Mon AM   branch → schema slice      → gate green → merge → trunk releasable
Mon PM   branch → commands slice    → gate green → merge → trunk releasable
Tue AM   branch → query slice       → gate green → merge → trunk releasable
Tue PM   branch → UI slice (flagged)→ gate green → merge → trunk releasable
```

When the other agent's query refactor lands Monday afternoon, the priority feature's query slice — opened
Tuesday morning — is cut from a trunk that *already has it*. There's no four-day divergence to reconcile;
the one place the two changes meet is resolved the same morning, in a diff small enough to read. Each slice
is reviewed at the slice, against real state, while it's fresh. The UI slice merges behind a feature flag so
the trunk stays shippable even though the feature isn't announced yet. By Tuesday evening the whole feature
is on the trunk, integrated, reviewed, and releasable — and at no point was there a big scary merge, because
there was never a big branch to merge.

The lesson the example carries: the danger was never the amount of code — it was the amount of code *kept out
of the trunk at once*. Agent throughput makes that pile grow faster, so the discipline that keeps it small —
short-lived branches, small slices, merge behind the gate — is what lets a fast team stay a *safe* fast team.

## Anti-patterns

- **The long-lived feature branch.** A whole feature developed in isolation for days or weeks, then merged
  in one heroic, conflict-ridden, unreviewable diff — the failure this entire practice prevents. At agent
  throughput the pile grows faster and the merge is worse. ([Failure modes](../40-anti-patterns/01-failure-modes.md))
- **The agent left to run on a branch.** Letting an agent generate volume for days on an isolated branch, so
  the merge is enormous, the review is a skim, and the [zombie code](../10-lifecycle/06-review-and-convergence.md)
  ships behind plausibility.
- **The merge-on-a-promise.** Landing a change on the trunk with a red or skipped [gate](02-ci-and-cd.md)
  "to unblock," so the always-releasable trunk quietly stops being releasable and nobody knows when it broke.
- **The branch that won't die.** A "develop" or "integration" branch that everyone works off of for a
  release, recreating the long-lived-branch problem one level up — divergence from the real trunk, just with
  more people on it.

---
[← Previous: Behaviour Harness](../20-harness/05-behaviour-harness.md) · [Contents](../README.md) · [Next → CI and CD](02-ci-and-cd.md)

Related: [Task Slicing](../10-lifecycle/03-task-slicing.md) · [Review and Convergence](../10-lifecycle/06-review-and-convergence.md) · [CI and CD](02-ci-and-cd.md) · [Keep Quality Left](../20-harness/04-keep-quality-left.md) · [Failure Modes](../40-anti-patterns/01-failure-modes.md)
