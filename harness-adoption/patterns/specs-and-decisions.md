# Pattern: specs, decisions, and behaviour-contract packs

## Intent

Feedforward that captures **what to build** (specs), **what was decided
and why** (ADRs), and — for complex domains — **the behaviour contract**
the implementation must satisfy (scenario packs). These shape the agent's
work before it acts and give review something to check against. See
`../../playbook-md/10-lifecycle/01-spec-the-contract.md`.

These grow *with the product*. Don't stand up a full spec corpus on day
one — adopt the lightweight version when the team keeps re-deciding the
same things, and the heavyweight packs only for domains that warrant them.

## Three levels — adopt the lightest that fits

### 1. A spec template + folder (most teams)

A `specs/` (or `docs/specs/`) folder and one template: problem, the
contract (inputs/outputs/invariants), out-of-scope, acceptance criteria
with stable `AC-n` ids. Status in frontmatter
(`draft → approved → implemented`). The agent writes a spec before
building anything non-trivial; review checks code against it.
Template: `../reference/spec-and-adr-templates.md`.

Two disciplines that keep level 1 honest:

- **Interview first.** The spec starts with the agent interviewing the
  intent-owner — challenging assumptions, pinning success criteria,
  deciding what should *not* be built — then writing the spec from the
  interview. Execution starts in a fresh session; the spec, not the
  transcript, carries forward.
- **Scale the ceremony.** A one-line fix gets a sentence and a repro
  check, not the full template. Elaborate machinery on a small change
  produces review burden, not safety — the paperwork grows and shrinks
  with the problem.

### 2. Decision records (ADRs)

When a decision is a *decision*, not an implementation — a choice with
trade-offs you'll be asked about later — record it as an ADR: context,
options, decision, consequences. One file per decision, numbered, never
edited after acceptance (supersede instead). Template:
`../reference/spec-and-adr-templates.md`.

### 3. Behaviour-contract packs (complex domains only)

The heavyweight option: pin **invariants** (cross-entity rules that must
always hold), enumerate **approved scenarios** (one file per scenario,
each naming the exact expected behaviour), and a **coverage matrix**
(which command × dimension cells need a scenario). Then add **sensors**
that fail the build when the chain breaks — every scenario maps to a
test, every test names a scenario, the matrix is well-formed, and a
change to the implementation moves in the same diff as the scenario it
affects. The chain: spec → invariant → matrix cell → scenario → test →
implementation. See `../reference/spec-and-adr-templates.md` (last
section) for when this is justified.

**Only adopt this when** a domain has many commands sharing invariants
and a real cost to silent divergence (billing, tenancy, access control).
For everything else it's over-engineering — assessment should *not* rank
it highly by default. It is the kit's most specialised piece, distilled
from one multi-tenant SaaS.

## Blocking behaviour

- Spec/ADR existence is mostly feedforward (not gated), but you *can*
  gate "a spec exists and is `approved` before implementation" for
  flagged domains.
- **Spec changes get classified before merge**: additive / compatible /
  breaking / ambiguous. The first two flow through the normal gate;
  breaking and ambiguous block on human sign-off — they redefine the
  contract the rest of the harness measures against. An agent can
  propose the classification; a person confirms the two that redefine.
- Behaviour-contract packs come with sensors that **do** gate: SCN↔test
  traceability, matrix well-formedness, same-diff evidence. Those run in
  the gate like any sensor. The lightweight version of the same idea is
  the AC-id traceability scan in `drift-and-health.md`.

## Assessment signal

- Where do specs/contracts live — a folder, a tracker, or nowhere?
- Are decisions recorded anywhere, or only in PR comments and chat?
- Is there a domain complex enough to warrant a contract pack, and is the
  team already feeling the pain of spec/code drift there?

## How adopt writes it

1. Default to **level 1** — drop the spec folder + template, add a line
   to `AGENTS.md`: "write/approve a spec in `specs/` before building
   anything non-trivial — interview the intent-owner first, and scale
   the spec to the change (a one-line fix needs a sentence and a repro
   check, not the template)."
2. Add **level 2** (ADR template + folder) if decisions are getting lost.
   Record the harness adoption itself as the first ADR — it's a real
   decision with consequences.
3. Reach for **level 3** only when assessment flags a genuinely complex
   domain. Port the pack structure and its sensors from the reference
   kit; apply it to *one* active domain, not retroactively across the
   codebase.

## Decay notes (for audit)

- **Spec rot**: a spec marked `implemented` that no longer matches the
  code is worse than no spec. Check a sample each audit; supersede stale
  ones.
- ADR index references that don't resolve = feedforward decay.
