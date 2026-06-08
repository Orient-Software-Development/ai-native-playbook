# AI-Native Development Playbook — Build Plan

> **Status:** In progress — as of 2026-06-05, `playbook/` contains this `PLAN.md`, the `00-foundations/` chapter (`01-why-ai-native`, `02-context-engineering`, `03-harness-engineering`), and `glossary.md`. The remaining chapters (`10-`–`50-`) and `README.md` are not yet scaffolded.
> **Owner:** Innovation Hub, Orient Software
> **Source mission:** [`../ai-native-playbook-mission.md`](../ai-native-playbook-mission.md)
> **Drafted:** 2026-06-04
> **Last synced:** 2026-06-05

This document is the plan for building the playbook that lives in this folder
(`docs/harness/playbook/`). It defines the structure, the page list, the
navigation model, where each page's content comes from, and the order of work.
It is **not** the playbook itself — nothing here ships to readers. Once approved,
we scaffold the pages described in §3.

---

## 1. What we are building

A **comprehensive, reusable, repo-agnostic playbook** for the AI-native software
development lifecycle: how to properly leverage AI coding agents across spec,
plan, code, verify, review, deliver, and operate — backed by harness engineering
and context engineering practices that are proven (in this repo and elsewhere),
not aspirational.

### 1.1 Hard constraints (from the mission)

- **Not about Orient One.** Orient One is the *evidence source*, never the subject.
  Every page states a general practice; repo specifics inform the worked example but are
  never named in reader-facing prose, and never become the lesson itself.
- **Not a tool review.** Method + harness, not "use tool X." Examples may name a
  tool, but every recommendation must survive a tool swap.
- **Two depths, one library.** Engineers live in the practice chapters; technical
  sponsors (CTO/CEO) read the scorecard + adoption cost. We serve both by ordering
  the library so leadership pages sit at the front and practice pages behind them.
- **No citation apparatus in the pages.** The published playbook does not carry
  reader-facing citations — no ADR/spec numbers, audit figures, external benchmarks, or
  source links in the prose. The practice and concept chapters therefore avoid bare
  checkable-metric claims ("saves 60%") entirely rather than cite them; an example carries
  the lesson by being concrete, not by being sourced. Sourcing that guides the writing lives
  in this plan (§3–§4). The single open exception is the leadership scorecard (§6 Q5).

### 1.2 Definition of done for the playbook

Mirrors mission §2. The playbook is "done" when a new project can adopt it in a
week and it contains: what works (with evidence), what doesn't (anti-patterns),
the harness (guides + sensors + setup), the workflow (spec→plan→code→verify with
templates), and a graduated adoption guide (minimum viable harness → grow it).

---

## 2. Information architecture

### 2.1 Folder layout

```
docs/harness/playbook/
  README.md                  ← entry page: intro + table of contents (the "first page")
  PLAN.md                    ← this file (build plan; not part of the published playbook)
  glossary.md                ← reader-facing glossary of terms of art; maintained as pages are written (§2.5)
  00-foundations/
    01-why-ai-native.md
    02-context-engineering.md
    03-harness-engineering.md
  10-lifecycle/
    01-spec-the-contract.md
    02-plan-before-code.md
    03-task-slicing.md
    04-code-with-the-agent.md
    05-verify-proof-not-vibes.md
    06-review-and-convergence.md
  20-harness/
    01-guides-feedforward.md
    02-repo-structure-and-legibility.md   ← code structure as feedforward (specs folder, co-located sensors)
    03-sensors-feedback.md                (the quality gate: unit → integration → performance → e2e)
    04-keep-quality-left.md
    05-behaviour-harness.md               (invariants/SCN, design-system guide, evidence images)
  30-delivery/                            ← NEW chapter: shipping + running AI-native, at team scale
    01-trunk-based-development.md
    02-ci-and-cd.md
    03-observability.md
    04-drift-and-health-sensors.md        ← scheduled sensors outside the change lifecycle (checklist §4 / CG1)
  40-anti-patterns/
    01-failure-modes.md
  50-adoption/
    01-minimum-viable-harness.md
    02-growing-the-harness.md
    03-responsible-team-and-ai-debt.md    ← NEW: human accountability, AI flop-zones, debt budget
    04-legacy-and-brownfield.md           ← adopting the harness into existing untested code (checklist §7 / CG2)
    05-leadership-scorecard.md
  _assets/                   ← diagrams, if any (optional)
```

**Naming rules**
- Numeric prefixes on folders and files fix reading order and make link targets stable.
- One lesson per file. A file answers one question and ends with where to go next.
- Kebab-case slugs; no spaces.

### 2.2 Navigation model

Every page is self-navigating so a reader never hits a dead end:

- **Top of page:** breadcrumb back to `README.md` + the chapter it belongs to.
- **Bottom of page:** a `← Previous` / `Next →` pair following numeric order, plus
  "Related" links to sibling lessons in other chapters.
- **README.md** is the single source of truth for the full table of contents; every
  page links back to it. (We keep the TOC in one place to avoid drift — same
  principle as `lesson-learned/lessons.json` being the index of record.)

### 2.3 Page template

Each lesson page follows a fixed shape so the library reads consistently. **Exception — the
conceptual chapter (`00-foundations/`) is not bound to this template.** Those pages are about
concepts and mindset rather than a step-by-step practice, so they read as intuitive essays with
natural section headings; they keep the thesis blockquote (with breadcrumb), at least one worked
**In practice** example, and the nav footer, but they need not carry the `The principle / Why it
works / How to apply it / Anti-patterns` scaffold. The practice chapters (`10-`–`50-`) follow the
template below.

```markdown
# <Lesson title — a practice, stated as a claim>

> One-sentence thesis. 

Breadcrumb: [Playbook](../README.md) › <Chapter>

## The principle
What to do, stated generally and tool-agnostically.

## Why it works
The mechanism — why this improves velocity / quality / rework.

## How to apply it
Concrete steps, do/don't, copy-pasteable snippets or templates.

## In practice
One real-world or practical example, walked through end to end, with an explanation
of how the example conveys the meaning of the page — what goes wrong without the
practice and what changes with it. Prefer a concrete scenario the reader recognises
(a specific kind of feature, bug, or review) over an abstract restatement. **No
reader-facing citations.** Do not name ADRs, spec numbers, audit figures, external
benchmarks, or source links in the published prose — the example must carry the lesson
on its own. The source material that informs a page lives in this plan (§3), not in the
page. (The lone possible exception is the leadership scorecard — see §6 open question.)

## Anti-patterns
The failure mode this practice prevents (links to 40-anti-patterns where deep).

---
← Previous · [Contents](../README.md) · Next →
Related: [...]
```

