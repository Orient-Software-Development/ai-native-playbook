# Graphic Plan — Illustrations for the Foundations & Harness Chapters

> **Status:** Proposal — not yet built. Scopes a set of diagrams for `00-foundations/03-harness-engineering.md` and the five `20-harness/` pages, modelled on (but not copied from) the figures in [`../references/Harness engineering for coding agent users.md`](../references/Harness%20engineering%20for%20coding%20agent%20users.md).
> **Owner:** Innovation Hub, Orient Software
> **Relates to:** `PLAN.md` §6 Q4 (“worth adding the steering-loop / keep-quality-left diagrams as assets, or keep it text-only for v1?”) — this is the concrete answer to that open question.
> **Drafted:** 2026-06-06

This file is a *plan for graphics*, the way `PLAN.md` is a plan for prose. Nothing here ships
to readers as-is. It proposes **what** each graphic shows, **where** it anchors, **why** it
beats the paragraph it sits next to, and **how** it stays a cousin of the reference figures
rather than a clone of them. Once approved, we build the P0 set first (§6) and wire them into
the pages.

---

## 1. Why graphics, and why these

The reference article carries six figures and they do real work: the concentric-circles figure
makes "harness" *mean something specific* in one glance, and the overview figure makes the
guides→agent→sensors loop legible before you've read a word of body text. Our Foundations and
Harness chapters argue the *same family of ideas* — feedforward vs feedback, computational vs
inferential, the steering loop, the layered gate — but currently carry them in prose and one
ASCII block apiece. Each of those ideas has a natural picture, and the picture lands faster than
the paragraph.

The goal is **10× more interesting, not 10× more pictures.** A small set of strong, reusable
diagrams that share one visual language will do more than a sprawl of one-off sketches. Every
graphic below has to clear three bars:

1. **It conveys the meaning faster than the prose can.** If the paragraph is already instant, no
   graphic — decoration is a budget tax on the reader, the same way a junk guide is a tax on the
   agent.
2. **It is a *cousin* of a reference figure, never a copy.** We borrow the aesthetic (warm,
   hand-drawn, annotated, human-in-the-loop) and reuse motifs, but each graphic shows *our*
   page's specific point with its own composition.
3. **It obeys the playbook's own rules.** Tool-agnostic (no product names in the artwork —
   §1.1), no reader-facing numbers or metrics (§4), and junior-readable: a one-year engineer
   should *get* the diagram without the caption (§2.4).

---

## 2. The reference figures, for kinship reference

So "similar, not the same" has a baseline. The six figures we're riffing on:

| # | Reference figure | Core device |
|---|---|---|
| R1 | Bounded contexts | Three concentric circles: model core → builder harness → user harness |
| R2 | Harness overview | Guides feedforward → agent → sensors feedback loop; human steers both; computational/inferential split |
| R3 | Change lifecycle | Feedforward + feedback controls distributed left-to-right across a change's life |
| R4 | Harness types | Grid: guides/sensors (horizontal) × maintainability/architecture/behaviour (vertical) |
| R5 | Harness templates | A stack of service topologies, each a bundle of guides + sensors |

Each graphic below names the reference figure it's kin to and states how it diverges.

---

## 3. The shared visual language (build this first)

Before any single diagram, agree a tiny **icon + colour kit** and reuse it everywhere — exactly
how the article reuses one human icon and one guide/sensor colouring across all six figures. A
shared vocabulary is what makes six diagrams feel like one playbook instead of six clip-art
grabs. Proposed kit:

