# Harness Scoring Prompt

> Hand this to a coding agent (Claude or Codex) with read access to a repository. It produces a filled-in
> [harness scorecard](harness-scorecard.md). It is model-agnostic by design — run it with either model, or
> both and merge the evidence. Copy everything in the block below as the agent's instruction.

---

You are auditing the **harness** of the repository you have access to — everything that steers and grades a
coding agent *except* the model itself. Produce a completed scorecard using the structure in
`harness-scorecard.md` (same directory). Score five layers 1–10. **This is an assessment only: do not modify
any source file outside the scorecard you are writing.**

Read [Harness Engineering](../00-foundations/03-harness-engineering.md) and
[Scoring an Existing Harness](../50-adoption/05-scoring-an-existing-harness.md) first if available — they
define the five layers and the scoring discipline.

## Phase 1 — Discover (do this before scoring anything)

Establish what *this* repo is, and how it realises each harness capability. **Do not assume another repo's
idioms.** Find:

- **Stack & harnessability:** language(s), static-typing state, framework, build system, CI provider, test
  runner. Are there clear module boundaries? Does a test suite exist and run? Record this in §0 — it sets the
  ceiling.
- **The repo's own conventions:** locate the instruction file(s) (`CLAUDE.md`, `AGENTS.md`, `.cursorrules`,
  `copilot-instructions.md`, …), the spec/ADR/docs corpus if any, the lint/format/type config, the CI
  workflows, the test layout, and any branch-protection / rulesets via the GitHub API if you have `gh`.
- **How each capability is realised here.** The rubric grades *capabilities*, not tools. Before scoring, map
  each to this repo's actual mechanism — e.g. "boundary enforcement" might be an ESLint `no-restricted-imports`
  rule, a Postgres FK audit, a Rails engine, or nothing. Find the mechanism (or confirm its absence).

## Phase 2 — Measure (evidence first, score second)

For each of the five layers — Feedforward, Maintainability, Architecture fitness, Behaviour, Inferential —
gather **reproducible evidence**, then assign a score against the band anchors in the scorecard appendix.

**Measurement integrity (non-negotiable):**

1. **Tracked source only.** Every count (suppressions, lines, test files, routes, dependency CVEs) must be
   measured over version-controlled source. Use `git grep` / `git ls-files`, **never** a store-walking
   `grep -r` that can follow symlinks into `node_modules`, `vendor`, `.venv`, build output, or generated
   files. A suppression count that includes dependencies can read 1,700 when the truth is 0. **Write the exact
   command into the scorecard** so the next run reproduces it.
2. **No unsourced scores.** Every score cites a command, a `file:line`, or a GitHub setting. If you can't cite
   it, you can't score it — investigate further or mark it unknown.
3. **Advisory ≠ gate.** A check that is `continue-on-error`, not in required status checks, or on an
   unprotected branch is a *report*, not a gate. Verify what actually *blocks a merge* (check branch
   protection AND rulesets — a `branches/{b}/protection` 404 does not mean unprotected; query rulesets too).
   Score enforcement by what genuinely fails the build.
4. **Committed vs lived.** For the inferential layer especially, distinguish controls the tree enforces from
   controls merely *running* in org settings. A `gh pr list --json reviews` pass over the last ~20–30 PRs
   shows which AI reviewer is actually active and on what fraction of PRs. Flag any control that lives only in
   org settings with no in-repo trace as **fragile** — it can vanish silently.
5. **Discover the real id schemes.** If you count scenario/test traceability, find the repo's actual id
   pattern (it may be `SCN-<AREA>-<n>`, a ticket ref, a `describe()` tag) — a naïve regex can under- or
   over-count by orders of magnitude. State the pattern you used.

## Phase 3 — Score and report

Fill in the scorecard:

- **§0 Harnessability** with the ceiling read.
- **Summary scorecard** — five scores. If a previous scorecard exists, fill the (Prev) and Trend columns and
  add a §6 status-of-prior-recommendations table; otherwise mark this the **baseline** and omit trends.
- **§1–§5** — for each layer: what's present/strong, what's missing/weak, and the exact evidence (commands +
  output). Run the Maintainability integrity check explicitly.
- **§7 Recommended actions** — ranked by impact-to-effort, each naming the *exact* change (the file to create,
  the one-line config flip, the check to ratchet from advisory to gating), an effort estimate, and why now.
- **§8 Open questions** — ambiguities the scan surfaced but couldn't resolve (e.g. "is this branch
  intentionally unprotected?").
- **Method note** — exactly what you ran, any prior figure you corrected after re-measurement, and the
  statement that no source outside the scorecard was modified.

## Calibration reminders

- Read every score against the §0 ceiling — an untyped legacy repo *cannot* top-band Maintainability, and a
  low score there is a structural ceiling, not negligence.
- A layer scoring 1 means **absent**. Say so plainly; an absent layer should be a deliberate choice, not an
  accidental gap.
- Prefer the blunt, reproducible fact over the sophisticated guess. The zeros and ones the scan makes
  undeniable are where most of the value is.
- If you are unsure between two bands, cite the evidence and pick the lower — then let the recommendation say
  what would earn the higher one.

Output the completed scorecard as a single Markdown document following `harness-scorecard.md`. Nothing else.
