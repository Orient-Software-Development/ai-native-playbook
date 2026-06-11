# Keep Quality Left

> Put each check at the *earliest* stage whose speed it can afford — fast cheap checks on every commit, slow expensive ones in CI. A hook fast enough that nobody skips it, plus a CI gate strict enough that nobody can dodge it, is the whole pattern.

Breadcrumb: [Playbook](../README.md) › Harness

## Story so far

The last page built the pyramid of sensors — cheap fast checks at the broad base, slow expensive ones at the narrow tip. Which raises the obvious follow-up: where in your workflow does each one actually fire? The instinct here — "run everything as early as possible!" — sounds like diligence and is actually a trap, one with a name, and this whole page exists to keep you out of it. The rule is gentler than the instinct: push each check as far left as its speed allows, and *not one stage further*.

## The principle

The [last page](03-sensors-feedback.md) built the layered gate — the pyramid of sensors, cheap
deterministic checks at the broad base, slow expensive ones at the narrow tip. This page answers the
question that pyramid raises: *where in the workflow does each tier actually run?*

The development lifecycle is a pipeline of stages, each with its own cadence — how often it runs and how
long you'll tolerate it taking:

```
pre-commit  →  pre-push  →  CI  →  staging  →  production
fast / cheap                                   slow / expensive
runs constantly                                runs rarely
```

A **git hook** is a check the version-control tool runs automatically at a moment in your local
workflow — `pre-commit` fires every time you commit, `pre-push` every time you push. They're the
left-most, highest-frequency stages. **CI** (continuous integration) is the check that runs on the shared
server when you open or update a pull request — lower frequency, and the gate that actually decides whether
a change may merge.

**Keep quality left** is the rule for placing checks along that line: push each check as *far left as its
speed allows*, and no further. A formatter that runs in milliseconds belongs on every commit. A full
end-to-end suite that takes minutes belongs in CI, where it runs once per PR instead of once per keystroke.
Match the check's cost to a stage whose cadence can absorb it.

Note the emphasis: as far left as it *can afford*, not all the way left. "Keep quality left" is often
misread as "run everything as early as possible." Do that and you get the failure this page exists to
prevent.

## Why it works

A check has one job: surface a failure as early and as cheaply as possible. "Early" is the whole value —
a type error caught the instant you save costs a glance; the same error caught only in CI costs a commit, a
push, a CI run, and a context-switch back. So your instinct is right that cheap checks want to be left.

But the pipeline has a second force pulling the other way: **a stage that runs constantly can only afford
checks that are fast.** Pre-commit fires on every single commit. Load it with a five-minute build and every
commit now costs five minutes — and here's what actually happens next, every time: the developer (and the
agent) starts reaching for the escape hatch. Git's `--no-verify` flag skips the hook entirely, and once
skipping becomes the only way to work at a reasonable pace, *the gate is bypassed by default.* A hook
everyone skips protects nothing — it's worse than no hook, because the team believes they're covered.

That's why the expensive checks go right. In CI they run once per pull request, on a server, where a few
minutes is fine because you're not blocked waiting on them — you've moved on to the next thing. And CI has a
property the hook can't: **it can't be bypassed.** `--no-verify` is a local choice; a required CI check is a
wall. So the authoritative "this must hold before merge" checks belong where they can't be skipped, and the
hooks stay a fast local courtesy that catches the cheap mistakes before they ever reach the PR.

Put the two forces together and the placement falls out: each check goes to the *earliest* stage whose
cadence its cost doesn't break.

## How to apply it

- **Map each check to the earliest stage it can afford.** Formatters, linters, type checks, and the
  fastest unit tests → pre-commit. Slightly heavier but still quick checks (schema/migration validation,
  the directly affected tests) → pre-push. Full build, integration against real services, the whole e2e
  suite, performance and load tests, inferential review → CI. The rule of thumb: if it takes longer than a
  breath, it doesn't belong on every commit.
- **Keep hooks fast enough that skipping is never worth it.** The moment a hook is slow enough that people
  reach for `--no-verify`, you've lost it. Treat hook speed as a feature: a sub-second pre-commit is one
  nobody has a reason to bypass.
- **Scope the check to the change, not the whole repo.** A pre-commit check should run on the *staged
  files*, not re-lint and re-test the entire codebase every time. And a change that touches only docs or
  comments should skip the executable gates entirely — there's nothing for them to catch.