### 2.4 Voice & reading level

Write like a friendly senior engineer explaining something to a teammate — warm and direct, never
stiff or academic. **Default to concise**: short sentences, plain words, no jargon for its own sake.
But concise is not cryptic — **a junior developer (roughly a year of experience) should be able to
read any page and act on it without having to flip to the glossary.** The glossary (§2.5) is a
backstop for the reader who forgets a term, never a substitute for a page defining its own terms
in-line. So when an idea is genuinely new or subtle, slow
down and explain it plainly — define the term the first time, say *why* it matters, show a small
example — rather than compressing it into something only an expert can decode. The test for every
paragraph: *would a junior get it on the first read?* If not, add a sentence — don't add jargon.

This does **not** conflict with the mission's "no fluff" rule: fluff is words that carry no
information; friendliness is *tone*, and clarity is *information tuned to the reader*. Cut empty
words; keep the sentence that makes a hard idea land.

Quick do/don't:
- Say "the agent only sees what's in its context window," not "operates within a bounded token horizon."
- Define an acronym or term of art the first time it appears (SCN, fitness function, reward hacking), then use it freely — and add it to `glossary.md` in the same change (§2.5).
- One idea per sentence; one job per paragraph.
- Friendly, not flippant — contractions and "you" are welcome; forced jokes are not.
- A one-line concrete example beats a paragraph of definition.
- The `00-foundations/` essays may read more narratively, but hold the same reading level.

### 2.5 The glossary (`glossary.md`)

The playbook carries a single reader-facing **glossary** at `glossary.md`. Every term of art the
pages use — SCN, fitness function, reward hacking, feedforward/feedback, harness, sensor, guide,
context rot, attention budget, property-based testing, mutation testing, LLM-as-judge, trunk-based
development, and the rest — gets one short, plain-language entry there.

Purpose and boundary:
- **Convenience index, not a dependency.** Per §2.4 every page still defines each term in-line on
  first use; a reader must never *need* the glossary to act on a page. The glossary is where a reader
  who forgets a term looks it up.
- **One definition of record.** When two pages use the same term they must agree, and the glossary is
  the single place that definition lives — the same drift-avoidance discipline `README.md` applies to
  the TOC (and `lesson-learned/lessons.json` to the lessons index).
- **Plain, short, uncited.** One or two sentences per entry, junior-readable, no reader-facing
  citations (per §1.1). A one-line concrete example is welcome where it earns its place.
- **Alphabetical**, each entry linking to the page that owns the fullest treatment of the term.

**Maintenance rule (the part that must not slip):** the glossary is written *as the pages are
written*, never in a later sweep. Whenever a page introduces or first-defines a term of art, add or
update its glossary entry **in the same change**. A page is not done until its new terms are in the
glossary and the two definitions match. The cross-link pass (§5 step 8) and the clean-prose pass
(§5 step 9) verify completeness and consistency, but they are a backstop — by then the glossary
should already be whole.

---

## 3. Page-by-page plan

For each page: its thesis and the source material it distills. Sources already in
the repo are linked; "external" means we cite a published source.

### README.md — Introduction + Table of Contents (the first page)
- **Thesis:** what AI-native development is, who this playbook is for, how to read it
  (engineer path vs leadership path), and the full linked TOC.
- **Source:** mission §1, §1a, §2.

> **Foundations reading order (decided 2026-06-05).** Within `00-foundations/`, context
> engineering comes *before* harness engineering. Context is the more concrete, jargon-light idea
> ("the agent only knows what's in its window") and a genuine prerequisite — a guide is *delivered
> through* the context budget — so teaching it first makes the harness's feedforward half land
> harder. The arc escalates: why (mindset) → context (the primitive) → harness (the full
> framework) → the lifecycle (practice). Page 01 still crowns the harness as the overarching spine
> so the through-line stays intact.

### 00-foundations/01-why-ai-native.md
- **Thesis:** the four outcomes (velocity, time saved, quality, client value) and the
  shift from "AI autocompletes" to "AI executes a governed lifecycle." Includes the explicit
  contrast that **AI-native spec-driven development is not vibe coding** — the governed loop merges
  nothing on vibes (spec = contract, sensors = proof, human owns the call).
- **Source:** mission §1, §4 (metrics framing); **article** — OpenAI Codex *agent-first world*
  ("humans steer, agents execute"; the engineer's job shifts from writing code to designing
  environments, intent, and feedback loops).

### 00-foundations/02-context-engineering.md
- **Thesis:** context is an **attention budget**, not storage — the smallest set of high-signal tokens
  that maximises the chance of the right result. Three points the page must make concrete:
  - **Context rot is real and measured.** Performance degrades non-uniformly as input grows — *even
    on easy tasks and even far below the window limit* — so a big context window is not usable
    context. This is the empirical ground under "keep context lean"; it is *why* loading everything
    backfires rather than helps.
  - **Compaction vs reset, and three memory layers.** Distinguish (a) in-window working memory
    (ephemeral, budgeted), (b) an agent memory store (semi-durable retrieval), and (c) the
    **repo-as-system-of-record** (durable, reviewable, version-controlled — specs, ADRs, notes). The
    durable source of truth is the repo; the store is an accelerator, never the canon. Prefer
    reset + durable notes over endless compaction when a file substrate exists.
  - **More guides/skills is not better — it's a budget tax.** A concrete failure mode: a library of
    *hundreds* of reusable skills/guides loaded up front can consume tens of thousands of tokens
    *before the first prompt*, and agents are unreliable at routing — they often fail to select the
    right skill even when it is present, retrieval from a big pool misses good skills, and noisy
    general-purpose skills can actively mislead weaker models. The fix is the budget discipline of
    this page (progressive disclosure, just-in-time loading, lean curation), not a bigger catalogue.
    Skill *design* and the retrieval pipeline that serves them belong to `20-harness/01`; this page
    owns *why* the token cost and routing failure happen. Cross-link to the anti-pattern in
    `40-anti-patterns/01`.
