# Drift and Health Sensors

> Some decay is caused by no single change, so no single change can catch it. **Drift and health sensors** are the controls that run on a *clock* instead of on a diff — dead-code scans, dependency and vulnerability checks, coverage and mutation *trends*, architecture-drift "janitor" runs — to catch the rot that accumulates between changes rather than inside one. Without them the harness sees every commit and misses the slow slide underneath them all.

Breadcrumb: [Playbook](../README.md) › Delivery

## The principle

Every sensor so far in this playbook fires on a **change**. A [test, a linter, a type
check](../20-harness/03-sensors-feedback.md) runs because a diff arrived; it grades *that diff* and
either blocks it or lets it through. That model is the backbone of the harness — but it has a blind
spot, and the blind spot is structural: **a change-triggered sensor can only catch problems a change
causes.** Some problems aren't caused by any one change. They accumulate across many, or they appear
with no local change at all because the world outside the repo moved. No diff is to blame, so no
diff-triggered sensor ever fires.

The trigger for catching those is not a change — it's **time**. A drift or health sensor runs on a
schedule (nightly, weekly, on a cron), out of band from the change lifecycle, and asks a question no
single pull request could answer: *what has quietly rotted since we last looked?* Four kinds are
worth standing up:

- **Dead-code detection.** Code that nothing calls anymore — a function whose last caller was deleted
  three changes ago. No single one of those changes "introduced" dead code; it emerged from their sum.
  A scheduled scan finds the orphans and proposes their removal.
- **Dependency and vulnerability scanning.** A library you depend on gets a CVE disclosed *upstream*,
  months after the code that uses it merged green. Nothing in your repo changed — the risk appeared
  anyway, from outside. A cadence scanner (the kind that opens "bump this dependency" pull requests)
  is the only sensor positioned to notice, because it re-checks the world on a clock rather than
  waiting for a diff that will never come.
- **Test-quality trend monitoring.** [Coverage](../20-harness/03-sensors-feedback.md) and
  [mutation score](../20-harness/05-behaviour-harness.md) tell you something *as a direction over
  time* that they can't tell you as a single number. Coverage that drifts down a point a week, or a
  mutation score that erodes as new code arrives faster than tests for it, is a real signal — and
  it's invisible to any one change, each of which moves the number too little to gate on. Tracking the
  *trend* turns a slow slide into a visible line.
- **Architecture-drift scans.** The codebase slowly diverging from its intended shape — boundaries
  blurring, layers leaking, a module accreting responsibilities it was never meant to hold. A
  scheduled "janitor" run (sometimes called garbage collection at agent scale) scans for that drift
  and *proposes* fixes, so entropy gets a standing counter-force instead of building until someone
  notices in disgust.

A fifth member you've already met: [**observability**](03-observability.md) — runtime sensors
watching production. It belongs to this same family (a sensor outside the change lifecycle), but it
runs continuously on live traffic rather than on a schedule against the repo, so it has
[its own page](03-observability.md). This page is about the *scheduled* scans of the codebase itself.

## Why it works

**No diff is responsible, so no diff-gate can help.** This is the whole argument. Picture dead code:
change A adds a helper, change B is the last caller, change C deletes B's caller. After C, the helper
is dead — but *C never touched the helper*. There is no diff whose review could have caught it,
because the death emerged from three changes none of which was individually wrong. The same logic
holds for an upstream CVE (no local diff at all), for an eroding coverage trend (each change drops it
imperceptibly), and for architecture drift (each boundary-crossing looked locally reasonable). When
the cause is *cumulative* or *external*, the only sensor that can see it is one whose trigger is also
not a single change. That's what "runs on a clock, not on a diff" buys you.