| Element | Motif | Colour |
|---|---|---|
| **The model** | a small chip / brain glyph | neutral grey (you don't own it) |
| **The agent** | the model chip *inside* a rounded frame (model + harness) | grey core, coloured frame |
| **Guide (feedforward)** | a chevron/arrow pointing *into* the act `▶` | warm (amber) |
| **Sensor (feedback)** | an eye / gauge pointing *back* at the output `◉` | cool (teal) |
| **Computational check** | solid outline, square corners | saturated |
| **Inferential check** | dashed outline, soft corners | faded (signals non-determinism) |
| **The human** | a small standing figure at the side, hand on a dial | accent colour |
| **The act / the diff** | a code-block glyph in the centre | dark |
| **Context window** | a bracket/frame around what the agent can see | thin outline |
| **Pass / fail** | green check ✓ / red wall ✗ | green / red |

Two cues do a lot of silent teaching and should be *consistent* across every graphic: **amber =
before / teal = after** (so feedforward and feedback are colour-coded the moment a reader sees
them), and **dashed = inferential** (so "this one is non-deterministic and costs tokens" reads
without a label). Ship this kit as `_assets/_kit.svg` (or an Excalidraw library) and draw
everything from it.

---

## 4. The catalog

Grouped by page, in reading order. Each entry: the anchor section, the concept, an ASCII mock of
the composition, why it lands, its kinship to a reference figure, and a suggested render format.

### 00-foundations/03 — Harness Engineering (the conceptual spine; carries the most graphics)

#### G1 · The lever — same model, two harnesses ⭐P0
- **Anchors:** "The harness is the lever you actually control."
- **Concept:** identical model, two wrappings, diverging output quality — the page's central
  claim made visual.
- **Composition:**
  ```
                ┌─ raw prompt ─────────────▶  ⌁ messy output
     [model]──┤
                └─ ▶guides + ◉sensors ─────▶  ▣ clean output
       (same chip on both branches; the fork is the whole point)
  ```
  Two branches off one grey model chip. Top branch: bare, output a jagged/scribbled block.
  Bottom branch: the chip wrapped in an amber guide chevron and a teal sensor eye, output a tidy
  block. The model glyph is *visibly identical* on both — that's the argument.
- **Why it lands:** the page's thesis ("the same model in a better harness produces meaningfully
  better work") is a comparison, and comparisons are what pictures do best. No numbers needed —
  the divergence carries it.
- **Kin:** none directly — this is our own. Loosely echoes R1's "the model is the core you wrap."
- **Render:** hand-drawn SVG (Excalidraw). Metaphor-heavy → not Mermaid.

#### G2 · Before & after the act ⭐P0
- **Anchors:** "Guides and sensors — before and after."
- **Concept:** a guide acts *before* the agent, a sensor *after* — placed on a left-to-right time
  arrow with "the act" in the middle.
- **Composition:**
  ```
     ▶ GUIDE            ░ THE ACT ░            ◉ SENSOR
   (feedforward)   →   agent writes code   →   (feedback)
   amber, before                               teal, after
   ───────────────────── time ──────────────────────▶
  ```
- **Why it lands:** "feedforward / feedback" is jargon until you *see* it pinned to a timeline.
  This single graphic teaches the two terms the whole rest of the chapters lean on.
- **Kin:** R2 (the overview loop) — but ours is stripped to a *timeline*, not a loop, so the
  before/after split is the only idea on screen. R2 shows the whole system; G2 shows the one
  distinction.
- **Render:** Mermaid (simple flow) *or* SVG. Lean SVG to colour-code amber/teal.

#### G3 · You need both — the gap where bugs live ⭐P1
- **Anchors:** "The trap is thinking you can get away with one."
- **Concept:** three panels — *guide only* (rule stated, never enforced → leaks), *sensor only*
  (caught every time, never learns → never converges), *both* (the gap closes).
- **Composition:**
  ```
   GUIDE ONLY        SENSOR ONLY         BOTH
   ▶……… ✗ leaks      ◉ catches, again    ▶……◉
   "a wish"          "never converges"    gap closed ✓
  ```
  Reuse the same "API routes must not contain business logic" rule the page already walks, so the
  graphic and the worked example reinforce each other.
- **Why it lands:** turns the page's three-paragraph walkthrough into one scannable triptych; the
  empty space between guide and sensor in the middle panel literally *is* "bugs live in the gap."
- **Kin:** none in the article — the article asserts the loop but never draws the failure of
  half-a-loop. This is value the reference doesn't have.
- **Render:** SVG triptych.

#### G4 · Reach for the cheap check first ⭐P1
- **Anchors:** "Reach for the cheap check first" + the computational/inferential table.
- **Concept:** a cost/speed gauge — computational on the cheap-fast-certain end, inferential on
  the slow-pricey-fuzzy end — with the rule "encode it computationally before you reach for
  inference" as a one-way arrow.
- **Composition:**
  ```
   cheap · fast · certain ───────────────▶ costly · slow · fuzzy
   ▢ types  ▢ lint  ▢ tests   ┆   ⌐ AI review  ⌐ spec-to-code
   COMPUTATIONAL (solid)      ┆   INFERENTIAL (dashed)
            └─ try here first ─┘
  ```
- **Why it lands:** establishes the *dashed = inferential* convention the kit relies on, and the
  spectrum makes "computational-first" feel like gravity rather than a rule to memorise.
- **Kin:** R2 encodes computational/inferential as a legend; G4 makes it a *spectrum with a
  default direction* — a different, more prescriptive device.
- **Render:** SVG (the dashed/solid distinction is the point).

#### G5 · Three targets, hardest last ⭐P1
- **Anchors:** "Three things you're regulating — hardest last."
- **Concept:** an ascending staircase — maintainability → architecture fitness → behaviour —
  with "how reliably a sensor can catch it" *falling* as the steps rise.
- **Composition:**
  ```
                                    ┌─ BEHAVIOUR ─┐  ← human stays
                      ┌─ ARCHITECTURE ┘  (hard)
        ┌─ MAINTAINABILITY ┘ (cheap, computational)
        rising difficulty  ▲   falling sensor reliability ▼
  ```
- **Why it lands:** "hardest last" is an *ordering* claim; a staircase is the canonical picture of
  an ordering. The human figure parked on the top step previews why behaviour keeps a human in the
  loop.
- **Kin:** R4 (the regulation grid). Ours drops the guides/sensors axis and reshapes the three
  categories as a *difficulty climb* — same three concepts, opposite emphasis (R4 = taxonomy, G5 =
  gradient).
- **Render:** SVG.

#### G6 · Harnessability — the ground you build on, pays off twice ⭐P1
- **Anchors:** "Some codebases take the harness better than others."
- **Concept:** a codebase drawn as terrain; four pillars (strong types · clear boundaries ·
  established frameworks · high coverage) raise the ground level; one arrow splits into **two**
  payoffs — "easier for humans" and "easier for agents."
- **Composition:**
  ```
        easier for HUMANS  ◀──┐
                              ├── one investment
        easier for AGENTS  ◀──┘
     ▟▙ types  ▟▙ boundaries  ▟▙ frameworks  ▟▙ coverage
     ════════════ raised, harnessable ground ════════════
     (vs. cracked, sunken ground = sprawling/untyped repo)
  ```
- **Why it lands:** "pays off twice" is the page's most memorable phrase and is begging for the
  one-arrow-two-outputs picture. The terrain metaphor also makes the warning ("the harness is
  hardest to build exactly where it's most needed") visual — show the cracked low ground beside the
  raised ground.
- **Kin:** none — the article states harnessability in prose only. Net-new.
- **Render:** SVG (metaphor).

#### G7 · The steering loop ⭐P0
- **Anchors:** "You don't design the harness — you grow it" (replaces/upgrades the existing ASCII
  block).
- **Concept:** the cybernetic governor loop, drawn as an actual cycle: *agent acts → a sensor
  fires → human looks → adds a guide or sensor → agent acts again*, with the human on the dial.
- **Composition:**
  ```
        ┌──────────────▶ agent acts ──────────────┐
        │                                          ▼
   add guide/sensor ◀── human looks ◀── a sensor fires
        ▲   (the harness compounds, one failure at a time)
        └─ 🧍 human steers the dial
  ```
- **Why it lands:** the page already ships this as a flat ASCII line; a *circle* shows the
  compounding ("each turn makes the next run more reliable") that a straight arrow can't. This is
  the chapter's climactic idea — it earns the upgrade.
- **Kin:** R2 shows a self-correcting loop with a human on the side; G7 zooms into *just* that loop
  and labels each station, making the steering action explicit. Different altitude, same DNA.
- **Render:** Mermaid `flowchart` in a cycle (structurally simple, renders in GitHub natively,
  cheap to keep current) — or SVG if we want the hand-drawn warmth here too.

---

### 20-harness/01 — Guides (Feedforward)

#### G8 · The guide budget — every guide spends ⭐P1
- **Anchors:** "every guide reaches the agent through the same channel — so every guide *spends
  budget*."
- **Concept:** a fixed-width context bar being consumed by guides; a lean set leaves room for *the
  task*, a bloated library crowds the task out.
- **Composition:**
  ```
   LEAN     [▶instr][▶blueprint][▶boundary][░░░ task room ░░░]  ✓
   BLOATED  [▶▶▶▶▶ hundreds of skills ▶▶▶▶▶][task]✗ no room
            └──────── one context window ────────┘
  ```
- **Why it lands:** makes "more guides is a tax, not a freebie" physical — the task getting
  squeezed off the right edge is the whole anti-pattern in one frame. No token numbers needed
  (§4-safe).
- **Kin:** none in the article. Ties visually to the (planned) context-engineering chapter's
  attention-budget idea — reuse the same "bar" motif there for cross-chapter cohesion.
- **Render:** SVG bar (or even a styled HTML/CSS bar if we go the HTML-render route per `PLAN.md`
  §6 Q2).

#### G9 · Blueprint beats a rule ⭐P2
- **Anchors:** "Prefer a blueprint over a rule."
- **Concept:** left — three dense paragraphs of prose the agent must interpret (and can misread);
  right — one arrow pointing at a single canonical example. Same outcome, far less to get wrong.
- **Composition:** `[¶¶¶ "rules…" → 🤔 guess]   vs   [▶ "looks like THIS one" → ▣ copy]`
- **Why it lands:** quietly argues the page's "point, don't describe" rule by showing the
  interpretation gap the prose version leaves open.
- **Kin:** echoes R5's "one canonical instance" idea (a template you instantiate) at the level of a
  single pattern. Different scale.
- **Render:** SVG.

#### G10 · The fork — cheapest path to green ⭐P0 (anti-cheat; reused across pages)
- **Anchors:** "Anti-cheat constraints" / the In-practice cheat ("softening the assertion").
- **Concept:** a failing test puts the agent at a fork — the *cheap* path (weaken the test / patch
  to satisfy → fake green) vs the *real* path (fix the code → true green). A guide-constraint plus a
  state-asserting sensor close off the cheap path.
- **Composition:**
  ```
                       ┌─▶ weaken the test ─▶ ✗ fake green  (← blocked by ▶rule + ◉state sensor)
   red test ─ agent ──┤
                       └─▶ fix the code ────▶ ✓ true green
  ```
- **Why it lands:** reward hacking is the headline agent-era failure across three pages (guides 01,
  sensors 03, anti-patterns 01). One shared "fork" graphic, reused in all three, becomes the
  playbook's signature image for it — and showing the cheap path *barred* is more memorable than any
  paragraph.
- **Kin:** none — net-new, and arguably the most valuable graphic in the set because the concept
  recurs.
- **Render:** SVG. Build once, reuse on 01 / 03 / 40-anti-patterns.

---

### 20-harness/02 — Repo Structure and Legibility

#### G11 · Map vs maze ⭐P0
- **Anchors:** "A **legible** repo … is a map. An illegible one is a maze."
- **Concept:** literally that — the same repo rendered twice. Left: a clean labelled map (`specs/`,
  tests beside code, a boundary arrow) the agent walks straight through. Right: a tangled maze the
  agent gropes through file-by-file, burning budget on navigation.
- **Composition:**
  ```
   LEGIBLE (map)              ILLEGIBLE (maze)
   specs/  ▶ findable         ┌┐┌─┐ ?  ┌┐
   mod/ + test beside it      │ grep… ┘ │ wrong file
   boundary → arrow shown     └─┐ ? ┌───┘ thrash
   agent walks it ✓           agent gropes ✗ (budget burned)
  ```
- **Why it lands:** the page *names* the map/maze metaphor in its thesis — drawing it is the
  obvious payoff, and the contrast format (already proven by every In-practice "without/with") is
  the page's native rhythm.
- **Kin:** none in the article. Net-new.
- **Render:** SVG (illustrative).

#### G12 · Buried boundary vs readable map ⭐P1
- **Anchors:** "Make boundaries a readable map, not just a buried config" + the In-practice
  monorepo example.
- **Concept:** the *same* boundary rule shown two ways — buried in a 300-line lint config (a sensor:
  fails the build *after* a violation) vs a short human-readable package map (a guide: tells the
  agent the shape *before* it acts). The point: you want both, and the map is the missing half.
- **Composition:**
  ```
   CONFIG (sensor, after)        MAP (guide, before)
   ▤ 300 lines, unread     +     finance ─▶ shared
   ◉ fails build on import        reporting ─▶ shared
   agent learns by colliding      reporting ✗▶ finance
                                  agent reads it first ✓
  ```
- **Why it lands:** crisply separates *enforced* from *legible* — the page's subtlest point, and
  the one teams get wrong most. The amber-guide / teal-sensor colour coding from the kit does the
  teaching for free.
- **Kin:** none. Reinforces G2's before/after split applied to one concrete rule.
- **Render:** SVG.

---

### 20-harness/03 — Sensors (Feedback)

#### G13 · The layered gate (sensor pyramid) ⭐P0
- **Anchors:** "it's a layered gate, not one fat suite" + the pyramid paragraph.
- **Concept:** the test pyramid, but *labelled by what each tier catches that the cheaper one
  can't*, and tagged by cadence (broad cheap base runs constantly, narrow tip runs rarely).
- **Composition:**
  ```
              ╱  e2e  ╲          rare · slow · whole flow
            ╱ perf/load ╲
          ╱ integration   ╲      real state, not mocks
        ╱   unit tests       ╲
      ╱ types · lint · fitness ╲  constant · free · certain
      ◉ each tier sees what the one below cannot
  ```
- **Why it lands:** "layered gate" is the page's organising image and there's no picture of it yet.
  Annotating *what each tier uniquely catches* turns a generic pyramid into the page's actual
  argument.
- **Kin:** the test pyramid is a known shape, not from the article — so this is a fresh device for
  the playbook. Pairs with G14 directly below it.
- **Render:** SVG (or Mermaid if we accept a blockier pyramid).

#### G14 · A sensor you can pass without doing the work ⭐P0
- **Anchors:** "Sensor integrity — the rule underneath all of it."
- **Concept:** two tests on the same command. Left: the vacuous one — a hollow ✓ that never looks
  at the result ("didn't throw"). Right: the honest one — an eye that reaches into the database and
  reads the real end-state back.
- **Composition:**
  ```
   VACUOUS                       STATE-ASSERTING
   call() → "no error" ✓         call() → ◉ read row back
   (green whether or not          assert: row exists,
    the write happened)            fields set, right tenant ✓
   ✗ reports safety it can't       green ⇔ the work was done
      back up
  ```
- **Why it lands:** "a check the agent can pass without doing the work is not a sensor" is the
  single most important rule in the harness chapters. The hollow-check-vs-eye-reaching-into-the-DB
  contrast makes it unforgettable.
- **Kin:** none — net-new, and a flagship graphic. Reuses the G10 fork's "fake green vs true green"
  language for cohesion.
- **Render:** SVG.

---

### 20-harness/04 — Keep Quality Left

#### G15 · The placement line — far left as it can afford ⭐P0
- **Anchors:** the `pre-commit → … → production` ASCII block + "as far left as it *can afford*, not
  all the way left."
- **Concept:** the pipeline as a line, with each check dropped onto the *earliest stage whose
  cadence its cost doesn't break* — fast cheap checks left, slow expensive ones in CI. Cadence
  ("runs constantly → runs rarely") drawn as a fading frequency comb.
- **Composition:**
  ```
   pre-commit ─▶ pre-push ─▶ CI ─────▶ staging ─▶ prod
   format,lint   affected   build,e2e,
   types,fast    tests,     perf,infer.review
   unit          migrations
   ‖‖‖‖‖‖‖‖‖ (constant) ……………… │ (rare) …………
  ```
- **Why it lands:** upgrades the page's flat ASCII into something that shows *placement by cost* —
  the actual rule — and sets up its companion failure graphic (G16).
- **Kin:** R3 (the change-lifecycle figure) is the direct ancestor. Ours is **deliberately
  narrower**: R3 catalogues many controls across the lifecycle; G15 shows *one placement rule*
  (match cost to cadence) on a bare line. Same skeleton, our specific lesson.
- **Render:** Mermaid (linear flow, renders in GitHub) — or SVG for the fading-comb cadence cue.

#### G16 · The overloaded hook & the escape hatch ⭐P1
- **Anchors:** "Overload the left … you get a hook the whole team learns to skip."
- **Concept:** the failure mode — a pre-commit hook stuffed with the full suite, a clock ticking
  minutes, and the `--no-verify` escape hatch everyone starts taking → the gate now fires on *no*
  commits.
- **Composition:**
  ```
   pre-commit: [full build + all unit + e2e]  ⏱ minutes/commit
        │                              │
        │  everyone reaches for ─▶  ⤵ --no-verify
        ▼
   gate fires on 0 commits  ✗  (trusted, but bypassed)
  ```
- **Why it lands:** the counterintuitive core of the page is that *more checks earlier = less
  safety*. Showing the escape hatch as the thing the overload *creates* makes the paradox click.
- **Kin:** none — the article doesn't draw this failure. Net-new, pairs with G15 as before/after.
- **Render:** SVG.

---

### 20-harness/05 — Behaviour Harness

#### G17 · Mirror vs invariant ⭐P0
- **Anchors:** "Derive it from the code and it inherits the code's bugs; derive it from the
  invariant and it stands *outside* the code."
- **Concept:** left — a test reverse-engineered from the code is a *mirror*: it reflects the
  implementation, bug and all, so it can never disagree. Right — a test derived from the invariant
  stands *outside* the code and can point back at it and say "wrong."
- **Composition:**
  ```
   MIRROR (from code)            INVARIANT (from intent)
   code ⟷ test  (reflection)     [rule] ─▶ test ─▶ judges code
   bug is copied into the test    test stands outside, can say ✗
   green = "code does what          green = "code meets the rule"
   it does" (says nothing)
  ```
- **Why it lands:** "a green suite built this way is a mirror, not a check" is the page's killer
  line — and a mirror is a *literal* drawable object. The strongest metaphor in the chapter.
- **Kin:** none. Net-new, flagship.
- **Render:** SVG.

#### G18 · Property-based testing — one example vs a swarm ⭐P1
- **Anchors:** "A property-based test … lets a tool **generate** the inputs — hundreds of them,
  including the weird edges."
- **Concept:** left — the agent's single hand-picked happy-path example, sailing past the bug.
  Right — a swarm of generated inputs, one of which lands on the overdraw edge and trips the
  invariant.
- **Composition:**
  ```
   HAND-PICKED                   GENERATED SWARM
   • $100 A→B  ✓ (misses bug)    •••••••• hundreds
                                 ••• ↑ overdraw case → ✗ caught
   one path the agent coded      asserts the invariant, hunts edges
  ```
- **Why it lands:** the "swarm finds the edge your one example skipped" idea is inherently spatial
  — dots scattered across an input space with one landing on the fault line.
- **Kin:** none in the article. Net-new.
- **Render:** SVG.

#### G19 · Mutation testing — inject a bug, does the suite scream? ⭐P1
- **Anchors:** "Mutation testing tests the tests themselves … a surviving mutant is proof your
  tests would *not* have caught that bug."
- **Concept:** inject a small deliberate bug (`<` → `<=`); a strong suite goes red (mutant
  *killed*); a toothless suite stays green (mutant *survived* → a worklist item for the agent). The
  visual answer to the coverage illusion.
- **Composition:**
  ```
   inject  <  →  <=
      ├─ suite RED   ⇒ mutant killed ✓ (teeth)
      └─ suite GREEN ⇒ mutant SURVIVED ✗ → hand to agent to kill
   coverage said "line ran"; survival says "nothing checked it"
  ```
- **Why it lands:** mutation testing is the least-intuitive idea in the chapter; the "inject a bug
  and see if anything screams" loop is far clearer drawn than described. Closes the loop with G18
  (the page's PBT → mutation-score → kill-the-survivor cycle could even be one combined figure).
- **Kin:** none. Net-new.
- **Render:** SVG, or Mermaid for the branch.

---

## 5. Render & tooling recommendation (answers `PLAN.md` §6 Q4)

**Recommendation: add the diagrams — text-only undersells these chapters — using two formats by
job.**

- **Hand-drawn SVG (Excalidraw export)** for the *metaphor* graphics — G1 lever, G5 staircase, G6
  terrain, G8 budget bar, G10 fork, G11 map/maze, G13 pyramid, G14 hollow-vs-eye, G16 escape hatch,
  G17 mirror, G18 swarm. The warm, sketchy aesthetic matches the article *and* the playbook's
  "friendly senior engineer" voice (§2.4). Export `.svg` (crisp, diff-able-ish, themeable) into
  `_assets/`, one file per graphic, named `gNN-slug.svg`.
- **Mermaid** for the *structural* graphics that are plain flows — G2 timeline, G7 steering loop,
  G15 placement line, G19 branch. Mermaid renders natively in GitHub-flavoured Markdown (no build
  step), stays in the repo as text, and is trivial to keep current — which matters because a stale
  diagram "misleads with the full authority of being read first" (the playbook's own warning,
  applied to its own artwork).

Both are checked into the repo, both survive a tool swap (an SVG is an SVG; Mermaid is a documented
open format) — so this stays inside the §1.1 tool-agnostic spirit. **No product names, no numbers,
no metrics inside any artwork** (§1.1 / §4). Captions follow the same no-citation rule as prose.

If the playbook later ships an HTML render (`PLAN.md` §6 Q2), G8 (budget bar) and G15 (placement
line) are natural candidates to become live HTML/CSS rather than static images.

---

## 6. Priority rollout

Build the **shared kit (§3) first**, then ship in waves so each page gets its single highest-value
image before any page gets its second.

- **P0 — the flagship eight (one per concept that recurs or anchors a chapter):**
  G1 (lever), G7 (steering loop), G10 (the fork / reward-hacking — reused 3×), G11 (map vs maze),
  G13 (sensor pyramid), G14 (hollow vs state-asserting), G15 (placement line), G17 (mirror vs
  invariant). These cover the eight ideas a reader *must* leave with.
- **P1 — the reinforcers:** G2, G3, G4, G5, G6, G8, G12, G16, G18, G19.
- **P2 — nice-to-have:** G9 (blueprint beats a rule).

One graphic per page is the floor; the spine page (Foundations 03) carries several because it
introduces every concept the others specialise.

---

## 7. Open questions

1. **Hand-drawn vs clean-line aesthetic.** Excalidraw gives the warm Fowler look; a clean
   geometric style (e.g. straight SVG) reads more "enterprise." Which voice fits the playbook's
   two audiences (engineers + technical sponsors)? *Leaning hand-drawn for warmth, per §2.4.*
2. **Reuse depth for G10.** Build the fork once and embed the identical asset on 20-harness/01, /03,
   and 40-anti-patterns? (Recommended — a recurring signature image teaches "this is the same
   failure" for free.)
3. **Combine G18 + G19** into one "PBT → mutation score → kill the survivor" loop figure, matching
   the page's "virtuous loop" paragraph, or keep them as two simpler graphics? *Leaning two, for
   junior readability.*
4. **Captions.** Figure captions in the style of the article ("Figure 1: …"), or rely on the
   anchoring section heading and keep images caption-light? The playbook has no figure-numbering
   convention yet.
5. **Dark-mode.** If an HTML render lands, do the SVGs need a dark variant, or do we author them
   theme-neutral from the start?