- **Source:** lessons README "long specs lose the AI" theme; mission §2; **first-hand** — our own
  team's experience that a force-loaded skill library cost up to ~80k tokens before the first prompt
  and the agent routed poorly over it (generalise away from repo specifics — state it as a practice);
  **external** — Chroma Research *Context Rot* (degradation with input length across 18 models);
  Anthropic *Effective context engineering* (attention budget, the "right altitude", compaction /
  just-in-time retrieval / structured note-taking / sub-agents); Claude Code context-hygiene
  practices (focused context, `/clear` between tasks, point the agent at the right files) + the
  `CLAUDE.md`/`AGENTS.md` instruction-file pattern; *How Well Do Agentic Skills Work in the Wild*
  (arXiv 2604.04323 — skill benefits are fragile in realistic settings: selection, retrieval, and
  adaptation all fail; treat skill infra as a pipeline, not a checkbox); **article** — Anthropic
  *long-running apps* on context resets vs compaction and "context anxiety"; OpenAI Codex on
  repository knowledge as the system of record (durable context beats chat memory).

### 00-foundations/03-harness-engineering.md
- **Thesis:** the harness is everything except the model; guides (feedforward) +
  sensors (feedback); computational vs inferential; the three regulation categories;
  **harnessability** (introduced here as a concept — see note below); the steering loop.
  The capstone of foundations and the conceptual spine of the whole playbook;
  builds on context (guides + sensors both reach the agent through the context channel). The page
  must also make the **lever argument**: the harness is the team's *biggest controllable input*.
  You don't train the model — you choose it — but you fully own the harness, and the *same model in
  a better harness produces meaningfully better results*. This is why the playbook invests in the
  harness over chasing the next model: outcome quality is roughly as much about the scaffolding around
  the model as the model itself.
- **Source:** [`../harness-engineering-reference.md`](../harness-engineering-reference.md)
  §§1–6 (Böckeler), generalized away from the Orient One column; **first-hand** — our team's working
  rule of thumb that results are ~60% model / ~40%+ harness (record the ratio here; per the no-numbers
  rule the page states the qualitative lever argument, not the figure); **external** — harness
  sensitivity in public benchmarks: identical model weights swing ~10–20 points on SWE-bench Verified
  depending on the eval harness (scaffolding, retries, prompt, parsing), so a benchmark score is only
  meaningful paired with its harness; **article** — the original Böckeler *Harness engineering for
  coding agent users* mirrored in [`../references/`](../references/) (cite for the canonical framework;
  cite the reference doc for our framing).
- **Note — harnessability layering (decided 2026-06-05).** The *concept* of harnessability (reference
  §5.2 — strong types, clear module boundaries, established frameworks, high test coverage, and the
  "pays off twice" insight) is introduced here at altitude, because it's a concept and this is the
  concept chapter — the same way 03 introduces guides/sensors while `20-harness/01,03` give the
  concrete ones. The *practice* of raising it stays with the pages already assigned it:
  `20-harness/02-repo-structure-and-legibility` (legibility/boundaries) and
  `50-adoption/02-growing-the-harness` (investments that compound). Page 03 forward-links to both; those
  pages reference the concept back rather than re-explaining it, to avoid drift.

### 10-lifecycle/01-spec-the-contract.md
- **Thesis:** write a short, concrete spec before code; the spec is the contract;
  vague specs make the agent confidently wrong. Two refinements the page must make:
  - **A normative spec beats an ad-hoc one.** State requirements as clear *shall/should* obligations
    and acceptance criteria — what must hold, in business language — rather than a loose narrative or
    a pile of implementation detail. Stripping the technical jargon is not dumbing down: a spec a
    product owner or business head can read, challenge, and sign off on is a *better contract*,
    because the people who own the intent can actually verify it captures what they meant. Keep the
    *what* (normative, business-legible) separate from the *how* (technical, deferred to the plan).
  - **For UI features, a spec with a clickable prototype is better still.** An HTML mockup/prototype
    embedded in or linked from the spec conveys look-and-feel that prose can't, and lets you validate
    expectations with a product owner or client *before* any code exists — collapsing the
    "is-this-what-you-meant?" loop from a sprint to a conversation. The prototype becomes part of the
    contract the behaviour harness later checks against (ties to `20-harness/05` evidence images and
    the design guide in `20-harness/01`).
- **Source:** lessons README "Spec-driven" theme; lessons.json takeaways (all three); **first-hand**
  — our finding that normative, business-legible specs and HTML-prototype specs validated intent with
  product owners/clients faster than jargon-heavy prose specs (generalise; no repo specifics in the
  page); **external** — structured/normative acceptance-criteria practice (e.g. EARS-style
  *shall* requirements; the spec→plan separation of *what* from *how* in spec-driven toolkits) stated
  as method. Note the format choice interacts with §6 Q2 (MD vs HTML).

### 10-lifecycle/02-plan-before-code.md
- **Thesis:** a plan stage between spec and code; <30% planning effort to save 60–70%
  rework; show the plan before executing.
- **Source:** mission §4; the spec→plan→code→verify pipeline (Son); **external** — Claude Code
  "explore → plan → code → commit" workflow and plan mode (Anthropic best-practice guide).

### 10-lifecycle/03-task-slicing.md
- **Thesis:** one task = one deliverable, one test, one commit (<3 files / <200 LOC);
  slice by layer (schema → commands → API → UI), verify each before the next.
- **Source:** lessons README "Task slicing" theme.

### 10-lifecycle/04-code-with-the-agent.md
- **Thesis:** quote concrete reference behavior over "like X does it"; the agent won't
  reliably self-follow rules — that's what sensors and review are for.
- **Source:** lessons README "Working with the AI" theme; **external** — Claude Code (early
  course-correction, subagents for investigation, be specific in prompts) + Codex (scoped tasks,
  sandbox/approval modes, keep diffs reviewable) best-practice guides.

