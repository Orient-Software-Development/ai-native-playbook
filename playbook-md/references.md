# References

The research and industry writing behind this playbook. The practice chapters teach method, not
citations — so they name no sources in-page. This is where the evidence lives: the canonical vendor
guides, the framework article the playbook is built on, and the papers and research notes that back
specific claims about context, skills, and testing.

Grouped by what they support. Every claim in the playbook that leans on outside work traces to
something here.

Breadcrumb: [Playbook](README.md)

---

## Foundational — harness engineering

The three articles that define the discipline and shape the playbook's spine.

### Anthropic — *Harness design for long-running application development*

The planner / generator / evaluator separation, context resets vs. compaction, and the
self-evaluation failure mode (why an agent grading its own work tends to pass it). The reference for
the harness as a steering loop around a long-running agent.

→ https://www.anthropic.com/engineering/harness-design-long-running-apps

### OpenAI — *Harness engineering: leveraging Codex in an agent-first world*

"Humans steer, agents execute." The repository as the system of record, legibility as a first-class
property, graduated autonomy levels (earn trust before widening scope), and throughput benchmarks for
what an agent-first workflow changes about how many small PRs a day ship.

→ https://openai.com/index/harness-engineering/

### Birgitta Böckeler (Martin Fowler) — *Harness engineering for coding agent users*

The framework source. Where the term "harness engineering" is set out for the engineer using a coding
agent — guides in front, sensors behind — generalized in this playbook away from any one tool or repo.

→ https://martinfowler.com/articles/harness-engineering.html

### Birgitta Böckeler (Martin Fowler) — *Sensors for coding agents*

The companion piece on the "behind" half of the harness: the feedback signals — tests, linters,
type checks, build and runtime errors — that grade an agent's work after it acts, and how to wire them
so the agent reads its own grade and self-corrects.

→ https://martinfowler.com/articles/sensors-for-coding-agents.html

---

## Vendor best-practice guides

Two canonical, product-specific guides folded into the method tool-agnostically: take the practice,
drop the product name.

### Anthropic — *Claude Code best practices*

→ https://code.claude.com/docs/en/best-practices

### OpenAI — *Codex best practices*

→ https://developers.openai.com/codex/learn/best-practices

---

## Context engineering

Why the context window is a budget, not a backpack — the research behind the context-engineering
chapter.

### Anthropic — *Effective context engineering for AI agents*

Attention budget, the "right altitude" for instructions, and compaction vs. context-reset strategy.

→ https://www.anthropic.com/engineering/effective-context-engineering-for-ai-agents

### Chroma Research — *Context Rot*

How model performance degrades as the input window grows — the empirical case for keeping context
tight rather than dumping everything in.

→ https://www.trychroma.com/research/context-rot

---

## Skills

The evidence behind treating reusable agent skills as a first-class guide.

### Anthropic — *Equipping agents for the real world with Agent Skills*

→ https://www.anthropic.com/engineering/equipping-agents-for-the-real-world-with-agent-skills

### *How Well Do Agentic Skills Work in the Wild?*

→ https://arxiv.org/pdf/2604.04323v1

---

## Spec-driven development

Writing the specification before the code, and treating it as a first-class artifact the agent works
from — the spectrum from spec-first to spec-as-source, and where it overlaps the harness.

### Leigh Griffin & Ray Carroll (InfoQ) — *Spec Driven Development: When Architecture Becomes Executable*

A five-layer model — specification, generation, artifact, validation, runtime — in which the
executable spec, not the code, is the source of truth. Makes the case for "architectural determinism"
with continuous drift detection between spec and implementation.

→ https://www.infoq.com/articles/spec-driven-development/

### Birgitta Böckeler (Martin Fowler) — *Understanding Spec-Driven Development: Kiro, spec-kit, and Tessl*

The taxonomy: three levels of SDD — spec-first, spec-anchored, and spec-as-source — surveyed across
three tools. A skeptical read on whether elaborate spec workflows improve development or just amplify
review burden and hallucination.

→ https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html

### Hari Krishnan — *The Intent Harness*

A disciplined process for turning rough intuition into executable agent instructions through
structured clarification and human–AI dialogue. Argues that execution failures should send you back to
the elicitation, specification, and review process — not just the agent.

→ https://intent-driven.dev/blog/2026/02/23/intent-harness/

---

## Verification & testing

The research behind "proof, not vibes" — property-based testing, mutation testing, and why weak tests
are a liability.

### Anthropic Red — *Property-Based Testing with Claude*

Generating spec-derived assertions and invariants instead of hand-picked examples.

→ https://red.anthropic.com/2026/property-based-testing/

### *Agentic Property-Based Testing*

→ https://arxiv.org/pdf/2510.09907

### *UTBoost* — weak tests pass bad patches on SWE-bench

Evidence that under-powered test suites let incorrect agent patches through — the case for grading the
tests themselves (mutation testing) rather than trusting a green run.

→ https://arxiv.org/pdf/2506.09289