- **Make CI the authoritative gate.** The hook is a courtesy that helps you catch mistakes early; the
  required CI check is the wall that decides what merges. Put every must-not-merge check where it can
  [fail the build](03-sensors-feedback.md) and can't be bypassed — not only in a hook a developer can skip.
  (How that merge gate is structured is its own topic: continuous integration and delivery.)
- **Thin the heavy stages deliberately.** When you find yourself adding the full suite to pre-push "to be
  safe," stop — that safety already lives in CI. Decide, on purpose, what each local stage runs and keep the
  slow work off it.
- **Don't:** pile the full build or e2e onto a git hook; run the same slow check at every stage "to be
  safe" (it taxes every iteration and buys no extra safety, since CI already has it); leave a cheap
  deterministic check — lint, types — to fire *only* in CI, making a trivial error cost a full round-trip
  instead of failing on save.

## In practice

A team sets up its harness with the best of intentions: to protect the trunk, it puts the *entire* check
suite — full build, every unit test, the end-to-end run — on the pre-commit hook. Nothing broken can be
committed, the thinking goes.

**What actually happens.** Every commit now takes minutes. Within days, the friction is unbearable:
committing a one-line fix means waiting through the whole e2e suite. So the developers — and the agent
working alongside them — start committing with `--no-verify` to get anything done. It's not laziness; it's
the only way to keep moving. But now the gate fires on *no* commits. The first time someone bypasses a hook
that *would* have caught a real break, a broken change lands on the branch behind a green-looking history,
and the team is slower to notice precisely because they trusted the hook they'd all quietly stopped running.

**The fix.** Distribute the checks by cost. The pre-commit hook is thinned to the fast, scoped checks —
format the staged files, lint and type-check them, run the handful of tests directly touched — so a commit
is back to near-instant and nobody has a reason to skip it. A docs-only change skips even those. The heavy
work — full build, integration against a real database, the full e2e suite — moves to CI, where it runs once
per pull request and *can't* be bypassed with a local flag. Now the cheap mistakes fail on the left in
seconds, the expensive checks catch the rest on the right where their cost is amortised over a whole PR
instead of a single keystroke, and `--no-verify` goes back to being the rare emergency tool it should be.

The lesson the example carries: "keep quality left" is a placement rule, not a "more is earlier" rule. A
check belongs at the earliest stage whose speed it can afford — cheap checks early so failures surface in
seconds, expensive checks in CI so they don't tax iteration and so they sit where they can't be dodged.
Overload the left and you don't get more safety; you get a hook the whole team learns to skip, which is no
safety at all.

## Anti-patterns

- **The heavyweight hook.** The full build or e2e suite on pre-commit/pre-push, so iteration crawls and the
  team routinely reaches for `--no-verify`. A gate everyone bypasses is no gate — and worse, it's trusted.
- **The everything-in-CI-only gate.** Even lint and type checks run *only* on the server, so a trivial
  error that should fail on save instead costs a commit, a push, and a full CI round-trip to discover.
- **The unscoped hook.** Re-checking the entire repository on every commit instead of just the staged
  files, and running the executable gates on docs-only changes that can't fail them.
- **The redundant heavy check.** Running the same slow suite at every stage "to be safe," taxing every
  iteration for safety CI already provides.
- **The skippable wall.** Treating a bypassable hook as the authoritative gate. The must-not-merge checks
  belong in CI where they [can actually fail the build](03-sensors-feedback.md), not only in a hook a flag
  can skip.

> **Next up — [Behaviour Harness](05-behaviour-harness.md):** the structural checks are placed and humming. But the hardest sensor of all is still ahead — the one that proves the code does what the *business* wanted, not merely that it's tidy and well-shaped. Next we tackle behaviour, and the agent's single favourite way to cheat at it: writing a test that just agrees with the code it wrote.

---
[← Previous: Sensors — Feedback](03-sensors-feedback.md) · [Contents](../README.md) · [Next → Behaviour Harness](05-behaviour-harness.md)

Related: [Sensors — Feedback](03-sensors-feedback.md) · [Behaviour Harness](05-behaviour-harness.md) · [Harness Engineering](../00-foundations/03-harness-engineering.md) · [CI and CD](../30-delivery/02-ci-and-cd.md) · [Trunk-Based Development](../30-delivery/01-trunk-based-development.md)