### 10-lifecycle/05-verify-proof-not-vibes.md
- **Thesis:** verification is non-negotiable — read logs, check state, test the spec's
  edge cases; AI-generated tests confirm the code, not the business need. Proof takes
  concrete forms: regular ad-hoc and **audit specs** (standing end-to-end walk-throughs re-run
  to catch drift), and **evidence artifacts** — captured screenshots/images for UI, saved log
  excerpts and state dumps for backend — attached to the change so a reviewer sees the proof, not
  a claim. A separate, skeptical evaluator (human, or an experimental local **LLM-as-judge** agent —
  see §6) grades against the spec; the author never self-certifies.
- **Source:** lessons README "Verification & sensors" theme; **article** — Anthropic *long-running
  apps* on the self-evaluation failure (agents confidently praise their own work) and why a separate,
  skeptical evaluator agent — not self-grading — is the lever that makes verification trustworthy.

### 10-lifecycle/06-review-and-convergence.md
- **Thesis:** deliberate diff review catches plausible-but-wrong "zombie code"; treat
  follow-up polish as convergence; extract the shared piece before the third copy.
- **Source:** lessons README "Reuse & convergence" + "Working with the AI" themes; **article** —
  Anthropic *long-running apps* generator/evaluator separation; OpenAI Codex on how throughput changes
  the merge philosophy and on entropy/garbage-collection (review and convergence at agent scale).

### 20-harness/01-guides-feedforward.md
- **Thesis:** the concrete guides — agent instruction files (CLAUDE.md/AGENTS.md),
  conventions, blueprints/curated examples, module-boundary context, and a **design guide**
  (a `DESIGN.md` / checked-in design tokens the agent reads before touching UI) — and how to set
  them up. Reusable **skills** are guides too, and the page must state the discipline: a skill is a
  *pipeline, not a checkbox* — good design (clear, narrowly-scoped, composable, with high-signal
  metadata) → strong retrieval/routing → smart selection-and-ignoring → optional per-task
  refinement. Design skills as general, composable patterns, **not single-task "answer keys"**;
  irrelevant or low-quality skills *hurt*, so monitor usage and downweight or discard the ones that
  don't earn their keep. The *token cost and routing-failure* side of this lives in
  `00-foundations/02`; this page owns skill *design and the retrieval pipeline*. Guides are also where
  **anti-cheat constraints** live — explicit rules that stop the agent hacking its way to green,
  because left unconstrained it optimises for *task-complete* over *task-correct*: e.g. "you may not
  modify or weaken a failing test to make it pass," "you may not patch source code merely to satisfy a
  test," "follow the test-writing rules." These constraints are the feedforward half of the anti-cheat
  story; the feedback half (tamper-resistant, state-asserting sensors) is `20-harness/03`, and the
  failure they prevent is catalogued in `40-anti-patterns/01`.
- **Source:** harness reference §2.1, §7 (Stripe blueprints, OpenAI linters); **first-hand** — our
  integration-testing harness rules (no modifying tests to pass; tests assert real state; follow the
  test-writing rules) generalised to tool-agnostic constraints; **external** — the
  `CLAUDE.md` (Anthropic) and `AGENTS.md` (OpenAI Codex) agent-instruction-file best practices:
  keep them concise, in-repo, checked in, and curated as the agent's first-read context; Anthropic
  *Agent Skills* (`SKILL.md`, progressive disclosure: name/description preloaded cheaply, body and
  linked files loaded only on match); *How Well Do Agentic Skills Work in the Wild* (arXiv
  2604.04323 — design for clarity/reuse/composability; agentic hybrid retrieval helps, query-specific
  refinement multiplies *existing* skill quality but adds no new knowledge); **article** — OpenAI
  Codex on repository-as-system-of-record and application/agent *legibility* (structuring the repo so
  the agent can navigate and extend it).

### 20-harness/02-repo-structure-and-legibility.md
- **Thesis:** how the repo is structured *is* feedforward — an industry-standard, predictable layout
  is the agent's map. A dedicated `specs/` folder (the contracts), sensors co-located with the code
  they guard (tests beside modules), clear package/module boundaries, and consistent naming let the
  agent find the right prior context and extend it without guessing. A legible repo turns "the agent
  can't find it" into "the agent navigates it." One refinement the page should make:
  - **A machine-generated structural index complements the human-authored layout.** Beyond a tidy
    folder tree, you can give the agent a *code graph* — an AST-derived index of the symbols and how
    they connect (files → functions → classes → call chains) that the agent queries instead of
    grepping the tree file-by-file. It's feedforward the repo generates about *itself*: a concise
    navigable reference that cuts the tokens an agent burns locating the right prior context and
    sharply improves navigation on a large codebase. Tool-agnostic per §1.1 — the practice is "give
    the agent a structural index it can query," not any one indexer; every recommendation must
    survive a tool swap, and no product names, license terms, or token-saving percentages appear in
    the page.
- **Source:** harness reference §2.1 (feedforward), §5.3 (harnessability investments); the `specs/`
  + product-package conventions generalized away from repo specifics; **article** — OpenAI Codex on
  repository-as-system-of-record and *legibility* (structure the repo so the agent can navigate and
  extend it); **external** — standard project-layout conventions stated as method; **external (code-graph
  tooling — writer apparatus only, names/licenses/figures stay OUT of the page per §1.1/§4):**
  open-source AST-based codebase knowledge-graph indexers that build a symbol-level graph an agent
  can query as an MCP server or CLI — e.g. *CodeGraphContext* (MIT; Python; KùzuDB embedded graph DB;
  Python/TS/JS/Rust/Go/C++/C), *codebase-memory-mcp* by DeusData (MIT; single Go binary + embedded
  SQLite; positioned for large token reduction vs file-by-file search), and *Graphify* (MIT; indexes
  code + docs + multimedia into one graph; works with Claude Code / Cursor / Aider). Note GitNexus,
  a prominent earlier option, moved to a restrictive PolyForm Noncommercial license — a reason to
  prefer the permissive MIT alternatives above for commercial use. Distil to the general practice
  ("an AST code-graph index as queryable feedforward"); the page names none of these and cites none
  of their figures.

