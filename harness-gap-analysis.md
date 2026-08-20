# Gap Analysis — Playbook vs. Its Own References

A comparison of the playbook's coverage against the core content of the sources listed in
[references.md](playbook-md/references.md), fetched and re-read 2026-08-20. The playbook covers the
foundations well — guides/sensors, computational-before-inferential, separate evaluator, mutation
testing as the coverage-illusion antidote, spec-as-contract, trunk-based delivery. The gaps below are
where the references go further than the playbook currently does.

## Gaps

### 1. No graduated autonomy ladder

The OpenAI harness-engineering piece is built around graduated autonomy — agents earn wider scope
level by level, with explicit criteria per level. The playbook has the zone-based version (flop
zones, "widen autonomy as the harness earns it") but "levels of autonomy" exists only as a glossary
entry. There is no chapter defining the levels, what evidence promotes a zone from one level to the
next, or what demotes it.

**Suggestion:** add a short chapter (or a section in `50-adoption/02`) defining 3–4 named autonomy
levels per zone (e.g. propose-only → merge-with-review → merge-on-green → self-merge), each with the
sensor coverage required to enter it and the regression signal that drops it back.

### 2. Evaluator calibration is missing

The Anthropic harness-design article treats the evaluator as something you *tune*: few-shot scoring
examples with detailed breakdowns to reduce score drift, a read-logs → find-divergence → update-prompt
loop, and a warning that criteria wording steers behavior beyond its literal intent ("museum quality"
caused visual convergence). It also documents that an untuned QA agent finds real issues and then
talks itself into approving anyway. The playbook prescribes "separate skeptical evaluator" but says
nothing about calibrating it.

**Suggestion:** extend `10-lifecycle/05` (or `20-harness/03`) with an "evaluator calibration"
practice: anchor scores with worked examples, tune against logged divergences, and stress-test
criteria wording. Include the counter-warning from Claude best practices: a reviewer prompted to find
gaps will report some even when the work is sound — chasing every finding over-engineers.

### 3. Spec/code drift has no mechanism

The InfoQ SDD article makes drift detection concrete: spec-differential checks in CI, changes
auto-classified as additive / compatible / breaking / ambiguous, and an explicit compatibility policy
gating breaking changes behind human sign-off. The playbook prescribes "regular audit for spec/code
drift" and an audit spec, but gives no CI-wired mechanism and no change-classification discipline.

**Suggestion:** add to `30-delivery/04` (drift sensors) a spec-drift sensor pattern: a scheduled or
CI-triggered comparison of spec REQ/AC ids against tests and implementation, plus a rule that spec
changes get classified for compatibility before merge.

### 4. No SDD spectrum — where the playbook's spec model sits

Böckeler's taxonomy (spec-first / spec-anchored / spec-as-source) is the reference frame for the
whole SDD conversation, and the playbook never names it. The playbook's durable, audited spec is
effectively *spec-anchored*, but a reader can't locate it on the spectrum or understand why the
playbook stops short of spec-as-source (Böckeler's own MDD parallel: inflexibility *plus*
non-determinism).

**Suggestion:** one section in `10-lifecycle/01` naming the three levels, placing the playbook at
spec-anchored, and giving the skeptical case against spec-as-source. Also adopt her granularity
warning: no tool adapts workflow to problem size — a bug fix must not become 4 user stories with 16
acceptance criteria. The playbook has this anti-pattern for plans but not for specs.

### 5. Intent elicitation before the spec

The Intent Harness article and the Claude best-practices interview pattern both put a structured
clarification step *before* spec writing: challenge assumptions, make constraints explicit, define
success criteria, decide what not to build — and when execution fails, trace the root cause back to
elicitation, not just the agent ("what in our process allowed this ambiguity to survive?"). The
playbook's lifecycle starts at the spec; "draft it with the agent" is the only elicitation guidance.

**Suggestion:** add an interview-then-spec practice to `10-lifecycle/01`: the agent interviews the
human until ambiguities are resolved, writes the spec, then execution starts in a *fresh session*
(clean context — ties to the existing reset-vs-compaction guidance). Add the cascading root-cause
frame to `40-anti-patterns/01`.

### 6. Sensor ergonomics — feedback designed for agent consumption