**A trend says what a snapshot can't.** A single coverage number is nearly meaningless — 80% of what,
and is that rising or falling? The same number measured every week draws a *line*, and the line is the
signal: flat is fine, climbing is great, and a steady decline is a warning that test-writing is
falling behind code-writing well before any individual change would trip a gate. [Mutation
testing](../20-harness/05-behaviour-harness.md) is doubly worth tracking this way — run once it tells
you the suite's strength today; tracked over time it tells you whether that strength is being
*maintained* as the codebase grows. One-shot is a photo; a trend is the weather.

**This is where the AI-native version earns its keep.** A drift scan that only produces a dashboard
produces nothing — someone has to read it, and nobody does. But the *janitor* doesn't have to stop at
a report: a [scheduled agent run](../50-adoption/02-growing-the-harness.md) can scan for the drift
*and draft the fix* — open the pull request that deletes the dead code, bumps the dependency, or
re-draws the leaked boundary — so the finding arrives as reviewable work, not a line item nobody owns.
The agent proposes; the [gate and a human still dispose](../10-lifecycle/06-review-and-convergence.md).
That keeps the cheap, non-deterministic scan advisory (where it belongs) while still turning its
findings into action automatically. The same move that [grows the
harness](../50-adoption/02-growing-the-harness.md) — using the agent to author controls — maintains it.

**Cheap insurance against silent decay.** None of these sensors is on the critical path of a change,
so none of them slows anyone down. They run while you sleep. The cost is a little compute on a
schedule and the discipline to act on what they surface; the thing they buy is that the codebase's
slow problems announce themselves on a cadence instead of surfacing all at once, years later, as "why
is everything like this?"

## How to apply it

- **Run them on a clock, out of band.** A nightly or weekly job (CI cron, scheduled action) — never a
  pre-commit or pre-push hook. These scans are slow and their findings aren't a single author's fault,
  so [putting them on the change gate](../20-harness/04-keep-quality-left.md) would block the wrong
  person for a problem they didn't cause. Out-of-band is the whole point.
- **Make every finding land as action, not a dashboard.** The failure mode of every health sensor is
  the report nobody reads. Wire each one to *produce work*: the dependency scanner opens a bump PR, the
  dead-code scan opens a deletion PR, the drift scan files an issue or a proposed diff. A finding with
  an owner and a next step gets fixed; a finding on a dashboard gets admired and forgotten.
- **Track trends, not snapshots.** Record coverage and mutation score over time and look at the
  *slope*. A single threshold gate on coverage invites [gaming with low-value
  tests](../20-harness/03-sensors-feedback.md); a trend line you watch tells you the honest direction
  without pretending a number is a quality bar.
- **Keep the non-deterministic scans advisory.** A janitor that *auto-merges* its own architecture
  "fixes" is a gate that can't be argued with making semantic calls it isn't qualified to make
  unsupervised. Let the agent *propose* — open the PR — and route it through the same
  [review](../10-lifecycle/06-review-and-convergence.md) and gate as any other change. Propose-then-dispose,
  not propose-and-merge.
- **Budget time to act, or don't run the scan.** A scanner whose PRs pile up unmerged for months is
  worse than none: it manufactures noise and trains the team to ignore a whole category of signal. If
  you can't commit to triaging the output on the same cadence you run it, you don't have a sensor — you
  have a guilt generator. Schedule the *acting*, not just the *scanning*.
