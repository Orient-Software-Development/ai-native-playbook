# Harness Engineering & Spec-Driven Development
### A 30-minute talk — slide-by-slide outline with talking points

| | |
|---|---|
| **Audience** | Internal engineering teams who already use AI coding tools daily (Copilot / Cursor / Claude Code / Codex) |
| **Their level** | Comfortable *prompting* an agent; mostly **not** yet building a system *around* it |
| **Goal (the one CTA)** | Get teams to stop tuning prompts and start engineering the **harness** — run the loop **Spec → Plan → Code → Review → adjust the spec/harness (never hand-patch the code)** |
| **Framing** | External-first: lead with the industry voices (Osmani, Böckeler/Thoughtworks, Grove, Beck, Thinking Machines). The internal playbook is offered at the end as the "how we do it here." |
| **Format** | Slides only — no live demo. Use short **recorded clips / screenshots** as evidence where noted. |
| **Duration** | ~25 min content + ~5 min Q&A. Running clock shown per slide. |
| **Slide count** | 24 content slides across 7 acts |

> **The spine of the talk in one sentence:** *An LLM is a nondeterministic statistical model; you can't make the model reliable, so you engineer everything **around** it — the harness — and you drive it with a spec and tests instead of prompts.*

---

## Act 0 — Hook (0:00 → 2:00)

### Slide 1 — Title
- **Title:** *Harness Engineering & Spec-Driven Development — how to get reliable work out of an unreliable model*
- **Talking point:** "Everyone in this room already uses an AI coding tool. So I'm not here to sell you on AI. I'm here to explain why your results are inconsistent — and what the best teams in the world are doing about it."
- **Visual:** Title + your name. One-line subtitle: *Everything around the model is the part you actually control.*

### Slide 2 — The uncomfortable demo we've all seen
- **Key message:** One prompt, two runs, two different programs — one correct, one that *looks* correct and is silently wrong. You've all lived this.
- **The example to put on the slide — one prompt, two outputs:**

  **Prompt (identical both times):** *"Write a Python function `median(nums)` that returns the median of a list of numbers."*

  **Run A — correct:**
  ```python
  def median(nums):
      s = sorted(nums)
      n = len(s)
      mid = n // 2
      return s[mid] if n % 2 else (s[mid - 1] + s[mid]) / 2
  ```
  **Run B — looks fine, silently wrong:**
  ```python
  def median(nums):
      nums.sort()
      return nums[len(nums) // 2]
  ```
- **Talking points (why this is the perfect example):**
  - Both pass the obvious test: `median([3, 1, 2])` → **2** in both. So a quick eyeball, and even a naive unit test, says "ship it."
  - But `median([1, 2, 3, 4])`: Run A → **2.5** (correct), Run B → **3** (wrong — it never averages the two middle values). Run B *also* mutates the caller's list in place with `.sort()`. Two bugs, zero syntax errors, fluent and confident.
  - "Same prompt. No typo, no bad luck on my part. The tool is *allowed* to hand me either of these — and the broken one is the one that looks cleaner."
  - "This isn't a bug in the tool. This is *what the tool is*. If we don't accept that, everything we build on top of it is superstition."
- **Visual:** two code cards side by side under one prompt; green ✓ on A, red ✗ on B with the failing input `[1,2,3,4] → 3` circled.
- **Transition:** "So let's be precise about *why* the same prompt can do this."

---

## Act 1 — Why the output is random (2:00 → 6:00)  → *your point #1*

### Slide 3 — An LLM is a statistical model, not a program
- **Key message:** It doesn't *retrieve* an answer; it *samples* the next token from a probability distribution.
- **Talking points:**
  - A program with the same input gives the same output. An LLM samples — temperature, top-p — so the "same input" produces a *distribution* of outputs, not one.
  - Non-determinism isn't a defect bolted on; it's the generative mechanism itself.
- **Visual:** A next-token probability bar chart → arrow → "sampled."

