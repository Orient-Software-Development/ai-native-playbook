# Pattern: architecture fitness

## Intent

Regulate **structural properties of the system** — module boundaries,
layer order, banned dependencies, no cycles. Fitness functions encode
"X may not import Y" as a machine check so the architecture can't erode
one convenient import at a time. See
`../../playbook-md/20-harness/02-repo-structure-and-legibility.md`.

Adopt this *after* the minimum viable harness, and only once there are
boundaries worth enforcing. On a repo with no clear modules, the higher-
value move is to draw the boundaries first (a feedforward + structure
job), then add the fitness check.

## What good looks like

- The rules a reviewer keeps repeating ("don't import the DB layer from
  the UI", "domain code can't depend on the web framework") are encoded
  as checks, not comments.
- Checks run in the gate (cheap ones pre-push, graph analysis in CI).
- Each rule maps to a line in `AGENTS.md` so feedforward and feedback
  name the same boundary.

## Blocking behaviour

A violated boundary fails the gate. Cheap pattern/lint-based rules can go
pre-push; full dependency-graph or cycle analysis usually belongs in CI.

## Assessment signal

- Are there named modules/packages or just a flat pile of files?
- Are imports between modules visible and lintable?
- Is any boundary already enforced (boundary lint, a graph tool), or only
  in review, or not at all?

## Recipes by stack

| Stack | Boundary lint | Dependency graph / cycles | Custom assertions |
|-------|---------------|---------------------------|-------------------|
| **JS/TS** | ESLint `no-restricted-imports` / `import/no-restricted-paths` | dependency-cruiser | AST/grep script (TypeScript compiler API or a regex pass) |
| **Python** | `import-linter` (contracts) | `pydeps`, `import-linter` layers | `ast` module script |
| **Go** | `depguard` (via golangci-lint) | `go mod graph`, `goda` | `go/analysis` vet pass |
| **JVM** | ArchUnit (tests that assert architecture) | ArchUnit / Gradle module deps | ArchUnit rules |
| **.NET** | Roslyn analyzers, NetArchTest | NetArchTest | Roslyn analyzer |
| **Ruby** | `packwerk` (package boundaries) | `packwerk` | custom rake check |

Generic fallback: a script that maps each file to a layer and asserts the
allowed-import matrix — ~60 lines in any language (build a path→layer map,
then scan each file's imports against an allowed-edges table).

## How adopt writes it

1. With the user, name the 1–3 boundaries that actually matter here (the
   rules review keeps repeating). Don't invent boundaries the codebase
   doesn't have.
2. Encode them with the stack's boundary tool (prefer the declarative
   option — `import-linter`/`depguard`/ArchUnit — over a hand-rolled
   script when one exists).
3. Wire into the gate at the right stage (lint-style pre-push; graph in CI).
4. Add each boundary to `AGENTS.md` with a pointer to the check.
5. Smoke-check: add a banned import, confirm failure, revert.

## Decay notes (for audit)

- New modules added since last audit without a boundary rule = drift.
- A fitness check with a growing allowlist of exceptions is eroding —
  review the exceptions.
