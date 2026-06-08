# Harness Scorecard — `<repo name>`

> **Date:** `<YYYY-MM-DD>`
> **Repo:** `<org/repo @ commit-sha or branch>`
> **Scored by:** `<Claude / Codex / model id>`
> **Method:** `<static tree scan (grep/find/wc) + GitHub-API pass (gh) — name what was run>`
> **Previous scorecard:** `<link, or "none — this is the baseline">`
> **Reference:** [Scoring an Existing Harness](../50-adoption/05-scoring-an-existing-harness.md) · [Harness Engineering](../00-foundations/03-harness-engineering.md)

A point-in-time assessment of this repository's harness posture, scored against the five-layer rubric.
Every score cites the command, file, config line, or GitHub setting behind it. Figures are measured over
**tracked source only** unless explicitly noted. Where a previous scorecard exists, this is a **delta**
against it.

---

## 0. Harnessability (sets the ceiling — score the layers against this, don't average it in)

> *Why first: an untyped, boundary-less, framework-less repo cannot reach the top band on Maintainability or
> Architecture no matter how disciplined the team — the cheap sensors that earn those points aren't available
> to it. Record the ground before grading what's built on it.*

| Property | State (with evidence) | Raises ceiling? |
|---|---|---|
| **Static types** | `<e.g. TypeScript strict / untyped JS / Python + mypy / none>` | `<yes/partial/no>` |
| **Module boundaries** | `<e.g. enforced workspace packages / implicit / monolith>` | `<yes/partial/no>` |
| **Established framework** | `<e.g. Next.js / Rails / bespoke>` | `<yes/partial/no>` |
| **Test coverage present** | `<e.g. suite exists + runs in CI / sparse / none>` | `<yes/partial/no>` |

**Stack:** `<language(s), framework, build system, CI provider>`
**Ceiling read:** `<one line — e.g. "high harnessability; top band is reachable" or "untyped legacy; Maintainability capped ~5">`

---

## Summary scorecard

> *Band anchors are in §Appendix. Trend column only applies when a previous scorecard exists.*

| Layer | Score | (Prev) | Trend | Headline (one line, evidence-backed) |
|---|---|---|---|---|
| **Feedforward (Guides)** | `<n>` / 10 | `<n>` | `<▲▬▼>` | `<…>` |
| **Maintainability harness** | `<n>` / 10 | `<n>` | `<▲▬▼>` | `<…>` |
| **Architecture fitness** | `<n>` / 10 | `<n>` | `<▲▬▼>` | `<…>` |
| **Behaviour harness** | `<n>` / 10 | `<n>` | `<▲▬▼>` | `<…>` |
| **Inferential sensors** | `<n>` / 10 | `<n>` | `<▲▬▼>` | `<…>` |

**Headline:** `<2–4 sentences: the single most consequential finding, the biggest blind spot, and any correction to a prior figure.>`

---

## 1. Feedforward layer (Guides) — `<n>` / 10

*What steers the agent before it acts: instruction files, conventions, blueprints, repo legibility.*

- **Present / strong:** `<finding — cite file:line or path>`
- **Missing / weak:** `<finding — e.g. dangling reference: CLAUDE.md cites X; `ls X` → missing>`
- **Evidence:** `<exact commands and their output: spec count, ADR count, instruction-file presence, dead links>`

## 2. Maintainability harness — `<n>` / 10

*Computational sensors over internal code quality: lint, format, types, dead code, complexity.*

- **Present / strong:** `<finding — cite config line, e.g. eslint.config.mjs, pre-commit hook>`
- **Missing / weak:** `<finding — e.g. no coverage threshold; vitest.config.ts has no floor>`
- **Evidence:** `<exact commands — git grep '@ts-ignore' (TRACKED SOURCE), gate/hook config, CI job lines>`
- **⚠ Integrity check:** `<confirm any count was measured over tracked source, not node_modules / vendored / generated>`

## 3. Architecture fitness harness — `<n>` / 10

*Automated assertions about the system's shape: boundary rules, dependency direction, fitness functions, budgets.*

- **Present / strong:** `<finding — cite the rule and where it gates, e.g. CI workflow line>`
- **Missing / weak:** `<finding — e.g. no graph-level boundary enforcement beyond lint import rules>`
- **Evidence:** `<exact commands — boundary-rule config, fitness-function scripts, what runs hard vs advisory>`

