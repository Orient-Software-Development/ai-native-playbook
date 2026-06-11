---
name: harness-assess
description: Read-only assessment of a repository's engineering practices against the harness-engineering model. Detects the stack, the AI coding tool, and the controls already present; scores the five harness layers; reports a ranked gap list. Use when the user asks to "assess our harness", "score our practices", "how harness-ready is this repo", "where are our gaps", or invokes /harness-assess. Writes nothing to the repo unless the user asks to save the report.
---

# Harness Assess

Score how well this repository already practises harness engineering,
and produce a ranked list of the highest-value gaps. **Read-only** — you
inspect and report; you do not modify the repo. Adoption is the separate
[adopt](adopt.md) skill.

> **Mental model.** A harness has five layers (feedforward, maintainability,
> architecture fitness, behaviour, inferential). Each layer needs the loop
> to *close*: a guide shapes the work, a sensor grades it, a gate enforces
> the verdict. Your job is to find, per layer, which of those three are
> present and which are missing — then rank the gaps by value, not by layer
> order. See `../reference/harness-model.md` for the full model.

---

## Step 0 — Detect the context

Run these in parallel and infer the stack, tool, and VCS/CI. Do **not**
ask the user what you can detect yourself.

```
pwd && git rev-parse --show-toplevel 2>/dev/null
git branch --show-current 2>/dev/null && git remote -v 2>/dev/null
ls -la
```

Then probe for the markers below. Each maps to a column in the patterns
library (`../patterns/`).

| Signal | Look for | Tells you |
|--------|----------|-----------|
| **Language / stack** | lockfiles & manifests: `package.json`+`pnpm-lock.yaml`/`package-lock.json`/`yarn.lock`, `pyproject.toml`/`requirements.txt`/`poetry.lock`, `go.mod`/`go.sum`, `pom.xml`/`build.gradle`, `*.csproj`/`*.sln`, `Gemfile`/`Gemfile.lock`, `Cargo.toml` | which stack recipe to use |
| **AI tool / guide file** | `AGENTS.md`, `CLAUDE.md`, `.cursor/rules` or `.cursorrules`, `.github/copilot-instructions.md`, `.codex/` | feedforward layer; which adapter the team uses |
| **Gate** | `.git/hooks/`, `lefthook.yml`, `.husky/`, `.pre-commit-config.yaml`, `Makefile` targets, CI config | is there a blocking gate, and where |
| **Maintainability sensors** | linter/formatter/type-check config; any suppression-budget, todo-budget, or secret-scan script | which sensors exist |
| **Tests / behaviour** | test runner config; presence of e2e (Playwright/Cypress/etc.); test count | the behaviour layer |
| **Architecture rules** | boundary lint rules, dependency-cruiser/import-linter/ArchUnit, module structure | architecture-fitness layer |
| **Inferential review** | any LLM-in-CI workflow (e.g. a Claude/Codex review action) | inferential layer |
| **CI / VCS** | `.github/workflows/`, `.gitlab-ci.yml`, `azure-pipelines.yml`, `Jenkinsfile`; remote host | which CI/VCS adapter |

State back to the user, in two or three lines: the detected stack, the
AI tool (or "none detected"), the VCS/CI host, and whether a harness is
already partly present.

---

## Step 1 — Readiness (harnessability)

Score the repo against `../reference/readiness-rubric.md`
(0–100). This measures whether the repo can *support* a harness at all
(typed language, lockfile, test runner, CI, branch protection). Report
the score and the lowest-scoring items — these are prerequisites that
make later controls no-ops until fixed.

Do not block on a low score. A low score reshapes the gap list (e.g.
"add a lockfile" outranks "add architecture fitness").

---

## Step 2 — Ask only what you cannot detect

Use at most five questions (group them). Skip any you already answered
from detection.

1. **Does your gate block or only warn?** (A check that warns is not a
   gate — this is the single most common false positive in detection.)
2. **Is there a behaviour test** — one that asserts an observable
   end-state of the product's most important path, not just "it ran"?
3. **Where do specs / contracts live**, if anywhere? (folder, ticket
   system, or "in people's heads")
