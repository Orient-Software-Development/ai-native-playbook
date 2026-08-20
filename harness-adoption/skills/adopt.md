---
name: harness-adopt
description: Generate and install a fitted harness control into a repository, in the target stack's idiom and for the team's AI tool. Takes the ranked gap list from harness-assess and writes only the controls the user approves, smallest first, then re-scores. Use when the user asks to "adopt the harness", "set up the gate", "add the behaviour test", "close the top gap", or invokes /harness-adopt. Writes files, but only with explicit approval and one increment at a time.
---

# Harness Adopt

Install the next harness control the team needs — **generated for their
stack and tool**, not copied from a fixed template. Work one increment
at a time, smallest first, and re-assess after each. This is the
[steering loop](../../playbook-md/50-adoption/02-growing-the-harness.md)
made executable.

> **Mental model.** You are not dropping a 23-file kit. You are writing
> the *one control* that closes the highest-value loop the assessment
> found, in the idiom of the detected stack, then stopping so the team
> can feel it work before you add the next. A harness the team adopts
> beats a bigger one they route around.

---

## Step 0 — Get or rebuild the gap list

If [assess](assess.md) ran this session, use its ranked gap list. If not,
run assess first (or its detection + scoring steps inline) — never adopt
a control the assessment didn't justify.

Confirm with the user **which single gap** to close now. Default to the
top of the ranked list. If the minimum viable harness is incomplete
(missing guide file, `check` command, blocking gate, or one behaviour
test), insist on those before any advanced control, and say why.

---

## Step 1 — Choose the recipe for the stack

Open the relevant `../patterns/*.md` for the chosen control. Each pattern
file carries **recipes per stack**. Select the recipe matching the
detected stack:

| Control | Pattern file |
|---------|--------------|
| Guide file (`AGENTS.md`) | `../patterns/guide-file.md` |
| `check` command + blocking gate | `../patterns/check-and-gate.md` |
| Format / lint / types / suppression budget / todo budget / secret scan | `../patterns/maintainability-sensors.md` |
| One behaviour test | `../patterns/behaviour-test.md` |
| Architecture fitness | `../patterns/architecture-fitness.md` |
| Inferential (LLM) review in CI | `../patterns/inferential-review.md` |
| Specs / ADRs / behaviour-contract packs | `../patterns/specs-and-decisions.md` |
| CI workflow / VCS wiring | `../patterns/ci-and-vcs.md` |
| Scheduled drift & health scans (deps, dead code, spec-drift, mutation trend) | `../patterns/drift-and-health.md` |

If the stack isn't covered explicitly, use the **generic recipe** in the
pattern (the control's intent + blocking behaviour) and implement it with
the stack's native tooling. Each pattern describes the control's algorithm
in prose — port that logic into the repo's own scripting idiom rather than
importing anything from this starter.

---

## Step 2 — Propose before writing

State, in the chat, before touching the repo:

- **What** you will create or modify (exact paths).
- **How it enforces** — at which stage it blocks, and what makes it
  fail (a warn-only control is not a gate; say explicitly that it blocks).
- **What it depends on** (a tool to install, a CI secret, a lockfile).
- **What it does NOT cover** — so the team knows the loop's edge.

Get a yes. For anything outward-facing or hard to reverse (CI changes,
branch protection, installing a hook globally), confirm specifically.

---

## Step 3 — Write the control

Apply the recipe. Hold to these rules:

1. **Guide file is `AGENTS.md` by default** — the cross-tool standard
   read by Copilot, Cursor, Codex, and most local agents. If the team
   uses Claude, also write a one-line `CLAUDE.md` that points at
   `AGENTS.md` (don't duplicate content). See `../patterns/guide-file.md`.
2. **One `check` entry point.** Format + lint + types behind a single
   command the gate runs, the guide names, and a human runs by hand.
   Never make anyone remember three commands.
3. **The gate blocks.** Wire it at the earliest stage you can *enforce*
   (git hook / pre-commit framework / CI). Warn-only is not a gate.
4. **Keep quality left.** Fast cheap checks early (pre-commit/pre-push);
   expensive or inferential checks in CI. Never put an LLM gate on the
   fast path. See `../reference/harness-model.md` §4.
5. **Sensored or aspirational, pick one.** Every rule you add to the
   guide file gets a sensor, or is explicitly marked soft. Don't write
   rules you can't enforce — they decay and erode trust in the whole guide.
   For an invariant that must hold every time, prefer a **hook** (a script
   the agent tool runs on the event, hard-blocking the action) over another
   line of prose — instructions are advisory; hooks are deterministic.
   See `../patterns/guide-file.md`.
6. **No dangling references.** Every path the guide mentions must exist.
7. **Match the surrounding code.** Use the repo's existing config style,
   naming, and tool versions. Don't introduce a second formatter.

---

## Step 4 — Smoke-check

Prove the control works before declaring it done:

- Run the new `check` (or sensor) on the current tree — it should pass or
  report real findings, not error on a missing dependency.
- Make a trivial **violating** change in a scratch spot and confirm the
  gate **fails** on it, then revert. A gate you didn't watch say "no" is
  a gate you can't trust.
- For a behaviour test: confirm it asserts an observable end-state and
  goes red when the behaviour is broken — not green-on-everything.

Report what you ran and the result honestly. If a smoke-check fails,
surface it; do not paper over it.

---

## Step 5 — Wire it into the loop

- Add the control to the `check` command (if it belongs on the fast path)
  or the CI workflow (if it's expensive/inferential).
- If it enforces a rule, add that rule to `AGENTS.md` with a pointer to
  the sensor, so feedforward and feedback name the same thing.
- Update any spec/decision record if this was a structural choice
  (`../patterns/specs-and-decisions.md`).

---

## Step 6 — Stop, then re-assess

Do **not** chain into the next control automatically. Commit only on
explicit instruction (propose one logical unit). Then:

1. Re-run the relevant part of [assess](assess.md) — the layer you
   touched should move from half-loop/open to closed.
2. Show the updated layer score.
3. Offer the **next single gap** from the ranked list — and remind the
   team that the right time to add it is *after they've watched this one
   catch something*, per
   [growing the harness](../../playbook-md/50-adoption/02-growing-the-harness.md).

Adopting all gaps at once is the
[big-bang anti-pattern](../../playbook-md/50-adoption/01-minimum-viable-harness.md#anti-patterns).
Resist it even when the team asks — explain that incremental adoption is
what survives contact with a real deadline.

---

## Greenfield vs. legacy

- **Greenfield:** the cheapest moment to bank harnessability. Write the
  guide file, `check`, gate, and one behaviour test *before* the first
  feature. Choose a typed language and clear boundaries now.
- **Legacy:** never retrofit a control across the whole codebase at once.
  **Seed budgets at the current count** (suppression budget, todo budget,
  coverage) so existing debt is grandfathered and only *net-new* debt is
  gated. Apply new behaviour-contract patterns to the next domain in
  active development, not retroactively. See
  `../../playbook-md/50-adoption/04-legacy-and-brownfield.md`.
