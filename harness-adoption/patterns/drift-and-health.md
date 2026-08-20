# Pattern: drift and health sensors (scheduled)

## Intent

Everything else in this library fires on a **change**. Some decay has no
diff to trigger on: dependencies rot in place, dead code accumulates,
docs and specs drift from the code they describe, suite strength slides
one innocuous change at a time. The trigger for catching those is
**time** — jobs on a schedule (nightly, weekly, cron in CI), out of band
from the change lifecycle, asking: *what has quietly rotted since we
last looked?* See
`../../playbook-md/30-delivery/04-drift-and-health-sensors.md`.

This is a **later** control: it presupposes CI and the minimum viable
harness. Adopt one scan at a time, like everything else.

## The scans, in adoption order

1. **Dependency freshness.** Nightly re-check of the dependency set
   against advisories and releases; opens a PR bumping to the patched
   version. The trigger is the calendar, not a diff — a CVE disclosed
   overnight gets a fix PR by morning.
2. **Dead-code detection.** Weekly scan for code nothing calls; opens a
   small deletion PR. Dead code misleads the agent (it reads it as
   convention) — deleting it is harness work, not tidying.
3. **Spec-drift detection.** Compare the spec's acceptance-criteria ids
   (the `AC-n` the template carries) against the tests and code that
   claim to implement them: every criterion traces to at least one test,
   every scenario-tagged test back to a criterion, the API surface
   matches the spec's contract. Catches the drift no diff review sees —
   each side locally green, the contract between them quietly void.
   Companion rule: a change to the *spec itself* gets classified before
   merge — **additive** / **compatible** / **breaking** / **ambiguous**
   — and the last two need a human sign-off, because they redefine the
   contract (the agent can propose the classification).
4. **Suite-strength trend.** The full mutation run is far too slow for
   any gate — run it on a schedule and track the *score as a trend*
   (incremental runs on changed files live in the change loop; see
   `behaviour-test.md`). A slow slide in suite strength gets noticed
   even when no single change caused it.
5. **Sensor-effectiveness review.** Log what each sensor catches, run
   over run; surface the silent ones (mastered or broken — test they
   still fire) and the constantly-failing ones (a better guide is
   needed there).

## Blocking behaviour

**Advisory by design.** Scheduled scans *propose* — they open PRs and
file reports; they do not auto-merge. A janitor that merges its own
fixes is a gate that can't be argued with. The proposals route through
the same review gate as any change. The one blocking piece is the
spec-change classifier: breaking/ambiguous spec changes block on human
sign-off.

## Assessment signal

- Any `schedule:`/cron-triggered CI workflows?
- `dependabot.yml` / `renovate.json` present and PRs acted on (not a
  pile of stale bot PRs — that's a dashboard nobody reads)?
- Any dead-code tool configured (knip, ts-prune, vulture, deadcode,
  unimport)?
- Specs with traceable ids, and anything checking them?
- Mutation score tracked anywhere over time?

## Recipes by stack

| Scan | JS/TS | Python | Go | JVM | .NET | Ruby |
|------|-------|--------|----|----- |------|------|
| Dependencies | Dependabot / Renovate (host-level, stack-agnostic) | same | same | same | same | same |
| Dead code | knip, ts-prune | vulture | `deadcode`, `staticcheck -unused` | ArchUnit freeze + IDE inspections in CI | Roslyn analyzers | debride, rubocop unused cops |
| Mutation | Stryker | mutmut / cosmic-ray | go-mutesting | PIT | Stryker.NET | mutant |

Spec-drift scan: portable script (~80 lines) — collect `AC-n` ids from
`specs/**`, collect the ids referenced in test names/tags, diff the two
sets both ways, fail the report on orphans. Run it on the schedule, or
in CI when either side changes.

## How adopt writes it

1. Pick **one** scan the assessment justified (a stale-dependency
   incident → dependencies; a "why is this here" moment → dead code;
   spec/code divergence pain → spec-drift).
2. Wire it as a scheduled CI job in the host's idiom
   (`ci-and-vcs.md`), opening a PR or writing a dated report — never
   auto-merging.
3. Note in `AGENTS.md` where the reports land and who triages them.
4. Smoke-check: trigger the job manually once; confirm it produces the
   PR/report and that a planted violation (an unused export, an orphan
   AC id) is caught.

## Decay notes (for audit)

- Bot PRs piling up unmerged = the scan became wallpaper — fix the
  triage routine or reduce frequency; don't let it train the team to
  ignore green machinery.
- Mutation trend flat-lining downward across audits = the behaviour
  layer is weakening invisibly; schedule survivors-list work
  (`behaviour-test.md`).
