# oss-fuzz pr_helper lab (Blume — google OSS VRP lane)

Reproduces `google/oss-fuzz/.github/workflows/pr_helper.yml` — `pull_request_target`,
`types: [opened]`, `branches: [master]`, `paths: ['projects/**']`, job permissions
`contents: read` + `pull-requests: write` — against both the **pre-fix** and the **current** version
of `infra/pr_helper.py`.

## Runs (evidence)

| run | base-branch script | payload | result |
|---|---|---|---|
| [45464313184](https://github.com/pwnhaven-afk/oss-fuzz-prhelper-lab/actions/runs/45464313184) (PR #2) | pre-fix `infra/pr_helper_vulnerable.py` | `projects/evilproj/project.yaml` with a block-scalar `main_repo` | **injection lands**: `CANARY_SINK=https://httpbin.org/anything` and `EVILPAD=<br/> - Criticality score: N/A<br/>` appear in the env of every later step; the privileged consumer sends the real `GITHUB_TOKEN` (log-masked) and base64 of repo secret `CANARY` to that host |
| [45465020091](https://github.com/pwnhaven-afk/oss-fuzz-prhelper-lab/actions/runs/45465020091) (PR #7) | current upstream `infra/pr_helper_fixed.py` | identical | **no injection**: `MESSAGE` collapses to one line, `CANARY_SINK=` and `EVILPAD=` are empty |
| [45465458223](https://github.com/pwnhaven-afk/oss-fuzz-prhelper-lab/actions/runs/45465458223) (PR #8) | current upstream | opened benign, force-pushed 2 s later | **ordering window proven**: the run is admitted by the opened snapshot's `paths` filter but evaluates the post-push head |

## Branches

- `main` — current upstream script (the control).
- `main-fixed` — same, kept for the branch-scoped experiment.
- `attack-*` — the payload branches behind the PRs above.

Note: `pull_request_target` does not fire for PRs whose base is a non-default branch in this repo,
so both variants are exercised on `main`, switched by commit (see PR #2 vs PR #7).

## Rig adaptations (not part of the vulnerability)

`OWNER`/`REPO` come from env instead of hard-coded `google`/`oss-fuzz`; `CRITICALITY_SCORE_PATH`
points at `/bin/true` so the subprocess takes the script's documented "N/A" branch.

## Status

This mirrors a bug that Google already found and fixed: `google/oss-fuzz` commit `ec12d23d`
(2026-04-02), *"Fix GITHUB_ENV injection via crafted project.yaml main_repo (#15285)"*,
`b/483022611`. The lab exists as reproduction evidence only; nothing here is a new finding.
