# Playbook Plan — Reference Gap Analysis

> **Purpose:** Cross-check of the three mirrored reference articles in
> [`../references/`](../references/) against [`PLAN.md`](./PLAN.md). Lists topics the
> references raise that the plan currently **misses or under-weights**, ranked by
> load-bearing value, each with a suggested home in the page map. This is a working
> note for the next planning step — not part of the published playbook.
>
> **Sources checked:**
> - Böckeler — *Harness engineering for coding agent users*
> - Anthropic — *Harness design for long-running application development*
> - OpenAI — *Harness engineering: leveraging Codex in an agent-first world*
>
> **Drafted:** 2026-06-06

---

## Already well-covered (for reference)

The plan places these reference topics cleanly, so they are **not** gaps:
context rot, computational vs inferential, the steering loop, the three regulation
categories, generator/evaluator separation, planner/generator/evaluator architecture,
context resets vs compaction, context anxiety, the self-evaluation failure, making
subjective quality gradable, repo-as-system-of-record, progressive disclosure /
`AGENTS.md`-as-table-of-contents, application legibility (in part), throughput-changes-
merge-philosophy, entropy/garbage-collection, increasing levels of autonomy, enforcing
invariants-not-implementations, keep-quality-left, harnessability.

---

## Strong gaps — genuinely absent from the plan

### G1. Harness templates — reusable per-topology harness bundles
- **Source:** Böckeler §"Harness templates".
- **The idea:** mature orgs already codify service topologies (CRUD service on JVM,
  event processor in Go, data dashboard in Node) as service templates; these can evolve
  into *harness templates* — a packaged bundle of guides + sensors that leashes an agent
  to a topology's structure, conventions, and stack. Consequence: **teams may start
  choosing tech stacks based on which harnesses already exist for them.** Böckeler also
  names the downside — template drift / versioning, worse for non-deterministic guides
  and sensors.
- **Why it matters:** a reuse + scaling lever the plan never touches; also leadership-
  relevant (standardise once, leash many services).
- **Suggested home:** new page or subsection in `50-adoption/` (alongside
  `02-growing-the-harness`), with a cross-link from `00-foundations/03`.

### G2. Evaluating the harness itself — "harness coverage"
- **Source:** Böckeler open questions.
- **The idea:** "If sensors never fire, is that a sign of high quality or inadequate
  detection? We need a way to evaluate harness coverage and quality, similar to what code
  coverage and mutation testing do for tests."
- **Gap:** the plan applies mutation testing to *code/tests* (`20-harness/05`) but never
  turns the lens on the **harness as the measured artifact** — how do you know your
  harness is any good?
- **Suggested home:** name it as an open frontier in `00-foundations/03` and/or
  `50-adoption/02`; ties to PLAN §6 (open questions).

### G3. Pruning the harness as models improve (it is not monotonic)
- **Source:** Anthropic §"Iterating on the harness", §"Removing the sprint construct".
- **The idea:** every harness component encodes an assumption about what the model *can't*
  do; those assumptions go stale as models improve. Remove one component at a time and
  measure what was load-bearing; re-examine the whole harness when a new model lands.
  (Anthropic dropped context-resets and the sprint construct after Opus 4.6.)
- **Gap:** `50-adoption/02-growing-the-harness` is **additive only** ("observe failure →
  add a guide or sensor"). The subtractive direction is missing.
- **Suggested home:** add a "prune the harness / stress-test your assumptions" subsection
  to `50-adoption/02`; reference the steering loop in `00-foundations/03`.

---

## Moderate gaps — touched elsewhere, but no clear home

### G4. Environment design — a runnable, isolated app the agent can drive to verify
- **Source:** OpenAI §"Increasing application legibility" (bootable per git worktree,
  Chrome DevTools wired into the agent, ephemeral per-worktree observability stack);
  Anthropic (Playwright-driven evaluator clicking through the live app).
- **The idea:** OpenAI's literal headline is "designing **environments**, feedback loops,
  and control systems." The agent reproduces bugs, validates fixes, and records
  before/after evidence by *driving the running app* in an isolated instance.
- **Gap:** the plan has `10-lifecycle/05-verify` (evidence artifacts) and
  `30-delivery/03-observability`, but no page on **provisioning the runnable, isolated
  environment that makes agent self-verification possible** — the enabler under "proof
  not vibes."
- **Suggested home:** expand `10-lifecycle/05`, or add a dedicated harness page
  (`20-harness/` — "the runnable environment as a sensor substrate").

### G5. A sensor's failure output is itself feedforward
- **Source:** OpenAI §"Enforcing architecture and taste" — "Because the lints are custom,
  we write the error messages to inject remediation instructions into agent context."
- **The idea:** a good sensor doesn't just fail, it *teaches the fix* — closing the
  feedback→feedforward loop in one artifact.
- **Gap:** the plan separates feedforward (`20-harness/01`) and feedback (`/03`) cleanly
  but never notes the crossover.
- **Suggested home:** one paragraph in `20-harness/03-sensors-feedback`.

### G6. Sensors on the guides themselves — doc-gardening / anti-rot
- **Source:** OpenAI §"We made repository knowledge the system of record" — linters/CI
  validate the knowledge base is fresh, cross-linked, owned; a recurring **doc-gardening
  agent** opens fix-up PRs for stale docs ("it rots instantly… an attractive nuisance").
- **Gap:** the plan's garbage-collection treatment (`30-delivery/01`, `50-adoption/03`)
  is about *code* drift; **guides/specs rotting** is a distinct failure mode.
- **Suggested home:** a line in `40-anti-patterns/01` (stale-guide failure mode) and
  `20-harness/02-repo-structure` (enforce knowledge-base freshness mechanically).

---

## Minor gaps — nuances worth a sentence

### G7. Technology selection for agent-legibility
- **Source:** OpenAI §"Agent legibility is the goal" — "boring" tech is easier for agents
  to model (composability, API stability, training-set presence); sometimes cheaper to
  **reimplement a subset than depend on opaque upstream behavior**.
- **Suggested home:** decision rule in `20-harness/02` / harnessability; currently only
  implied.

### G8. Sprint contracts — builder/QA negotiate "done" before code
- **Source:** Anthropic §"The architecture" — generator and evaluator agree testable
  acceptance criteria *before* any code is written.
- **Suggested home:** enrich `10-lifecycle/02-plan-before-code` or `/05-verify`.

### G9. When is an inferential evaluator worth its cost?
- **Source:** Anthropic §"Removing the sprint construct" — "the evaluator is worth the
  cost when the task sits beyond what the model does reliably solo."
- **Suggested home:** the LLM-as-judge question (PLAN §6 Q7) and the cost framing in
  `50-adoption/04-leadership-scorecard`.

### G10. Builder-harness vs user-harness bounded context
- **Source:** Böckeler Figure 1 — what's baked into the agent vs what you build around it.
- **Suggested home:** a small framing aid in `00-foundations/03` so readers know what is
  actually theirs to control.

---

## Recommended next step

Act on **G1–G3** first — each names a frontier the plan's current additive / single-
project framing does not reach (reusable harnesses, measuring the harness, shrinking the
harness). **G4–G6** are loop-closing refinements with obvious homes. **G7–G10** are
one-liners to fold in during drafting.

Concrete edits to PLAN.md:
1. Add a `50-adoption/` page (or subsection) for **harness templates** (G1).
2. Add a **"prune the harness"** subsection to `50-adoption/02` (G3).
3. Add **harness-coverage** as an open frontier in `00-foundations/03` and PLAN §6 (G2).
4. Fold G4–G10 into the named pages above as the chapters are written.
