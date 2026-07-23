<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v4.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v4.6.0** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions by mutable version tags instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved.

.github/workflows/main.yml:
  - uses: actions/checkout@v4.2.0 (appears in all 3 jobs: run, run-macos-latest-xlarge, run-container)

.github/workflows/codeql-analysis.yml:
  - uses: actions/checkout@v4.2.0
  - uses: github/codeql-action/init@v3.26.9
  - uses: github/codeql-action/autobuild@v3.26.9
  - uses: github/codeql-action/analyze@v3.26.9

.github/workflows/scorecards-analysis.yml:
  - uses: actions/checkout@v4.2.0
  - uses: github/codeql-action/upload-sarif@v3.26.9

Locations:

- `.github/workflows/main.yml:10`
- `.github/workflows/main.yml:47`
- `.github/workflows/main.yml:79`
- `.github/workflows/codeql-analysis.yml:32`
- `.github/workflows/codeql-analysis.yml:36`
- `.github/workflows/codeql-analysis.yml:43`
- `.github/workflows/codeql-analysis.yml:55`
- `.github/workflows/scorecards-analysis.yml:19`
- `.github/workflows/scorecards-analysis.yml:55`

### missing-permissions (severity: medium)

These workflow files have no top-level `permissions:` block and no job-level `permissions:` block on any job. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially write) permissions, violating the principle of least privilege.

- .github/workflows/main.yml: 3 jobs (run, run-macos-latest-xlarge, run-container) — none have permissions defined.
- .github/workflows/enforce-license-compliance.yml: 1 job (enforce-license-compliance) — no permissions defined.

Locations:

- `.github/workflows/main.yml:1`
- `.github/workflows/enforce-license-compliance.yml:1`

### broad-permissions (severity: medium)

The workflow sets top-level `permissions: read-all`, which grants read access to all repository scopes rather than the minimal specific permissions required. This should be replaced with only the specific permissions needed by each job.

Locations:

- `.github/workflows/scorecards-analysis.yml:11`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, broad-permissions

**Notes:**

Fixed all three findings across four workflow files:

1. unpinned-uses: Pinned all mutable tag references to full commit SHAs:
   - actions/checkout@v4.2.0 → @d632683dd7b4114ad314bca15554477dd762a938 (3 occurrences in main.yml, 1 in codeql-analysis.yml, 1 in scorecards-analysis.yml)
   - github/codeql-action/init@v3.26.9 → @461ef6c76dfe95d5c364de2f431ddbd31a417628
   - github/codeql-action/autobuild@v3.26.9 → @461ef6c76dfe95d5c364de2f431ddbd31a417628
   - github/codeql-action/analyze@v3.26.9 → @461ef6c76dfe95d5c364de2f431ddbd31a417628
   - github/codeql-action/upload-sarif@v3.26.9 → @461ef6c76dfe95d5c364de2f431ddbd31a417628

2. missing-permissions: Added `permissions: contents: read` top-level block to main.yml and enforce-license-compliance.yml.

3. broad-permissions: Replaced `permissions: read-all` with `permissions: contents: read` in scorecards-analysis.yml. The job-level block already had specific minimal permissions (security-events: write, id-token: write, actions: read, contents: read) which are preserved.

