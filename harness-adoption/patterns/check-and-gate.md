# Pattern: the `check` command and the blocking gate

## Intent

Two halves of one idea. The **`check` command** is a single entry point
that runs format + lint + types (and, once it exists, the behaviour
test) and exits non-zero if any fail. The **gate** runs `check` at the
earliest stage it can *enforce* and refuses to let red through. Together
they turn "we have a linter" into "red does not get in." See
`../../playbook-md/20-harness/04-keep-quality-left.md`.

## What good looks like

- **One command.** `check` is what the guide names, what the gate runs,
  and what a human runs by hand. Nobody memorises three commands.
- **Earliest enforceable stage.** Pre-commit for fast checks, pre-push
  for the heavier ones, CI as the authoritative gate that can't be
  bypassed with a local flag.
- **Keep quality left.** ms–5s checks pre-commit; 5–90s pre-push;
  minutes-long and LLM checks in CI. A slow pre-commit trains people to
  use `--no-verify`.

## Blocking behaviour

**This is the whole point.** A failing `check` must fail the push / fail
the merge. Warn-only is not a gate; it's a dashboard nobody reads. The
adopt smoke-check (make a violating change, confirm the gate says no) is
mandatory for this pattern.

## Assessment signal

- Is there a single `check`-like command? (script entry, Makefile target.)
- Is there a hook or CI step that runs it?
- **Does it block or warn?** Detection can't tell reliably — ask. This is
  the most common false "closed loop".
- Bypass rate: grep history/config for `--no-verify`, disabled checks,
  `continue-on-error: true` in CI.

## Recipes by stack

### The `check` command

| Stack | One `check` entry point |
|-------|-------------------------|
| **Generic** | A `Makefile` target `check:` that runs format-check, lint, and type-check, `&&`-chained so any failure exits non-zero. Works for every stack and is CI-portable. |
| **JS / TS** | `package.json` script: `"check": "prettier --check . && eslint . --max-warnings=0 && tsc --noEmit"`. For a richer router (touched files on commit, full set on push), wrap these in a small script and split by hook stage. |
| **Python** | `ruff format --check . && ruff check . && mypy .` — wrap in a `make check` target or a `[tool.poe]`/`tox` task. |
| **Go** | `gofmt -l . (fail if non-empty) && go vet ./... && golangci-lint run` — `go build ./...` covers type-checking. |
| **JVM (Java/Kotlin)** | A Gradle/Maven verify task: `spotlessCheck` + `checkstyle`/`ktlint` + `compileJava`/`compileKotlin`. |
| **.NET** | `dotnet format --verify-no-changes && dotnet build -warnaserror`. |
| **Ruby** | `bundle exec rubocop` (+ `bundle exec srb tc` if Sorbet) — wrap in a Rake `check` task. |

The router idea (run only touched files on commit, the full set on push)
is portable — start with the simple `&&` chain; graduate to a router when
the full run gets slow.

### The gate

| Mechanism | When to choose it | Note |
|-----------|-------------------|------|
| **Bare git hook** (`.git/hooks/pre-push` calling `make check`) | any stack, zero dependencies | not shared automatically — commit a `scripts/install-hooks` step |
| **pre-commit framework** (`.pre-commit-config.yaml`) | Python and polyglot repos | language-agnostic, widely adopted; `pre-commit install` shares it |
| **Lefthook** (`lefthook.yml`) | JS/TS and polyglot | single binary, easy YAML; `pre-commit:`/`pre-push:` each run one command that dispatches to `check`. See lefthook docs. |
| **Husky** | JS/TS teams already on it | fine, but heavier than lefthook |
| **CI step** | always, as the authoritative gate | the local hook is a courtesy; CI is what can't be bypassed. See `ci-and-vcs.md` |

## How adopt writes it

1. Create the `check` entry point in the stack's idiom (above). Make it
   the single source of truth for "passing".
2. Point `AGENTS.md` at it.
3. Wire the gate at the earliest stage the team can enforce. Pick the
   mechanism from the table that fits the stack and the team's existing
   tooling — don't introduce a second hook manager if one exists.
4. **Smoke-check the block:** run `check` clean; then introduce a lint
   error / type error in a scratch file and confirm the gate fails;
   revert. Report both results.
5. If CI exists, add `check` there too as the authoritative gate
   (`ci-and-vcs.md`), and treat the local hook as the fast courtesy copy.
