# Pattern: maintainability sensors

## Intent

Regulate **internal code quality** — what a reviewer cares about reading
a diff. Format, lint, and types are the cheap free sensors every project
should have. The budgets and the secret scan are the next increment: they
stop quiet, cumulative decay (suppressions, orphan TODOs, leaked secrets)
that no single diff makes obvious.

## The controls, in adoption order

1. **Formatter + linter** — cheapest sensors there are; catch style drift
   and a class of obvious bugs deterministically in ms.
2. **Type check** — if the language has one, the single highest-signal
   free sensor: a proof on every change that whole categories of bug
   can't be in the diff. Untyped language → skip and lean on tests.
3. **Suppression budget** — gate *net-new* escape hatches (`@ts-ignore`,
   `as any`, `# type: ignore`, `//nolint`, `@SuppressWarnings`,
   `#pragma warning disable`, `rubocop:disable`). Seed the ceiling at the
   current count so existing debt is grandfathered.
4. **TODO budget** — gate *net-new* orphan TODOs (no owner / no ticket).
5. **Secret scan** — a pre-commit tripwire for obvious secret patterns.
   Not a SAST; a last line before a key hits history.

Items 1–2 are minimum-viable (fold into `check`). Items 3–5 are the
"grow the harness" increment — adopt when assessment shows decay or on a
legacy repo where you want to stop the bleeding.

## Blocking behaviour

- Format/lint/types: fail `check` (and thus the gate). Lint runs with
  **warnings-as-errors** (`--max-warnings=0` or equivalent) — a warning
  nobody must fix is noise.
- Budgets: fail when the count exceeds the committed ceiling. The diff
  that adds the Nth+1 suppression is the one that's blocked.
- Secret scan: fail the commit on a match; allow an explicit, reviewed
  allowlist entry for false positives.

### Sensor ergonomics — design the interface for the agent

- **Failure messages carry the fix.** A check that can fail the build
  but can't explain itself does half its job. Name the rule, the
  violation, and the shape of correct code.
- **Suppress-with-reason.** Every suppression must carry a stated
  justification — most linters support it natively (`eslint-disable-next-line
  rule -- reason`, `# noqa: CODE` plus a comment, `//nolint:rule // reason`,
  `rubocop:disable` with a comment). Without a legitimate way out, an
  agent facing a rule it can't satisfy games it silently; with one, the
  exception sits in the diff as a documented judgment call a reviewer
  can read and veto. The escape hatch doesn't weaken the sensor — it
  routes exceptions into the open.
- **Threshold latitude, "refactor first".** For graded rules (a budget
  ceiling, a complexity limit), raising the threshold *with a stated
  reason* is legitimate — but write the order of preference into the
  guide: refactor first, raise only with a reason. Agents reach for the
  raise first and produce good refactors when pushed one step further.
- **Wrap big reports in a query interface.** A large sensor output
  (coverage, mutation report) dumped raw into context wastes the
  agent's attention on the 95% it doesn't need. Give it a summary /
  per-file / hotspots mode so the agent queries the slice it's working on.

## Assessment signal

- Config files present (`.eslintrc`/`eslint.config.*`, `.prettierrc`,
  `ruff.toml`/`pyproject`, `.golangci.yml`, `checkstyle.xml`,
  `.editorconfig`, `.rubocop.yml`)?
- Do they run in `check` / the gate, or only in an editor?
- Are warnings tolerated? (A linter run without a zero-warning gate is a
  half-loop.)
- Any existing budget files or secret-scan step? Count current
  suppressions/TODOs to know the seed ceiling.

## Recipes by stack

### Format / lint / types

| Stack | Format | Lint | Types |
|-------|--------|------|-------|
| **JS/TS** | Prettier `--check` | ESLint `--max-warnings=0` | `tsc --noEmit` |
| **Python** | `ruff format --check` | `ruff check` | `mypy`/`pyright` |
| **Go** | `gofmt -l` | `golangci-lint run` | `go build`/`go vet` |
| **JVM** | Spotless / ktlint | Checkstyle / detekt | compiler `-Werror` |
| **.NET** | `dotnet format --verify-no-changes` | analyzers | `dotnet build -warnaserror` |
| **Ruby** | RuboCop (format cops) | RuboCop | Sorbet `srb tc` (optional) |

### Suppression budget (net-new escape hatches)

Portable algorithm (~40 lines in any scripting language):

1. Count occurrences of the stack's suppression tokens across tracked files.
2. Compare to a committed ceiling in `.harness/suppression-budget.json`.
3. Fail if `count > ceiling`. An `--init` flag writes the current count as the ceiling.

Tokens per stack: TS `@ts-ignore|@ts-nocheck|as any`; Python
`# type: ignore|# noqa`; Go `//nolint`; Java `@SuppressWarnings`; C#
`#pragma warning disable`; Ruby `rubocop:disable`. Implement in whatever
scripting language the repo already uses — it's ~40 lines of grep+count.

### TODO budget

Same count-vs-ceiling shape: count `TODO`/`FIXME` lacking an owner or
ticket reference; gate net-new against a seeded ceiling.

### Secret scan

Prefer a maintained tool over hand-rolled: **gitleaks** or **trufflehog**
as a pre-commit/CI step (works for any stack). A hand-rolled regex
tripwire (a handful of high-signal patterns — private-key headers, AWS
keys, `password=`) is fine as a stopgap, but say plainly it's not a
replacement for gitleaks/Snyk.

## How adopt writes it

1. If format/lint/types aren't in `check`, add them (see `check-and-gate.md`)
   — that's the increment; stop there unless assessment showed decay.
2. For a budget: implement the count-vs-ceiling check in the repo's
   scripting idiom, run `--init` to seed the ceiling at current state
   (**critical on legacy repos** — grandfathers existing debt), wire it
   into pre-push, and note in `AGENTS.md` that net-new suppressions/TODOs
   are gated.
3. For secrets: prefer wiring gitleaks; only hand-roll if a dependency
   can't be added. Provide an allowlist mechanism for false positives.
4. Smoke-check each: clean run passes; a planted violation fails; revert.

## Decay notes (for audit)

- Budget *trend* matters more than the absolute number — a ceiling that
  only ever rises is a budget in name only. Report the trend.
- A linter with hundreds of warnings nobody fixes is noise — either gate
  at zero or remove the rules generating ignored warnings.
- **Track what each sensor catches, run over run.** Failures trending
  down means the guides are absorbing what feedback used to catch. A
  sensor that *never* fires is mastered (prune candidate) or broken —
  plant a violation and prove it still can. A rule that fails constantly
  marks exactly where a better guide is needed.
- Suppressions without reasons accumulating = the escape hatch has
  become a bypass. Re-require the justification.
