# Pattern: inferential (LLM) review

## Intent

An LLM reviews the diff for **semantic** problems that computational
sensors can't see — does the change match the spec's intent, are there
subtle logic errors, does it violate a convention that's hard to lint.
Reserve inference for semantic problems; never use it where a
deterministic check would do. See
`../../playbook-md/00-foundations/03-harness-engineering.md`.

This is a **later** control. Most teams should close the other four
layers first. Add it when computational sensors are solid and the
remaining escapes are semantic.

## What good looks like

- Runs on PRs in CI, posting findings as review comments.
- **Never on the fast path.** An LLM gate on pre-commit/pre-push destroys
  the local feedback loop and costs more than it's worth.
- Tuned to be advisory or to gate only on high-confidence categories —
  a noisy LLM reviewer gets ignored like any noisy sensor.
- The team actually acts on it (assessment/audit checks this).

## Blocking behaviour

Default **advisory** (comments, doesn't fail the merge) until the team
trusts it. Graduate specific high-precision checks to blocking once
proven. Keep it in CI only.

## Assessment signal

- Any LLM-review workflow in CI? Is it acted on or ignored?
- Is an API key already in CI secrets (lowers the cost of adopting)?

## Recipes by tool

The reviewer is **tool-agnostic** — pick whatever the team already pays
for; the pattern is the same:

| Tool | How |
|------|-----|
| **Claude** | A CI step that pipes the PR diff to the Anthropic API with a review rubric (Claude Code ships a GitHub Action for PR review; or call the API directly from a workflow step). |
| **Copilot** | GitHub Copilot code review on PRs (native, if the org has it). |
| **Codex / OpenAI** | A CI step calling the OpenAI API with the same rubric. |
| **Cursor / local model** | A CI step (or self-hosted runner) invoking the local model with the rubric. |

The valuable, portable part is the **review rubric** — the specific
things to check (spec alignment, error handling, tenant scoping, etc.).
Keep the rubric in the repo so it's reviewed like code; swap the model
behind it freely.

## Calibrating the reviewer

A separated evaluator is necessary, not automatically good. Untuned,
it fails in two documented ways: it finds a real issue and talks itself
into approving anyway, and it tests the happy path instead of the edges.
Separation removed the author's bias; calibration is what adds the
skepticism:

- **Anchor verdicts with worked examples.** Put a few graded cases in
  the rubric — including the reasoning that led to each verdict, not
  just the score. Anchors reduce drift run to run.
- **Tune in a loop.** Read the reviewer's output and find where its
  judgment diverged from yours — issues it excused, passes you'd have
  failed. Update the rubric; run again. Expect several rounds.
- **Stress-test the wording.** Criteria steer beyond their literal
  intent — watch what a phrase *rewards*, not just what it says.
- **Don't chase every finding.** A reviewer prompted to find gaps will
  report some even when the work is sound. Findings are input to human
  judgment, not a to-do list — acting on all of them over-engineers.

## How adopt writes it

1. Confirm the four computational layers are solid first — if they're
   not, redirect to those; inference is not a substitute for a type check.
2. Write the review rubric (repo-specific checks) into the workflow or a
   committed prompt file.
3. Add a CI-only job that runs on PR, using the team's chosen model and
   the credentials already in CI secrets. Default to advisory.
4. Remind the user to add the API key to CI secrets if absent.
5. No smoke-check of "blocking" needed while advisory; once graduated to
   blocking on a category, smoke-check that category fails as expected.

## Decay notes (for audit)

- Is it acted on, or has it become wallpaper? An ignored inferential
  sensor should be tuned or switched off — cost without signal.
- **The lenient skeptic** — a reviewer trusted without its verdicts ever
  being checked against a human's. Until someone has compared a few of
  its calls to their own, the team has separation, not skepticism.
  Sample its verdicts each audit.