## 4. Behaviour harness — `<n>` / 10

*Does the code do what the business wanted: spec-as-guide + tests that prove it. The hardest layer.*

- **Present / strong:** `<finding — test count, scenario traceability, E2E suite, what's required in CI>`
- **Missing / weak:** `<finding — e.g. green PRs ship no rendered-UI evidence; no mutation testing>`
- **Evidence:** `<exact commands — test file count, scenario-id occurrences (use the repo's real id scheme), screenshot/trace config>`

## 5. Inferential sensors — `<n>` / 10

*Semantic checks computation can't reach: AI reviewers, spec-to-code validators.*

- **Committed (in-repo, gating, spec-aware):** `<finding — or "none">`
- **Lived (org-side, advisory):** `<finding — gh pass over recent PRs: which AI reviewer ran, on how many of last N PRs>`
- **Fragility flag:** `<any control that lives only in org settings with no in-repo trace>`
- **Evidence:** `<exact commands — gh pr list --json reviews, presence of copilot-instructions.md / review config>`

---

## 6. Status of prior recommendations (delta runs only)

| Priority | Action (from `<prev date>`) | Status now |
|---|---|---|
| `<P1>` | `<…>` | `<✅ done / ◑ partial / ❌ not done — cite evidence>` |

---

## 7. Recommended actions, by impact-to-effort

| Priority | Action | Effort | Why now |
|---|---|---|---|
| **P1** | `<the highest-leverage gap — name the exact change>` | `<e.g. 1h>` | `<…>` |
| **P2** | `<…>` | `<…>` | `<…>` |
| **P3** | `<…>` | `<…>` | `<…>` |

---

## 8. Open questions

1. `<ambiguity the scan surfaced but couldn't resolve — e.g. "is the unprotected integration branch intentional?">`

---

## Appendix — band anchors

Score each layer against these. When a repo sits between two bands, the *evidence* decides — cite it.

| Band | Feedforward | Maintainability | Architecture fitness | Behaviour | Inferential |
|---|---|---|---|---|---|
| **1–2 · absent** | no instruction file; conventions live only in people's heads | no linter/formatter/type check, or none enforced | no boundary or fitness rule of any kind | no tests, or tests that don't run | no AI sensor, committed or lived |
| **3–4 · nascent** | a thin README; scattered, partly-stale notes | tools exist but run only locally / advisory | a lint import rule or two; nothing at the schema/graph level | a sparse suite, no CI gate, no traceability | a generic org-side bot with zero repo context |
| **5–6 · partial** | concrete instruction file; some conventions documented, some gaps/dead links | lint+format+types enforced at one stage (pre-commit *or* CI) | one real fitness function gating; lint boundaries | tests run in CI, weak coverage, thin traceability | org-side AI reviewer running, advisory, no in-repo config |
| **7–8 · solid** | instruction file + spec/ADR corpus + legible layout; minor gaps | full computational sensors gating in CI; pre-commit→CI split | boundary + dependency-direction enforced hard in CI | broad suite + scenario traceability + E2E required; one blind spot (e.g. no visual evidence) | AI reviewer with repo context (instruction file committed), advisory |
| **9–10 · exemplary** | corpus + maintained map/blueprints + no dead guides; pruned, not just grown | above + coverage thresholds + gating security/dead-code sensors | above + graph-level boundary enforcement + budgets (bundle/perf) | above + mutation testing + visual/behavioural evidence on green | in-repo, gating, **spec-aware** AI sensor that reads specs and judges drift |

**Scoring discipline (non-negotiable):**
- Every score cites a command, file:line, or GitHub setting. No unsourced numbers.
- Every count is measured over **tracked source** (`git grep`, not store-walking `grep -r`). State the command.
- Discover the repo's *own* idioms — don't grep for another repo's conventions.
- An advisory check (`continue-on-error`, un-required, unprotected branch) is **not** a gate. Score it as a report.
- Read every score against the harnessability ceiling recorded in §0.

> **Method note (`<date>`):** `<exactly what was run — tree commands, gh queries — and any prior figure corrected after re-measurement. State that no source outside this document was modified.>`
