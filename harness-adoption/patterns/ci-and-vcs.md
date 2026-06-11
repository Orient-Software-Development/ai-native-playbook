# Pattern: CI and VCS wiring

## Intent

The local gate is a fast courtesy that a developer can bypass with a
flag. **CI is the authoritative gate** — it runs the same `check` where
nobody can `--no-verify` it, and it's where expensive and inferential
checks live. VCS settings (branch protection, required checks) are what
make CI actually enforce. See
`../../playbook-md/30-delivery/02-ci-and-cd.md`.

This is a "grow the harness" control — week one a pre-push hook is
enough. Add CI as the second gate once the team merges through a shared
remote.

## What good looks like

- CI runs `check` (the same one the local gate runs) on every PR.
- Expensive checks (e2e, dependency graph, bundle budget) and inferential
  review live in CI, not on the fast path.
- The default branch is protected: PR required, CI required to merge, no
  force-push.
- CI is fast and trusted — a red CI nobody believes is no gate at all.

## Blocking behaviour

A failing required check **blocks the merge**. The point of CI over the
local hook is that this can't be skipped locally. Branch protection is
what turns "CI ran" into "CI must pass to merge".

## Assessment signal

- Is there CI config, and does it run on PRs? Is it green and trusted?
- Does it run the same `check`, or a divergent set (drift risk)?
- Is the default branch protected with required checks?
- `continue-on-error: true` / soft-fail steps = a half-loop in CI.

## Recipes by CI host

| Host | File | Run `check` |
|------|------|-------------|
| **GitHub Actions** | `.github/workflows/ci.yml` | a job that checks out, sets up the toolchain, installs deps, then runs `make check` / the stack's check |
| **GitLab CI** | `.gitlab-ci.yml` | a `test` stage job running the same check |
| **Azure Pipelines** | `azure-pipelines.yml` | a step invoking the check |
| **Bitbucket** | `bitbucket-pipelines.yml` | a step invoking the check |
| **Jenkins** | `Jenkinsfile` | a `sh 'make check'` stage |
| **CircleCI** | `.circleci/config.yml` | a job running the check |

Because the local gate and CI both call **the same `check` command**,
the CI recipe is nearly host-independent — install dependencies, then run
`check`. Keep them identical to avoid "passes locally, fails in CI" drift.

## Recipes by VCS (branch protection)

| Host | Make the gate enforce |
|------|----------------------|
| **GitHub** | Settings → Branches → protect default: require PR, require the CI status check, disallow force-push. `gh api` can script it. |
| **GitLab** | Protected branches + "pipelines must succeed" merge check. |
| **Azure DevOps** | Branch policies: require build validation + min reviewers. |
| **Bitbucket** | Branch permissions + required builds. |

## How adopt writes it

1. Add a CI workflow for the detected host that runs the **same `check`**
   the local gate runs. Add expensive/inferential jobs as separate steps.
2. Turn on branch protection / required checks for the default branch
   (confirm with the user first — it changes everyone's workflow).
3. Note in `AGENTS.md` that CI is the authoritative gate and the local
   hook is the fast copy.
4. Smoke-check: open a throwaway PR with a failing check and confirm the
   merge is blocked; close it.

## Decay notes (for audit)

- CI and local `check` drifting apart = "green locally, red in CI" pain →
  reunify them.
- Required checks quietly removed, or `continue-on-error` added = the
  authoritative gate going soft.