### 20-harness/03-sensors-feedback.md
- **Thesis:** the concrete sensors — the full **quality gate** (unit → integration → performance →
  end-to-end tests), linters, type checkers, fitness functions, AI semantic reviewers — computational
  first, inferential where structural can't reach. Each tier catches a class the cheaper tier can't;
  a "super good" gate is the layered pyramid, not one fat suite. An experimental inferential sensor —
  a local **LLM-as-judge** agent scoring diffs/output against the spec — is noted as to-be-validated
  (see §6), not yet recommended method.
- **Sensor integrity — a sensor the agent can satisfy without doing the work is not a sensor.** The
  agent optimises for *task-complete*, so it will happily make a check pass cheaply: an integration
  test that only asserts "ran without throwing" is green whether or not the write actually persisted.
  Sensors must assert the **observable end-state** — verify real persisted/DB state and externally
  visible outcomes, not the mere absence of an exception. A weak assertion is a false sensor: it
  reports safety it cannot back up (the same lesson UTBoost showed at benchmark scale — weak gating
  tests let wrong patches pass). This pairs with the *anti-cheat constraints* in `20-harness/01`
  (the agent may not weaken a test to pass) and the reward-hacking failure mode in `40-anti-patterns/01`.
- **Source:** harness reference §2.2, §3, §4; **first-hand** — our integration-testing harness work
  on a current project: tests must verify actual DB state rather than just run without throwing
  (generalise; no repo/client specifics in the page); **external** — the test pyramid
  (unit/integration/e2e) plus performance/load testing as standard quality practice; UTBoost
  (arXiv 2506.09289) on weak tests passing incorrect patches.

### 20-harness/04-keep-quality-left.md
- **Thesis:** distribute checks by cost/speed (pre-commit → pre-push → CI → staging);
  don't pile the heavy suite on git hooks. The pyramid from the previous page maps onto the pipeline:
  fast checks left/early, slow ones (performance, full e2e) in CI/CD where they pay off.
- **Source:** harness reference §5.1; ties forward to `30-delivery/02-ci-and-cd`.

### 20-harness/05-behaviour-harness.md
- **Thesis:** the hardest category — making the agent build what the business wants. The lever is the
  **invariant**: identify what must always hold for a scenario, name the scenario (the SCN), and
  describe its invariant precisely — that description becomes the contract a fixture-scenario-style
  behaviour test proves. For UI, the design guide (`DESIGN.md` / tokens, see page 01) plus captured
  **evidence images** turn subjective "looks right" into gradable criteria. Two techniques make
  invariant tests *resist the AI's favourite failure* — tests that merely confirm what the code
  already does:
  - **Property-based testing** asserts an invariant (round-trip identity, "output never negative",
    monotonicity) and generates the inputs, so the test is derived from the *spec*, not the
    implementation, and cannot silently encode the code's current bugs. Ground the properties in
    docstrings, types, and names — semantics independent of the function body.
  - **Mutation testing tests the tests:** inject small deliberate bugs; if the suite still passes,
    the tests are too weak. Surviving mutants are the quantitative signal that a green, high-coverage
    suite is actually toothless — the empirical defence against "confirms-the-implementation" tests
    (and the reason coverage alone is false comfort — see §6 Q6). The virtuous loop: PBT for
    spec-derived properties → mutation score to grade suite strength → agent tasked to kill the
    survivors.
  Why human verification stays.
- **Source:** harness reference §4.3, §6; "frontend is the weak point" open problem; the
  fixture-scenarios / SCN-as-contract convention generalized; **external** — property-based testing
  as the structural antidote to implementation-mirroring tests (Anthropic Red *Property-Based Testing
  with Claude*: properties grounded in docstrings/types found real merged bugs in NumPy/AWS, ~86%
  valid among top-ranked reports; *Agentic Property-Based Testing*, arXiv 2510.09907); mutation
  testing as the measure of test strength (surviving mutants = weak assertions); the observation that
  even SWE-bench Verified contained ~3% false-passing patches because the gating tests were too weak
  (UTBoost, arXiv 2506.09289) — "a passing test is not proof if the test is weak"; **article** —
  Anthropic *long-running apps* on turning subjective quality (frontend/design) into gradable
  criteria and using a separate evaluator agent — a concrete attack on the frontend self-verify
  blind spot; OpenAI Codex on enforcing architecture and taste.

### 30-delivery/01-trunk-based-development.md
- **Thesis:** trunk-based development — short-lived branches, small slices merged continuously behind
  a green gate — is the git strategy that *empowers a high-performing AI-native team*. Many humans and
  agents converge on one trunk precisely because each change is small, verified, and merged before it
  can rot; long-lived branches are where agent output goes stale and conflicts compound. The team's
  velocity comes from continuous integration, not from working in isolation.
- **Source:** lessons README "Task slicing" theme (small slices → small merges); the feature-branch /
  PR-into-integration flow generalized away from repo specifics; **external** — trunk-based development
  and continuous integration (DORA capabilities) stated as method; **article** — OpenAI Codex on how
  throughput changes the merge philosophy (many small PRs/day) and on entropy/garbage-collection at
  agent scale.
- Related: task-slicing, ci-and-cd, review-and-convergence.

### 30-delivery/02-ci-and-cd.md
- **Thesis:** separate **continuous integration** (verify every change — build, lint, test on each
  push/PR) from **continuous delivery/deployment** (release the verified artifact to environments). CI
  is the authoritative *merge* gate; CD is the path to production. Conflating them either blocks merges
  on deploy concerns or ships unverified code. Draft-aware gating keeps the heavy suite where it pays
  off, so the gate stays both strict and fast.
- **Source:** harness reference §5.1 (keep-quality-left; the pipeline as the authoritative gate); the
  draft-aware CI model generalized; **external** — CI/CD separation as established delivery practice.
- Related: keep-quality-left, sensors-feedback, trunk-based-development.

### 30-delivery/03-observability.md
- **Thesis:** logging and monitoring are the runtime feedback loop — sensors that extend *past merge*
  into production. Structured logs and metrics let humans and agents see what the system actually does,
  turning "it deployed" into "it works," and feed diagnosis back into the next spec. Without
  observability the harness goes blind the moment code leaves CI.
- **Source:** harness reference §2.2 (sensors), extended from build-time to runtime; **external** —
  observability / monitoring (logs, metrics, traces) as standard operational practice; ties back to
  evidence-based verification (`10-lifecycle/05`).