### Slide 4 — "But I set temperature to 0…" — it's still not deterministic
- **Key message:** Even at temp 0, the same prompt returns *dozens* of different answers in practice.
- **Concrete example to show:**
  - Thinking Machines Lab (Mira Murati's lab) ran one prompt — **"Tell me about Richard Feynman"** — **1,000 times at temperature 0** on Qwen3-235B. Result: **80 distinct completions** (the most common appeared only 78 times).
  - The kicker to read aloud: all 1,000 were *word-for-word identical for the first 102 tokens* — "...Feynman was born on May 11, 1918, in ___" — then at token 103, **992 said "Queens, New York" and 8 said "New York City."** Same prompt, same temperature, same model. The dice are still rolling.
  - Root cause: outputs change with **batch size**, which changes with server traffic — GPU kernels aren't batch-invariant. So part of the randomness is *outside your control entirely*, even in principle.
  - Punchline: "You cannot make the model reliable by configuring the model. Stop trying."
- **Cite:** [Thinking Machines Lab — *Defeating Nondeterminism in LLM Inference*](https://thinkingmachines.ai/blog/defeating-nondeterminism-in-llm-inference/)
- **Visual:** one prompt → 1000 runs (temp=0) → 80 outputs; a branch diagram at token 103 splitting "Queens, New York" (992) vs "New York City" (8).

### Slide 5 — The reframe: don't reduce randomness, *constrain the output space*
- **Key message:** The question is not "how do I make the model deterministic?" It's **"how do I scope the output down so the randomness lands inside an acceptable range?"**
- **Talking points:**
  - Analogy: you can't control which exact way a river flows, but you can build banks so it always reaches the sea. Guides = the banks. Sensors = the dam that rejects overflow.
  - Every technique in the rest of this talk is one move: **shrink the space of things the agent can plausibly produce, and grade what comes out.**
  - That discipline has a name.
- **Transition:** "It's called harness engineering."

---

## Act 2 — Agent = Model + Harness (6:00 → 12:00)  → *your point #2*

### Slide 6 — The equation
- **Key message:** **Agent = Model + Harness.** The harness is *everything that isn't the model.*
- **Talking points:**
  - Addy Osmani: *"A decent model with a great harness beats a great model with a bad harness."*
  - Viv Trivedy's line (via Osmani): *"If you're not the model, you're the harness."*
  - The model is the lever you *don't* control and everyone shares. The harness is the lever you *do* control and a competitor can't copy. That's where the leverage is.
- **Cite:** [Addy Osmani — *Agent Harness Engineering*](https://addyosmani.com/blog/agent-harness-engineering/)
- **Visual:** `Agent = [ Model ] + [ Harness ]`, with "you can only pick this" vs "you fully own this."

### Slide 7 — The harness taxonomy (the map)
- **Key message:** "Harness" isn't fuzzy — it's seven concrete categories. Everything you add to your agent is one of these.
- **Talking points (walk the list fast, one line each):**
  1. **Context & knowledge** — system prompts, `AGENTS.md`/`CLAUDE.md`, skill files, memory
  2. **Execution environment** — filesystem, git, sandbox, bash, code execution
  3. **Tool integration** — MCP servers, tool descriptions, skill registries
  4. **Control flow** — orchestration, subagents, hooks, middleware
  5. **Observability** — logging, tracing, cost/latency metering
  6. **Long-horizon work** — planning, verification loops, context compaction
  7. **Safety & enforcement** — permission gates, approvals, destructive-action blocks
- **Cite:** [Addy Osmani — *Agent Harness Engineering*](https://addyosmani.com/blog/agent-harness-engineering/)
- **Visual:** 7-box grid. This slide is the reference map for the next 5 slides.

### Slide 8 — Example 1: Context — the `AGENTS.md` / guide file
- **Key message:** A single instruction file is the cheapest, highest-leverage harness you own.
- **Concrete snippet to put on the slide (short, real-looking, every line earns its place):**
  ```markdown
  # AGENTS.md
  ## Architecture (non-negotiable)
  - API routes (`src/api/**`) MUST NOT contain business logic.
    They call functions in `src/product/**`. A lint rule enforces this — do not disable it.
  - No new dependency without an ADR in `docs/adr/`.
  ## Testing
  - Tests live next to the code (`*.test.ts`). Run `npm run check` before you claim done.
  - NEVER weaken or delete an existing test to make a build pass. Fix the code.
  ## Conventions
  - Money is `Decimal`, never `number`. Dates are UTC ISO-8601.
  ```
- **Talking points:**
  - Note *why* each line exists — the "NEVER delete a test" line is there because an agent did exactly that (foreshadows Kent Beck, Slide 20).
  - The ratchet rule (Osmani): *"Every line in a good AGENTS.md should be traceable back to a specific thing that went wrong."* Don't design it — grow it from failures.
- **Evidence (optional):** overlay a real `AGENTS.md`/`CLAUDE.md` from one of your repos next to it.

### Slide 9 — Example 2: Tools — MCP
- **Key message:** MCP lets the agent *do* things (read Jira, query the DB, hit staging) instead of guessing.
- **Concrete before/after to show:**
  - **Without MCP:** "Add the field to the deals table." → agent *invents* a column name and type from context and is subtly wrong.
  - **With a Postgres/Atlassian MCP server:** the agent calls `describe_table("deals")`, sees the real schema, then calls `get_issue("PROJ-412")` to read the actual acceptance criteria — and writes against reality.
  - One-liner: **"The agent stops hallucinating your schema because it can read it, and stops guessing the requirement because it can fetch the ticket."**
- **Talking points:** MCP is the USB-C of tools — one open protocol, hundreds of prebuilt servers (Jira, Linear, GitHub, Postgres, Sentry). We return to MCP + Jira/Linear as the *entry point* of the loop in Act 6.
- **Cite:** [Model Context Protocol](https://modelcontextprotocol.io)
- **Evidence (optional):** screenshot of a real MCP tool call in your agent resolving a ticket or a table schema.

### Slide 10 — Example 3: Control flow & safety — subagents, hooks, gates
- **Key message:** You can wire *structure* around the model: a separate reviewer agent, a pre-commit hook, an approval gate on destructive actions.
- **Talking points:**
  - Anthropic's harness design: separate **planner / generator / evaluator** roles, because an agent grading its own work tends to pass it (the self-evaluation failure mode).
  - This is the seed of "review by a different, cheaper model" — Act 5.
- **Cite:** [Anthropic — *Harness design for long-running application development*](https://www.anthropic.com/engineering/harness-design-long-running-apps); [OpenAI — *Harness engineering* ("humans steer, agents execute")](https://openai.com/index/harness-engineering/)
- **Visual:** planner → generator → evaluator loop.

### Slide 11 — The mental split: guides (before) vs sensors (after)
- **Key message:** All that variety collapses into two jobs. A **guide** shapes the agent *before* it acts (feed-forward). A **sensor** grades it *after* (feedback).
- **Talking points:**
  - Guides without sensors = a wish nobody enforces. Sensors without guides = the agent re-makes the same mistake every run; it never converges. You need both.
  - This is the frame for the next act.
- **Cite:** [Birgitta Böckeler (Thoughtworks / Martin Fowler) — *Harness engineering for coding agent users*](https://martinfowler.com/articles/harness-engineering.html)
- **Visual:** timeline — `guide → [AGENT ACTS] → sensor`.

---

## Act 3 — Sensors: feed-forward & feedback (12:00 → 16:00)  → *your point #3*

### Slide 12 — Sensors are how the agent grades itself
- **Key message:** A sensor exists to give the agent feedback so it can **self-correct** — not just to fail a build.
- **Talking points:**
  - Böckeler: *"A sensor is meant to give the agent feedback so that it can self-correct."*
  - Write sensor output *for the agent* — an error message that says what to do next is worth more than a red X. "Success is silent, failures are verbose" (Osmani).
- **Cite:** [Böckeler — *Sensors for coding agents*](https://martinfowler.com/articles/sensors-for-coding-agents.html)

### Slide 13 — Two axes: *when* it fires × *how* it decides
- **Key message:** Sort every check on two axes. **Timing:** feed-forward (during coding, real-time) vs feedback (pipeline / scheduled). **Cost:** computational vs inferential.
- **Talking points:**
  - **Computational** — type checker, linter, tests, Semgrep, dependency rules: deterministic, fast, cheap, never falsely confident.
  - **Inferential** — an LLM reviewing the diff for modularity/security: semantic, slow, costs tokens, can be talked out of a real finding.
  - **The rule:** encode a check as computational *before* you reach for an inferential one. Don't pay an AI to catch what a type checker catches for free.
- **Cite:** [Böckeler — *Sensors for coding agents*](https://martinfowler.com/articles/sensors-for-coding-agents.html)
- **Visual:** 2×2 grid (feed-forward/feedback × computational/inferential) with example checks in each quadrant.

### Slide 14 — Concrete sensor ladder (cheapest first)
- **Key message:** Layer them cheapest-to-most-expensive; most failures die at the cheap layers.
- **Talking points (one line each):**
  - Type checker → lint (with AI-friendly messages) → unit/behaviour tests → architecture fitness rules (e.g. "clients must not import services") → mutation testing (are the tests even real?) → inferential review of the diff.
  - Note the trap: high coverage with weak asserts still passes bad patches. Mutation testing is the sensor *on your sensors*.
- **Cite:** [Böckeler — *Sensors for coding agents*](https://martinfowler.com/articles/sensors-for-coding-agents.html); [UTBoost — weak tests pass bad patches](https://arxiv.org/pdf/2506.09289)
- **Visual:** vertical ladder, cheap at bottom.

---

## Act 4 — Spec-Driven Development (16:00 → 21:00)  → *your point #4*

### Slide 15 — The prompt is the wrong artifact
- **Key message:** When the agent writes the code, the **spec** — not the prompt, not the code — is where human intent actually lives. So write it first and keep it.
- **Talking points:**
  - Sean Grove (OpenAI), *"The New Code"*: specs are the new source of truth; code is just the last-mile expression of them in one language.
  - A vague spec doesn't slow the agent down — it makes it *confidently wrong*, because it fills every gap with the most probable completion (tie back to Act 1).
- **Cite:** [Sean Grove / GitHub Spec Kit — SDD](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/)
- **Visual:** `prompt` crossed out → `SPEC` as the durable artifact committed next to code.

### Slide 16 — The loop
- **Key message:** **Idea → Spec → Plan → Code → Review → loop back.** And the loop-back is the whole point.
- **Talking points:**
  - Spec = *what* & acceptance criteria. Plan = *how* (ephemeral). Code = the agent's job. Review = an independent, cheaper model.
  - **The rule that changes everything:** when review fails, you do **not** hand-edit the code. You fix the **spec** or the **harness**, and re-run. Hand-patching the output is patching a symptom; the next generation will regress. (Böckeler: execution failures should send you back to the elicitation/spec, not the agent.)
- **Cite:** [GitHub Spec Kit — Spec → Plan → Tasks → Implement](https://github.com/github/spec-kit); [Hari Krishnan — *The Intent Harness*](https://intent-driven.dev/blog/2026/02/23/intent-harness/)
- **Visual:** circular loop with a bold red arrow from Review back to **Spec/Harness**, and a struck-through arrow from Review back to Code.

### Slide 17 — Why "don't touch the code" is non-negotiable
- **Key message:** The spec is the source of truth; the code is a build artifact. Editing the artifact desyncs it from the source.
- **Talking points:**
  - If a human keeps hand-fixing generated code, the spec silently becomes fiction and you lose the ability to regenerate. You're back to maintaining code by hand — you gave up the leverage.
  - This is the discipline that separates "vibe coding" from an engineering process (Karpathy coined "vibe coding" as the *casual* mode — SDD is the opposite end).
- **Visual:** spec = source, code = compiled output; arrow only flows one way.

### Slide 18 — Why "factory"? The line, not the worker, guarantees quality
- **Key message:** A factory produces consistent goods from *variable* workers by engineering the **line** around them — jigs, fixtures, QC stations. That's exactly what we've built. The agent is the worker; the spec + tests + gates are the line. **You stop being the craftsman and become the line manager.**
- **The mapping — everything so far, in one picture:**

  | Factory floor | Our workflow |
  |---|---|
  | Blueprint | The spec (Act 4) |
  | Jigs & QC stations | Guides + tests + gates — the harness (Acts 2, 3, 5) |
  | Workers (fast, capable, not individually reliable) | The model (Act 1) |
  | Line manager | You |

- **The key point (say this line):** Ford's breakthrough wasn't faster workers — it was that a standardized line with inspection yields reliable output from *ordinary, variable* workers. We can't make the model reliable (Act 1), so we make the **line** reliable. The factory *is* harness + SDD, run at scale.
- **What changes for you:** typing speed stops being the bottleneck; spec clarity, decomposition, and output evaluation become it — because a vague blueprint doesn't stay one bad unit, it replicates across every parallel build (Osmani).
- **Cite:** [Addy Osmani — *The Factory Model*](https://addyosmani.com/blog/factory-model/)
- **Visual:** an assembly line — spec → agents → QC stations → shipped, with a reject arrow looping back to the *blueprint*, one manager overseeing several parallel lines.

---

## Act 5 — TDD as the behaviour sensor (21:00 → 24:00)  → *your point #5*

### Slide 19 — Acceptance criteria → acceptance tests → red first
- **Key message:** Every requirement in the spec carries an **acceptance criterion**; the AI turns those into **acceptance tests that fail first**, and its job is to make them green.
- **Talking points:**
  - Red-first is not a nicety here — it's the *only* honest signal. Tests written *after* the code just certify "whatever happened to happen."
  - Osmani's factory model: TDD goes from best-practice to **safety system**. Write tests before implementation so the agent optimises toward *correct behaviour*, not toward *passing*.
- **Cite:** [Addy Osmani — *The Factory Model*](https://addyosmani.com/blog/factory-model/)
- **Visual:** AC-001 → test (red) → agent codes → test (green).

### Slide 20 — The genie cheats — plan for it
- **Key message:** Left alone, the agent will delete or weaken your tests to "pass." You must remove the conflict of interest.
- **Talking points:**
  - Kent Beck: the agent is an *"unpredictable genie."* His exact observation — *"The genie doesn't want to do TDD. It wants to write the code and then write tests that pass."*
  - His fix, and ours: **separate the roles.** One agent writes code; a *different, isolated* agent audits it. Separated actors can't collude — this is game theory, not politeness. (Ties directly to Slide 10's planner/generator/evaluator and Act 6's "review by a cheaper model.")
  - Guard the tests as harness: don't let the coding agent edit the acceptance tests; make weakening them a failing gate.
- **Cite:** [Kent Beck on *The Pragmatic Engineer* — TDD, AI agents and coding](https://newsletter.pragmaticengineer.com/p/tdd-ai-agents-and-coding-with-kent)
- **Visual:** two separated agents — "builder" and "auditor" — with a wall between them.

### Slide 21 — Review by a different, cheaper model
- **Key message:** The reviewer in the loop should be a *separate, cheaper* model reading the diff against the spec.
- **Talking points:**
  - Cheaper is fine — the reviewer's job is narrow (does this diff satisfy the contract?), and an independent model won't rationalise the builder's mistakes.
  - This is an *inferential* sensor — so it sits *after* the computational ladder (Act 3): tests/lints must already be green before you spend tokens on a semantic review.
- **Visual:** builder (expensive) → diff → reviewer (cheap, independent) → back to spec/harness on fail.

---

## Act 6 — Ideas as tickets: MCP + Jira/Linear (24:00 → 26:00)  → *your point #6*

### Slide 22 — Close the loop to where ideas actually live
- **Key message:** The "Idea" at the top of the loop isn't a Slack message — for most teams it's a **Jira / Linear ticket**. Connect the agent to it via **MCP**.
- **Talking points:**
  - With an MCP server for your tracker, the agent reads the ticket, drafts the spec from it, links the PR back, and updates status — the ticket becomes the entry point of the SDD loop.
  - The ticket is the *idea*; the spec (committed next to the code) is the *contract*. MCP is the wire between them.
  - **Practical note:** this needs one-time authorization of the connector (Atlassian / Linear) in your agent tool — flag it as a setup step, not a live-demo item today.
- **Cite:** [Model Context Protocol](https://modelcontextprotocol.io) (Jira via Atlassian MCP; Linear MCP)
- **Visual:** Jira/Linear ticket → (MCP) → agent → spec → PR → back to ticket.

---

## Act 7 — Land it (26:00 → 30:00)

### Slide 23 — What to do Monday (the CTA)
- **Key message:** You don't design a harness — you *grow* it. Start the loop this week on one real ticket.
- **The ask (pick the smallest real thing):**
  1. Write **one spec with acceptance criteria** for your next ticket — before any code.
  2. Turn those criteria into **failing tests** first.
  3. Add **one guide** (`AGENTS.md`) + **one computational gate** (lint/type/test in CI).
  4. Run **review with a second, cheaper model** — and when it fails, fix the **spec/harness**, not the code.
  5. Every failure you hit becomes a new line in the guide or a new sensor. That's the ratchet.
- **Visual:** a 5-item checklist, "this week."

### Slide 24 — The one thing to remember + resources
- **Key message:** *You can't control the model. You can engineer everything around it. Do that with a spec and tests.*
- **Talking points:** "Chasing the next model release is the lever everyone shares. Engineering your harness is the lever only you own."
- **Resources (leave on screen for Q&A):**
  - Addy Osmani — [Agent Harness Engineering](https://addyosmani.com/blog/agent-harness-engineering/) · [The Factory Model](https://addyosmani.com/blog/factory-model/)
  - Birgitta Böckeler / Thoughtworks — [Harness engineering](https://martinfowler.com/articles/harness-engineering.html) · [Sensors for coding agents](https://martinfowler.com/articles/sensors-for-coding-agents.html)
  - Sean Grove / GitHub — [Spec Kit](https://github.com/github/spec-kit) · [SDD launch post](https://github.blog/ai-and-ml/generative-ai/spec-driven-development-with-ai-get-started-with-a-new-open-source-toolkit/)
  - Kent Beck — [TDD, AI agents and coding](https://newsletter.pragmaticengineer.com/p/tdd-ai-agents-and-coding-with-kent)
  - Thinking Machines Lab — [Defeating Nondeterminism in LLM Inference](https://thinkingmachines.ai/blog/defeating-nondeterminism-in-llm-inference/)
  - Anthropic — [Harness design for long-running apps](https://www.anthropic.com/engineering/harness-design-long-running-apps) · OpenAI — [Harness engineering](https://openai.com/index/harness-engineering/)
  - **Our internal playbook** — the AI-Native Harness Playbook (assess → adopt → audit skills; guides & sensors chapters).

---

## Appendix A — Timing cheat-sheet

| Act | Slides | Window | Minutes |
|---|---|---|---|
| 0 · Hook | 1–2 | 0:00–2:00 | 2 |
| 1 · Why output is random | 3–5 | 2:00–6:00 | 4 |
| 2 · Agent = Model + Harness | 6–11 | 6:00–12:00 | 6 |
| 3 · Sensors | 12–14 | 12:00–16:00 | 4 |
| 4 · Spec-Driven Development | 15–18 | 16:00–21:00 | 5 |
| 5 · TDD as behaviour sensor | 19–21 | 21:00–24:00 | 3 |
| 6 · MCP + Jira/Linear | 22 | 24:00–26:00 | 2 |
| 7 · Land it | 23–24 | 26:00–30:00 | 4 (incl. buffer) |
| **Q&A** | — | after | ~5 |

> If you run long, the cuttable slides are **9, 14, 17** (each reinforces a point made elsewhere). Never cut **5, 6, 11, 16, 20** — they carry the argument.

## Appendix B — Voice → slide map (so every claim has an owner)

| Voice | Their idea | Slide(s) |
|---|---|---|
| Thinking Machines Lab | Non-determinism even at temp 0 (batch-invariance); Feynman 1000-run example | 4 |
| Addy Osmani (+ Viv Trivedy) | Agent = model + harness; taxonomy; ratchet; factory model; TDD as safety system | 6, 7, 8, 18, 19 |
| Birgitta Böckeler / Thoughtworks | Guides vs sensors; feed-forward vs feedback; computational-first; self-correction | 11, 12, 13, 14, 16 |
| Sean Grove / GitHub Spec Kit | Specs are the new code; Spec→Plan→Tasks→Implement | 15, 16 |
| Kent Beck | "Unpredictable genie"; genie won't do TDD; separate builder/auditor | 20 |
| Anthropic / OpenAI | Planner/generator/evaluator; self-eval failure; humans steer, agents execute | 10, 21 |
| Hari Krishnan | Failures send you back to the spec, not the agent | 16 |
| UTBoost paper | Weak tests pass bad patches → mutation testing | 14 |

## Appendix C — Anticipated Q&A

- **"Isn't writing specs + tests slower than just prompting?"** — Slower to first diff, faster to *correct, shippable* diff, and far faster across a feature's life because you can regenerate. Vagueness isn't speed; it's deferred rework.
- **"Which model for the reviewer?"** — A cheaper, *different* model than the builder. Independence matters more than raw capability for a narrow spec-conformance check.
- **"Do we have to adopt all of this at once?"** — No. Grow it. One spec, one failing test, one gate this week (Slide 23). The harness compounds from real failures.
- **"What about legacy code with no tests?"** — Harnessability is lower there; earn a foothold on one high-risk seam first. (Point to the playbook's brownfield chapter.)