The sensors article's most actionable material is missing: custom lint messages that carry
self-correction guidance and a suppress-with-reason escape hatch; letting the agent raise a threshold
with justification instead of a binary suppress-or-comply; query tools that let an agent interrogate a
large sensor report (the Stryker JSON query-tool pattern) without flooding its context; and
sensor-effectiveness tracking over time (a sensor that never fires is a prune candidate — which the
playbook asks as an open question but the article answers with logging and trend charts).

**Suggestion:** extend `20-harness/03` beyond "the failure message is feedforward" with these four
mechanics. The query-tool pattern also belongs in `00-foundations/02` as a context-budget technique.

### 7. Mutation testing lacks operational detail

`20-harness/05` has the concept; the sensors article has the practice: the survivors metric as the
headline number, a concrete case where 100% statement coverage hid 13 survivors and zero unit tests,
incremental mutation testing at pre-commit vs. full runs post-integration, and mutation-score
*trends* as a scheduled sensor. UTBoost supplies the hard numbers the playbook alludes to without
citing: 7.7% of SWE-bench Lite and 5.2% of Verified had insufficient tests, and 28.4% / 15.7% of
"passing" patches failed once tests were strengthened — despite Verified being reviewed by 93
engineers.

**Suggestion:** add cadence guidance (incremental pre-commit, full in CI, trend on schedule) to
`20-harness/05` and `30-delivery/04`, and wire the UTBoost figures into `20-harness/03`'s currently
unsourced "benchmark suites let provably wrong fixes pass" claim.

### 8. Property-based testing is one paragraph; the research gives it a loop

The Anthropic PBT work shows a reproducible agent loop: comprehend target → propose properties from
spec/type/docstring context → generate hypothesis tests → execute with self-reflection (is this a real
bug or a test artifact?) → report only above a confidence threshold. Ranking findings with a scoring
rubric lifted valid-bug precision from 56% to 86%. That reflection-and-ranking discipline is exactly
the playbook's "proof, not vibes" applied to test generation, and it's absent.

**Suggestion:** expand the PBT section of `20-harness/05` with this loop, especially the reflection
step and the rubric-ranking idea for triaging agent-generated findings.

### 9. Skills: loading and retrieval are failure points, not givens

The skills-in-the-wild paper quantifies what the playbook's "skill-library tax" only gestures at:
agents load all relevant curated skills in only 49% of trajectories (31% with distractors present),
and for weaker models noisy retrieved skills score *below* the no-skills baseline — irrelevant skills
actively mislead. Refinement helps only above a coverage-quality threshold ("a multiplier on existing
skill quality, not a generator of new knowledge").

**Suggestion:** in `20-harness/01`, make "did the agent actually load and apply the skill" an
explicit check rather than an assumption, and add curation guidance: fewer, higher-coverage skills
beat a large noisy library — now with numbers to back it.

### 10. Deterministic enforcement vs. advisory guides

Claude best practices draws a line the playbook blurs: instruction files are advisory ("bloated
CLAUDE.md files cause Claude to ignore your actual instructions"), hooks are deterministic ("for
actions that must happen every time with zero exceptions"), with an escalation ladder from in-prompt
check → session goal → blocking stop-hook → independent second opinion. The playbook's anti-cheat
constraints live entirely at the advisory level.

**Suggestion:** add the advisory-vs-deterministic distinction and the escalation ladder to
`20-harness/01`/`04`, plus the CLAUDE.md litmus test: "would removing this line cause the agent to
make mistakes?"

## Smaller notes

- **Context engineering** (`00-foundations/02`) could absorb three named techniques from the
  Anthropic context article: progressive disclosure, just-in-time retrieval over up-front dumping,
  and structured note-taking as agentic memory. The subagent-as-context-firewall point exists but the
  "condensed 1,000–2,000-token summary back to the lead" framing sharpens it.
- **Merge philosophy tension**: OpenAI runs minimal blocking gates with post-merge correction
  tolerance; the playbook is strict-gate. Worth one paragraph acknowledging this as a maturity
  spectrum, not a contradiction — loose gates presuppose the drift-cleanup agents OpenAI also runs.
- **References hygiene**: the OpenAI harness-engineering page now 403s direct fetches; content was
  verified only via secondary sources. Worth noting in `references.md` or archiving key quotes.

## Priority

If ordering by leverage: **2 (evaluator calibration)**, **6 (sensor ergonomics)**, and **7 (mutation
mechanics)** strengthen chapters that already exist and are cheap to land. **1 (autonomy ladder)** and
**5 (intent elicitation)** are the two genuinely missing lifecycle pieces. The rest are enrichment.