- Related: sensors-feedback, verify-proof-not-vibes, drift-and-health-sensors.

### 30-delivery/04-drift-and-health-sensors.md
- **Thesis:** the sensors that run on a *clock*, not a diff — the category outside the change lifecycle.
  A change-triggered sensor can only catch what a change *causes*; some decay is cumulative (dead code
  accreting across many diffs, an eroding coverage/mutation *trend*) or external (an upstream CVE with no
  local change at all), so the only sensor that can see it is one whose trigger is time. Members:
  scheduled dead-code detection, dependency/vulnerability scanning on a cadence, coverage/mutation-score
  *trend* monitoring, and an architecture-drift "janitor" that flags drift and — as a scheduled agent run
  — opens the PR that fixes it. Findings must land as *action* (a PR/issue), never a dashboard; keep
  non-deterministic scans advisory (propose, don't auto-merge). Runtime observability is the sibling
  member with its own page (`30-delivery/03`).
- **Source:** harness reference §4 (continuous drift & health sensors); checklist §4 / `checklist-gaps.md`
  CG1; **first-hand** — the repo's own Dependabot cadence generalised to the practice; **external** —
  dead-code/dependency scanning and mutation/coverage trend monitoring as standard cadence practice;
  Codex entropy/"garbage-collection" framing for the janitor at agent scale.
- Related: observability, sensors-feedback, behaviour-harness (mutation), growing-the-harness (agent
  builds/maintains the harness).

### 40-anti-patterns/01-failure-modes.md
- **Thesis:** the catalog of what doesn't work — long specs, zombie code, vibe
  verification, AI tests that only confirm the implementation, frontend self-verify
  blindness, **AI-generated technical debt that accumulates silently** (plausible code that
  compounds because no one is accountable for it), and the **"more guides/skills is better" trap**
  (force-loading a large skill/instruction library up front — tens of thousands of tokens before the
  first prompt — when agents route poorly over big pools and noisy skills actively mislead, so a
  bigger catalogue lowers quality instead of raising it) — each with the practice that prevents it.
  The headline agent-era failure gets top billing: **reward hacking — the agent cheats its way to
  green.** Faced with a failing test it optimises for *task-complete*, not *task-correct*: it rewrites
  the test so it passes but verifies nothing, or patches the source just enough to satisfy the test —
  green CI, a new silent bug. It isn't malicious, it's Goodhart's law (when the test becomes the
  target it stops measuring correctness); an unconstrained agent does this *every time*. The prevention
  is the two-sided anti-cheat harness: **feedforward constraints** (`20-harness/01` — may not weaken a
  test or patch source merely to pass) plus **honest, state-asserting sensors** (`20-harness/03` —
  verify real end-state, not "ran without throwing").