4. **Which AI coding tool(s)** does the team use day to day?
5. **Context:** greenfield or legacy? team size? any recent incident the
   harness should have caught?

---

## Step 3 — Score the five layers

For each layer give a score **0–10** using the bands below, and record
**which of {guide, sensor, gate} is present**. A layer can score high
on guides and zero on enforcement — say so explicitly; that is the most
important finding a harness assessment produces.

### Feedforward (guides)
- 9–10: a concrete, current guide file (`AGENTS.md`/`CLAUDE.md`); every
  reference resolves; conventions are specific enough to act on.
- 5–6: a guide exists but is vague, stale, or aspirational ("be careful
  with X").
- 0–4: no guide file, or it is a stub the agent will mis-infer from.

### Maintainability
- 9–10: format + lint + types run as one `check`; warnings fail;
  net-new suppressions and orphan TODOs are budgeted; no committed secrets.
- 5–6: linter/types exist but warnings are tolerated or not gated.
- 0–4: no formatter/linter/type-check wired, or they only warn.

### Architecture fitness
- 9–10: module boundaries are enforced by a machine (boundary lint,
  dependency graph, fitness assertions).
- 5–6: boundaries are documented but enforced only in review.
- 0–4: no boundaries, or imports go anywhere.

### Behaviour
- 9–10: at least one real behaviour test asserting an observable
  end-state, run by the gate; coverage of critical paths.
- 5–6: unit tests exist but nothing proves the product's main path
  end to end.
- 0–4: no tests, or tests assert only "did not throw".

### Inferential
- 9–10: an LLM review runs on PRs in CI (never on the fast path) and the
  team acts on it.
- 5–6: tried once or runs but is ignored.
- 0–4: none. **(This is fine for most teams — do not over-weight it.)**

Map these to the scorecard at
`../reference/scorecard-template.md` (Overall / 50).

---

## Step 4 — The loop-closure check (the headline)

For each layer, classify the loop:

- **Closed** — guide + sensor + gate all present.
- **Half-loop** — guide with no gate (a *suggestion box*), or sensor
  with no gate (a *dashboard nobody reads*).
- **Open** — nothing.

The most valuable gaps are almost always **half-loops on a layer that
already scored well on guides** — the team has done the thinking and
gets none of the enforcement. Call these out first. See the
[half-loop anti-pattern](../../playbook-md/50-adoption/01-minimum-viable-harness.md#anti-patterns).

---

## Step 5 — Rank the gaps by value

Produce a ranked list. Rank by *value to this repo*, applying these
rules in order:

1. **Readiness blockers first** — a missing lockfile or test runner
   makes downstream sensors meaningless (Step 1).
2. **Close the minimum viable harness before widening it** — if any of
   {guide file, `check` command, blocking gate, one behaviour test} is
   missing, those outrank every advanced control. This is the playbook's
   [MVH](../../playbook-md/50-adoption/01-minimum-viable-harness.md).
3. **Half-loops over open loops** — finishing a loop the team half-built
   is cheaper and higher-trust than starting a new one.
4. **Match severity to history** — if the user named an incident,
   prioritise the layer that would have caught it.
5. **Demote inferential and behaviour-contract packs** to "later" unless
   the domain clearly warrants them (complex multi-command domains with
   shared invariants — see `../patterns/specs-and-decisions.md`).

---

## Step 6 — Report

Present, in the chat:

1. **Context line** — stack, tool, VCS/CI, greenfield/legacy.
2. **Readiness score** (/100) with the lowest items.
3. **Layer scorecard** (/50) with per-layer loop state (closed /
   half / open).
4. **Ranked gap list** — each gap as: *what's missing → which loop it
   closes → which `patterns/` recipe adopt would use → rough effort
   (afternoon / day / week)*.
5. **One-line recommendation** of the next single increment.

Offer to save the report to `docs/audits/harness-YYYY-MM-DD.md` (using
the scorecard template), but **do not write it unless asked**. End by
offering to run [adopt](adopt.md) on the top gap.

Do not propose more than the user can absorb. The goal of assessment is
a *next step*, not a backlog.
