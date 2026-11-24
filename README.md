# oss-fuzz pr_helper lab (Blume, google OSS VRP lane)

Reproduces `google/oss-fuzz/.github/workflows/pr_helper.yml` (pull_request_target, `types: [opened]`,
`paths: projects/**`, `contents: read` + `pull-requests: write`) against the **pre-fix** version of
`infra/pr_helper.py` (i.e. before commit ec12d23d / PR #15285, "Fix GITHUB_ENV injection via crafted
project.yaml main_repo", 2026-04-02) and against the **current fixed** version.

`main`       = vulnerable (pre-fix) script
`main-fixed` = current upstream script

Rig adaptations (not part of the vulnerability): OWNER/REPO come from env instead of hard-coded
google/oss-fuzz; CRITICALITY_SCORE_PATH points at /bin/true so the subprocess returns non-JSON and the
script takes its documented "N/A" path.