- **Watch for the silent scan.** A drift sensor that never finds anything is either genuinely good news
  or a [sensor that isn't actually detecting](../20-harness/03-sensors-feedback.md). Periodically
  confirm it can still fire — the same discipline you'd apply to a test that always passes.
- **Don't:** put these on a git hook; let findings accumulate on a dashboard with no owner; gate hard
  on a coverage *number* instead of watching the *trend*; auto-merge a janitor's semantic fixes; or run
  a scan you've no intention of acting on.

## In practice

A team ships steadily for a year. Every change is [verified](../10-lifecycle/05-verify-proof-not-vibes.md),
[gated green](02-ci-and-cd.md), [merged on a short-lived branch](01-trunk-based-development.md). The
change lifecycle is healthy. And yet.

**Without drift sensors.** A JSON-parsing library the app has used since month one has a vulnerability
disclosed against it in month nine. No one notices: nothing in the repo changed, so no CI run fired,
and the library sits quietly exploitable in production. Meanwhile the agent, building fast, has left a
trail — three helpers whose callers were refactored away, a whole module reachable by nothing. And
coverage, 84% at launch, has slid to 71% one imperceptible drop at a time, because features arrived
faster than tests and no single PR ever dropped it enough to trip the gate. None of this is anyone's
*fault* in a way a diff review could have caught — each change was locally fine. It surfaces all at
once in month twelve, as a security audit that finds the CVE, a refactor that trips over the dead code,
and a senior engineer asking why the test suite feels so thin. A year of silent decay, billed in one
lump.

**With drift sensors.** The same team runs four scheduled jobs. The dependency scanner re-checks the
world every night; the morning the CVE is disclosed it opens a PR bumping the library to the patched
version, and it's reviewed and merged before lunch — no local change ever needed to happen, because the
sensor's trigger was the calendar, not a diff. A weekly dead-code scan opens a small deletion PR each
time an orphan appears, so the helpers never accumulate. A coverage-and-mutation trend, posted weekly,
shows the line ticking down in month three — early enough that the team makes test-writing part of
every slice again and the slide reverses while it's still gentle. And a monthly architecture-drift
janitor — an agent run — spots a module starting to absorb responsibilities from across a boundary and
opens a PR proposing the split, which a human reviews and merges. Nothing surfaces in one lump in month
twelve, because nothing was allowed to accumulate silently for twelve months.

The lesson the example carries: the decay existed in both worlds — the difference was purely whether
anything was *looking between the changes*. The change lifecycle, however healthy, is structurally
blind to problems no single change causes; drift and health sensors are the clock-driven counterpart
that catches exactly those — and in an AI-native team they don't just report the rot, they open the PR
that fixes it.

## Anti-patterns

- **The dashboard nobody reads.** A drift report, a coverage graph, a vulnerability list — generated
  on a schedule, owned by no one, actioned never. A sensor whose output doesn't become work is just
  decoration. ([Failure modes](../40-anti-patterns/01-failure-modes.md))
- **The snapshot mistaken for a trend.** Running mutation testing or coverage *once* and treating the
  number as settled, when the signal that matters is the *direction* it moves as the codebase grows.
  ([Behaviour harness](../20-harness/05-behaviour-harness.md))
- **The auto-merging janitor.** Letting a non-deterministic drift scan merge its own semantic "fixes"
  with no review — a [gate that can't be argued with](../10-lifecycle/06-review-and-convergence.md)
  making architectural calls unsupervised. Propose, don't dispose.
- **The bot whose PRs pile up.** A dependency or dead-code scanner whose output is never triaged, so
  the unmerged PRs become noise and the team learns to ignore the whole channel — the
  [alert that cried wolf](03-observability.md), in PR form. Budget the acting, not just the scanning.
- **Drift on the diff gate.** Bolting a slow, cumulative-by-nature scan onto a pre-push hook, so it
  blocks the wrong author for rot they didn't introduce — and gets [disabled with
  `--no-verify`](../20-harness/04-keep-quality-left.md) by the end of the week.

---
[← Previous: Observability](03-observability.md) · [Contents](../README.md) · [Next → Failure Modes](../40-anti-patterns/01-failure-modes.md)

Related: [Observability](03-observability.md) · [Sensors — Feedback](../20-harness/03-sensors-feedback.md) · [Behaviour Harness](../20-harness/05-behaviour-harness.md) · [Keep Quality Left](../20-harness/04-keep-quality-left.md) · [Growing the Harness](../50-adoption/02-growing-the-harness.md) · [Review and Convergence](../10-lifecycle/06-review-and-convergence.md)