- **Source:** lessons README "Open problems"; lessons.json takeaways framed as failures; **first-hand**
  — our team's ~80k-tokens-before-first-prompt skill-bloat experience, and the reward-hacking pattern
  observed building an integration-testing harness (agent rewrote tests / patched source to force
  green) — both generalised, no repo/client specifics in the page; **external** —
  *How Well Do Agentic Skills Work in the Wild* (arXiv 2604.04323); reward hacking / specification
  gaming as a documented agent-alignment failure (Goodhart's law). Pairs with
  `50-adoption/03-responsible-team-and-ai-debt` (the team-side response to these flop-zones) and the
  budget/design treatment in `00-foundations/02` + `20-harness/01`.

### 50-adoption/01-minimum-viable-harness.md
- **Thesis:** the smallest harness a new project stands up in week one (one guide file +
  lint + type check + a pre-push gate + one behaviour test).
- **Source:** mission §2.5; harness reference §4.1 "good starting harness."

### 50-adoption/02-growing-the-harness.md
- **Thesis:** the steering loop in practice — observe failure → add guide or sensor →
  re-run; harnessability investments (types, boundaries, coverage) that compound.
- **Source:** harness reference §5.2, §5.3; **article** — OpenAI Codex on graduated *levels of
  autonomy* (earn trust before widening scope) and Anthropic *long-running apps* on iterating on the
  harness itself (observe failure → tune a guide/sensor → re-run).

### 50-adoption/03-responsible-team-and-ai-debt.md
- **Thesis:** the team stays accountable for what the agent ships — responsibility does not transfer
  to the model. Name the zones where AI reliably flops (subjective frontend/design, cross-cutting
  refactors, ambiguous business rules, security-sensitive code) and write a deliberate harness plan
  for each weak zone; and treat AI-generated technical debt as a *managed budget* — make it visible,
  schedule its paydown, don't let it compound. The responsible team decides what to automate, what to
  guard, and what stays human.
- **Source:** mission §1a + "humans steer, agents execute" framing; lessons README "Open problems"
  (where AI flops); harness reference §5.2/§5.3 (steering loop, harnessability investments) applied to
  risk zones; **article** — Anthropic *long-running apps* (self-evaluation failure, frontend blind
  spot) and OpenAI Codex on autonomy levels (earn trust before widening scope) and enforcing
  architecture/taste.
- Related: failure-modes, growing-the-harness, behaviour-harness.

### 50-adoption/04-legacy-and-brownfield.md
- **Thesis:** the harness is hardest to build exactly where it's needed most — an existing, large,
  sparsely-tested, untyped codebase. You don't close that gap with a rewrite; you earn a foothold and
  widen it: **characterization tests** pin current behaviour (bugs and all) so a refactor's effects
  become *visible*; **boundaries are drawn incrementally**, one real seam at a time; **high-risk seams go
  first** (money, auth, data integrity — the flop zones), and the low-risk tangle stays un-harnessed until
  it changes. The agent is the lever that makes it affordable — characterization tests are archaeology, a
  task it's good at — which ties this page to "use the agent to build the harness" (`50-adoption/02`).
  `50-adoption/01` is the greenfield on-ramp; this is its brownfield counterpart.
- **Source:** harness reference §7 (legacy: characterization tests, incremental boundaries, highest-risk
  seams); checklist §7 / `checklist-gaps.md` CG2 (also flagged in `GAPS.md`); **external** — legacy-code
  practice (characterization tests; incremental seam-by-seam refactoring) stated as method.
- Related: minimum-viable-harness, growing-the-harness, responsible-team-and-ai-debt, repo-structure-and-legibility.

### 50-adoption/05-leadership-scorecard.md
- **Thesis:** the thin strategic layer — the metrics to instrument and adoption cost,
  written for a technical sponsor who will check the numbers.
- **Source:** mission §1a, §4; the measured scorecard, gating model, and trend data in
  [`../audit-reports/`](../audit-reports/) (latest: `harness-audit-2026-06-04.md`); **article** —
  OpenAI Codex *agent-first world* throughput benchmarks (≈3.5 PRs / engineer / day, ≈1/10th
  hand-coding time) as an external comparator for the velocity numbers — labelled external, not ours.

---

## 4. Content sourcing rules

> **Note on citations.** The rules below govern how sources *inform the writing*. None of this
> attribution appears in the published pages — per §1.1, the pages carry no reader-facing
> citations (no ADR/spec numbers, audit figures, external benchmarks, or source links). Sourcing
> is the writer's working apparatus, recorded here in the plan; the reader gets the distilled
> practice and a concrete example, nothing to chase down.

- **Distill, don't copy.** Pull the *practice* out of the three retrospectives and the
  Böckeler reference; keep the page body general and example-driven. Orient-One specifics may
  *inform* a worked example (a realistic feature, bug, or rule) but are never named in the prose
  and never become the lesson itself.
- **Fold in published agent-coding best practices, tool-agnostically.** Two canonical vendor
  guides feed the method (take the practice, drop the product name — see mission §4a and the
  non-goals): **Anthropic — Claude Code best practices**
  (https://code.claude.com/docs/en/best-practices) and **OpenAI — Codex best practices**
  (https://developers.openai.com/codex/learn/best-practices). Where a vendor practice converges
  with our retrospectives, state it plainly as established method; where only one vendor backs it,
  still state it as method — the proven-vs-single-source distinction is tracked here in §3 to keep
  the writer honest, not attributed in the page. Per-page mapping is in §3.
- **Three reference articles are mirrored in [`../references/`](../references/)** (full descriptions in
  mission §4a): **Böckeler — *Harness engineering for coding agent users*** (the framework source
  behind `harness-engineering-reference.md`); **Anthropic — *Harness design for long-running
  application development*** (planner/generator/evaluator, context resets vs compaction, the
  self-evaluation failure); **OpenAI — *Harness engineering: leveraging Codex in an agent-first
  world*** ("humans steer, agents execute", repository-as-system-of-record, legibility, autonomy
  levels, throughput benchmarks). These mirrors are the writer's stable reference set; the pages
  link to none of them.
- **No numbers in the practice/concept chapters.** A score, count, ratio, or "this is gated" claim
  does not appear in the published practice (`10-`–`40-`) or concept (`00-`) pages — they teach
  method, not measurements, so there is nothing to cite. Candidate metrics live only in the
  leadership scorecard, framed as *targets to instrument* (mission §4 lists candidates, not
  yet-measured ones); whether that one page may surface measured audit figures, and how without a
  citation apparatus, is the §6 Q5 open question.
- **Convergence = confidence, not a citation.** Where the three retrospectives agree (per the
  lessons README themes), state the practice plainly as established. Don't badge claims in-page with
  "(proven)" or "(single-source)" — that confidence calculus is the writer's, recorded in §3.
- **First-hand findings count as evidence too.** Where our own team's experience supplies the lesson
  (e.g. the skill-bloat finding in `00-foundations/02` / `40-anti-patterns/01`), distil it to the
  general practice and the worked example — never name repo specifics or quote the raw token figure
  in the prose. The first-hand origin is recorded in §3, not the page.
- **Additional external sources (added this revision; URLs for the writer, not the page).** These
  feed the strengthened context-engineering, skills, and testing pages and are not mirrored in
  `../references/`:
  - Chroma Research, *Context Rot* — https://www.trychroma.com/research/context-rot
  - Anthropic, *Effective context engineering for AI agents* —
    https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents
  - Anthropic, *Equipping agents for the real world with Agent Skills* —
    https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills
  - *How Well Do Agentic Skills Work in the Wild* — https://arxiv.org/pdf/2604.04323v1
  - Anthropic Red, *Property-Based Testing with Claude* — https://red.anthropic.com/2026/property-based-testing/
  - *Agentic Property-Based Testing* — https://arxiv.org/pdf/2510.09907
  - UTBoost (weak tests pass bad patches on SWE-bench) — https://arxiv.org/pdf/2506.09289
- **Keep the index in one place.** `README.md` owns the TOC; pages link to it. Mirror
  the discipline already used for `lesson-learned/lessons.json`.
- **Keep the glossary in one place, and current.** `glossary.md` owns every term-of-art definition
  (§2.5); pages define inline on first use and link to the glossary entry. Maintain it *as you write* —
  add each new term in the same change that introduces it, never in a cleanup pass — so the definition
  of record never drifts from the prose.

---

## 5. Build order

1. **Approve this plan** (and the page list / naming in §2–§3).
2. **Scaffold the skeleton** — create `README.md` with the full TOC and create every
   page as a stub (title + thesis + section headers + nav footer), so navigation is
   wired end-to-end before any prose is written. Create `glossary.md` as an empty,
   headed stub at the same time, ready to grow with the chapters.
3. **Write the foundations chapter** (`00-foundations/*`) — the conceptual spine the
   rest links back to.
4. **Write the lifecycle chapter** (`10-lifecycle/*`) — highest-value for engineers.
5. **Write the harness chapter** (`20-harness/*`).
6. **Write the delivery chapter** (`30-delivery/*`) — how the verified work ships and runs at team scale.
7. **Write anti-patterns + adoption** (`40-*`, `50-*`).
8. **Cross-link pass** — fill Related links and verify every Previous/Next/Contents
   link resolves.
9. **Example + clean-prose pass** — confirm every page's **In practice** section carries a
   concrete worked example that conveys the lesson on its own; that the prose meets the §2.4 voice
   and reading-level bar (friendly, concise, junior-readable — every term of art defined on first
   use); and that no reader-facing citation, ADR/spec number, audit figure, external benchmark, or
   source link has crept into the prose anywhere (per §1.1 / §4). The scorecard exception, if any, is
   settled in §6 Q5. Also confirm `glossary.md` is complete and consistent: every term of art used in
   the prose has a matching entry, and no entry contradicts the page that owns it.

> **The glossary runs alongside, not after.** Steps 3–7 each maintain `glossary.md` as they go
> (§2.5): every term a chapter first-defines lands in the glossary in the same change. The glossary
> is never deferred to a later pass — step 9 verifies it, it does not build it.

Each step is reviewable on its own; we stop and confirm after the skeleton (step 2)
before committing to prose.

---

## 6. Open questions for you

1. **Scaffold depth:** stub every page now (full skeleton), or build chapter-by-chapter
   and only stub the chapter in flight? (Plan assumes full skeleton first.)
2. **HTML vs MD:** the existing retrospectives are HTML; this playbook is MD-first. Keep
   it MD-only, or also generate an HTML render later (matching `lesson-learned/`)? (Sub-question
   for the design guide referenced in `20-harness/01`/`05`: ship the `DESIGN.md` example as Markdown,
   or as design tokens in an HTML/CSS snippet the agent can read directly?) Note this is no longer
   purely a render-format choice: `10-lifecycle/01` now recommends **HTML specs with a clickable
   prototype** for UI features (to validate look-and-feel with product owners/clients before code),
   which gives HTML a *content* rationale, not just a styling one. Decide whether the playbook's own
   spec template ships a normative-Markdown form, an HTML-prototype form, or both side by side.
3. **Page count:** 24 pages as listed (was 17, then 22; the checklist-gap fix adds
   `30-delivery/04-drift-and-health-sensors` and `50-adoption/04-legacy-and-brownfield`, bumping the
   leadership scorecard to `05`), or do you want it tighter (fewer, denser pages) or broader (split
   lifecycle/delivery steps further)?
4. **Diagrams:** worth adding the steering-loop / keep-quality-left diagrams as assets,
   or keep it text-only for v1?
5. **Leadership scorecard vs the no-citations rule (decided for the rest of the playbook, open
   only here):** §1.1 and §4 ban reader-facing citations and keep measured numbers out of every
   chapter. But the leadership scorecard (`50-adoption/05`) is written for a sponsor who "will
   check the numbers" (mission §1a). Does that one page get an exception to show measured audit
   figures — and if so, how do we keep them verifiable without the citation apparatus the rest of
   the playbook drops? Candidate answers: (a) keep the ban absolute and let the audit reports
   themselves be the sponsor's source, referenced once in prose as "the audits" with no inline
   cites; (b) a single linked *methodology & sources* note the scorecard points to once, keeping
   the page bodies clean; (c) show numbers only as *targets to instrument*, never as measured
   claims. Leaning (a)/(c); confirm when we reach the adoption chapter.
6. **Unit-test coverage threshold:** does the playbook recommend a hard line-coverage gate at all?
   A percentage gate is easy to game with low-value, AI-generated tests that confirm the
   implementation rather than the spec — which cuts against the sensors/behaviour-harness thesis that
   tests must prove the invariant, not chase a number. Candidate answers: (a) drop the coverage
   threshold entirely and gate on behaviour/invariant coverage instead; (b) keep a low floor purely as
   a "did anyone test this at all" smoke signal, never as a quality claim; (c) leave it to each team.
   Leaning (a)/(b); this question shapes `20-harness/03-sensors-feedback`.
7. **LLM-as-judge as a standing sensor:** the *concept* of a separate skeptical evaluator is
   established method (`10-lifecycle/05`, Anthropic long-running apps). But wiring a **local agent as
   an automated LLM-as-judge** into the pipeline is still experimental ("to be tested"). Do we present
   it as recommended method, or explicitly flag it as an experiment to validate (false-positive rate,
   cost, determinism) before it earns a place in the gate? Plan currently flags it as experimental in
   `20-harness/03` and `10-lifecycle/05`; confirm.

---

## 7. Team lessons folded in (running log)

As the team collects first-hand lessons, each is recorded here and folded into the relevant §3
page(s). This is the registry behind the **first-hand** source tags in §3 — it keeps the sourcing
auditable and gives new lessons a clear home before they are distilled into prose. **Rule:** distil
to the general, tool-agnostic practice; the raw story, repo/client names, and exact figures live here
in the plan, never in the published page (per §4).

| # | Lesson (as told) | Distilled practice | Folded into | External corroboration |
|---|---|---|---|---|
| L1 | Hundreds of skills force-loaded before the first prompt cost up to ~80k tokens, and the agent routed/selected poorly over them. | More guides/skills is not better — it's a budget tax; treat skills as a designed retrieval pipeline, not a checkbox. | `00-foundations/02` (budget + routing), `20-harness/01` (skill design), `40-anti-patterns/01` (the trap) | *How Well Do Agentic Skills Work in the Wild* (arXiv 2604.04323) |
| L2 | Normative, low-jargon specs let product owners/business heads actually sign off; an HTML spec with a clickable prototype validated look-and-feel with the client before any code. | Write normative, business-legible specs (separate *what* from *how*); for UI, attach an HTML prototype to validate intent pre-code. | `10-lifecycle/01` (spec craft); open question §6 Q2 (MD vs HTML/prototype template) | EARS-style normative requirements; spec-driven *what/how* split |
| L3 | Building an integration-testing harness, the agent cheated to green — rewriting tests to verify nothing, or patching source just to satisfy a test. Constraining *how* it works mattered more than the model; rule of thumb ~60% model / ~40%+ harness. | Reward hacking is the headline agent-era failure; prevent it with a two-sided anti-cheat harness (feedforward constraints + honest, state-asserting sensors). The harness is the team's biggest controllable lever. | `40-anti-patterns/01` (reward hacking), `20-harness/03` (sensor integrity), `20-harness/01` (anti-cheat constraints), `00-foundations/03` (harness > model) | UTBoost (weak tests pass bad patches); SWE-bench harness sensitivity (±10–20 pts); Goodhart's law / specification gaming |

> **Next lessons:** append a row, distil the practice, and link the page(s) it informs. When a single
> lesson grows into a full treatment (e.g. reward-hacking + sensor integrity), consider graduating it
> from a strengthened existing page into its own dedicated page rather than overloading the host.
