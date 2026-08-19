<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v6.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v6.0.0** was hardened automatically. 14 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ }} expressions are interpolated directly inside run: shell command strings in action.yml, violating sub-rule (a). This allows an attacker-controlled value to be parsed by the shell before quoting can protect it.

1. 'Check system dependencies' step: `if [ "${{ inputs.skip_validation }}" != "true" ]` — inputs.skip_validation interpolated directly into shell.
2. 'Set safe directory' step: `git config --global --add safe.directory "${{ github.workspace }}"` — github.workspace interpolated directly into shell.
3. 'Get and set token' step: `if [ "${{ inputs.use_oidc }}" == 'true' ]` — inputs.use_oidc interpolated directly into shell.
4. 'Get and set token' step: `elif [ -n "${{ env.CODECOV_TOKEN }}" ]` and `echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV"` — env.CODECOV_TOKEN interpolated directly into shell.
5. 'Get and set token' step: `if [ -n "${{ inputs.token }}" ]` — inputs.token interpolated directly into shell.

Locations:

- `action.yml:163`
- `action.yml:176`
- `action.yml:196`
- `action.yml:199`
- `action.yml:201`
- `action.yml:203`

### script-injection (severity: high)

In .github/workflows/main.yml, `${{ steps.codecov-upload.outcome }}` (a steps.*.outputs.* context value) is interpolated directly inside run: shell command strings in two separate steps, violating sub-rule (a). Offending lines: `if [ "${{ steps.codecov-upload.outcome }}" = "failure" ]` in the 'Verify dependency check failed' steps of the run-alpine-missing-deps and run-alpine-partial-deps jobs.

Locations:

- `.github/workflows/main.yml:131`
- `.github/workflows/main.yml:196`

### github-env-injection (severity: high)

Multiple run: steps in action.yml write untrusted values to $GITHUB_ENV without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`):

1. 'Get and set token' step: `echo "CC_TOKEN=$CC_OIDC_TOKEN" >> "$GITHUB_ENV"` — CC_OIDC_TOKEN is set from steps.oidc.outputs.result (untrusted step output), no sanitization.
2. 'Get and set token' step: `echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV"` — env.CODECOV_TOKEN (untrusted env context) written directly to GITHUB_ENV without sanitization.
3. 'Override branch for forks' step: `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` — TOKENLESS is set from $GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL (attacker-controlled via PR), no sanitization.
4. 'Override branch for forks' step: `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"` — CC_BRANCH is set from $GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL (attacker-controlled), no sanitization.
5. 'Override commits and pr for pull requests' step: `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` — CC_SHA is set from $GITHUB_EVENT_PULL_REQUEST_HEAD_SHA (attacker-controlled), no sanitization.
6. 'Override commits and pr for pull requests' step: `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"` — CC_PR is set from $GITHUB_EVENT_NUMBER (attacker-controlled), no sanitization.

Locations:

- `action.yml:196`
- `action.yml:201`
- `action.yml:218`
- `action.yml:220`
- `action.yml:234`
- `action.yml:235`

### unpinned-uses (severity: high)

Multiple workflow files reference GitHub Actions by mutable version tags instead of full 40-character commit SHAs, making them vulnerable to supply-chain attacks if the tag is moved.

In .github/workflows/main.yml:
- `actions/checkout@v5.0.0` (6 occurrences)

In .github/workflows/codeql-analysis.yml:
- `actions/checkout@v5.0.0`
- `github/codeql-action/init@v3.30.0`
- `github/codeql-action/autobuild@v3.30.0`
- `github/codeql-action/analyze@v3.30.0`

In .github/workflows/scorecards-analysis.yml:
- `actions/checkout@v5.0.0`
- `github/codeql-action/upload-sarif@v3.30.0`

Locations:

- `.github/workflows/main.yml:10`
- `.github/workflows/codeql-analysis.yml:28`
- `.github/workflows/codeql-analysis.yml:33`
- `.github/workflows/codeql-analysis.yml:43`
- `.github/workflows/codeql-analysis.yml:50`
- `.github/workflows/scorecards-analysis.yml:20`
- `.github/workflows/scorecards-analysis.yml:55`

### broad-permissions (severity: medium)

The workflow file .github/workflows/scorecards-analysis.yml has a top-level `permissions: read-all` declaration. This grants overly broad read access to all scopes and should be replaced with specific minimal permissions.

Locations:

- `.github/workflows/scorecards-analysis.yml:10`

### missing-permissions (severity: medium)

The workflow file .github/workflows/enforce-license-compliance.yml has no top-level `permissions:` key and no job-level `permissions:` key on any of its jobs. Without explicit permissions, the workflow inherits the repository's default token permissions, which may be overly broad.

Locations:

- `.github/workflows/enforce-license-compliance.yml:1`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.skip_validation }}" appears directly in run: block of step "Check system dependencies"; move to env: map

Locations:

- `action.yml:191`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.use_oidc }}" appears directly in run: block of step "Get and set token"; move to env: map

Locations:

- `action.yml:248`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.token }}" appears directly in run: block of step "Get and set token"; move to env: map

Locations:

- `action.yml:256`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.token }}" appears directly in run: block of step "Get and set token"; move to env: map

Locations:

- `action.yml:259`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $CC_OIDC_TOKEN in step "Get and set token" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:250`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $CC_BRANCH in step "Override branch for forks" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:278`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $CC_SHA in step "Override commits and pr for pull requests" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:297`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $CC_PR in step "Override commits and pr for pull requests" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:298`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, broad-permissions, missing-permissions, static-inline-injection, static-unsanitized-env-write

**Notes:**

Fixed all findings across action.yml and workflow files:

1. action.yml script-injection: Moved inputs.skip_validation (Check system dependencies), github.workspace (Set safe directory), inputs.use_oidc, env.CODECOV_TOKEN, and inputs.token (Get and set token) from inline ${{ }} expressions in run: blocks to env: blocks.

2. main.yml script-injection: Moved steps.codecov-upload.outcome to env: blocks (CODECOV_UPLOAD_OUTCOME) in both 'Verify dependency check failed' steps in run-alpine-missing-deps and run-alpine-partial-deps jobs.

3. action.yml github-env-injection: Added printf '%s' ... | tr -d '\n\r' sanitization before all GITHUB_ENV writes for CC_OIDC_TOKEN, CODECOV_TOKEN, CC_BRANCH, TOKENLESS, CC_SHA, and CC_PR.

4. unpinned-uses: Pinned actions/checkout@v5.0.0 → @08c6903cd8c0fde910a37f88322edcfb5dd907a8 (6 occurrences in main.yml, 1 in codeql-analysis.yml, 1 in scorecards-analysis.yml); pinned github/codeql-action/{init,autobuild,analyze,upload-sarif}@v3.30.0 → @2d92b76c45b91eb80fc44c74ce3fce0ee94e8f9d.

5. broad-permissions: Replaced top-level `permissions: read-all` with `permissions: {}` in scorecards-analysis.yml (job-level permissions already specify needed scopes).

6. missing-permissions: Added `permissions: contents: read` to enforce-license-compliance.yml.

