<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v4.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **codecov--codecov-action/v4.6.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions by mutable version tags instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved. Failing references:
- codeql-analysis.yml: actions/checkout@v4.2.0, github/codeql-action/init@v3.26.9, github/codeql-action/autobuild@v3.26.9, github/codeql-action/analyze@v3.26.9
- main.yml: actions/checkout@v4.2.0 (used in all three jobs)
- scorecards-analysis.yml: actions/checkout@v4.2.0, github/codeql-action/upload-sarif@v3.26.9

Locations:

- `.github/workflows/codeql-analysis.yml:38`
- `.github/workflows/codeql-analysis.yml:42`
- `.github/workflows/codeql-analysis.yml:52`
- `.github/workflows/codeql-analysis.yml:61`
- `.github/workflows/main.yml:10`
- `.github/workflows/scorecards-analysis.yml:27`
- `.github/workflows/scorecards-analysis.yml:57`

### missing-permissions (severity: medium)

main.yml has no top-level permissions: key and none of its jobs (run, run-macos-latest-xlarge, run-container) define a permissions: block. This means the workflow runs with the default (potentially broad) GitHub token permissions. enforce-license-compliance.yml similarly has no top-level or job-level permissions: block.

Locations:

- `.github/workflows/main.yml:1`
- `.github/workflows/enforce-license-compliance.yml:1`

### broad-permissions (severity: medium)

scorecards-analysis.yml sets top-level permissions: read-all, which grants read access to all repository scopes. This is overly broad; only the specific permissions actually needed (security-events: write, id-token: write, actions: read, contents: read) should be listed at the job level, not read-all at the workflow level.

Locations:

- `.github/workflows/scorecards-analysis.yml:11`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, broad-permissions

**Notes:**

Fixed all three findings: (1) Pinned all mutable action tags to full 40-char commit SHAs in codeql-analysis.yml, main.yml, and scorecards-analysis.yml — actions/checkout@v4.2.0 → d632683dd7b4114ad314bca15554477dd762a938, github/codeql-action/*@v3.26.9 → 461ef6c76dfe95d5c364de2f431ddbd31a417628. (2) Added top-level `permissions: {}` to main.yml and enforce-license-compliance.yml; added `contents: read` at each job level in main.yml since jobs need to checkout code. (3) Replaced broad `permissions: read-all` with `permissions: {}` at the top level of scorecards-analysis.yml; the job already had specific minimal permissions defined at the job level.

