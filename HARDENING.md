<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v7.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v7.0.0** was hardened automatically. 10 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The 'Set safe directory' step directly interpolates `${{ github.workspace }}` inside the run: shell command string. Before the shell executes the script, GitHub Actions substitutes the expression value verbatim into the shell source, allowing an attacker who can control the workspace path to inject arbitrary shell commands. Offending line: `git config --global --add safe.directory "${{ github.workspace }}"`

Locations:

- `action.yml:196`

### github-env-injection (severity: high)

The 'Get and set token' step writes `CC_OIDC_TOKEN` (sourced from `steps.oidc.outputs.result`, an untrusted step output) directly to $GITHUB_ENV without sanitization: `echo "CC_TOKEN=$CC_OIDC_TOKEN" >> "$GITHUB_ENV"`. A newline in the token value could inject arbitrary environment variables into subsequent steps. The required sanitization (`printf '%s' "$CC_OIDC_TOKEN" | tr -d '\n\r'`) is absent.

Locations:

- `action.yml:232`

### github-env-injection (severity: high)

The 'Get and set token' step writes `INPUT_CODECOV_TOKEN` (sourced from `env.CODECOV_TOKEN`, a workflow-controlled environment variable) directly to $GITHUB_ENV without sanitization: `echo "CC_TOKEN=$INPUT_CODECOV_TOKEN" >> "$GITHUB_ENV"`. A calling workflow can set CODECOV_TOKEN to a value containing newlines, injecting arbitrary environment variables. The required sanitization is absent.

Locations:

- `action.yml:236`

### github-env-injection (severity: high)

The 'Override branch for forks' step writes `TOKENLESS` and `CC_BRANCH` — both derived from `github.event.pull_request.head.label` (attacker-controlled in pull requests) via the `GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL` env var — to $GITHUB_ENV without sanitization: `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"`. A malicious PR branch name containing newlines can inject arbitrary environment variables into subsequent steps.

Locations:

- `action.yml:253`
- `action.yml:256`

### github-env-injection (severity: high)

The 'Override commits and pr for pull requests' step writes `CC_SHA` (sourced from `inputs.override_commit` / `github.event.pull_request.head.sha`) and `CC_PR` (sourced from `inputs.override_pr` / `github.event.number`) to $GITHUB_ENV without sanitization: `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` and `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"`. Attacker-controlled input values containing newlines can inject arbitrary environment variables into subsequent steps.

Locations:

- `action.yml:269`
- `action.yml:270`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $CC_OIDC_TOKEN in step "Get and set token" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:252`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_CODECOV_TOKEN in step "Get and set token" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:256`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $CC_BRANCH in step "Override branch for forks" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:283`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $CC_SHA in step "Override commits and pr for pull requests" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:302`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $CC_PR in step "Override commits and pr for pull requests" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:303`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all 10 findings in action.yml:
1. script-injection (line 196): Moved `${{ github.workspace }}` into an env var `WORKSPACE_PATH` and referenced it as `"$WORKSPACE_PATH"` in the shell script.
2. github-env-injection + static-unsanitized-env-write for CC_OIDC_TOKEN and INPUT_CODECOV_TOKEN (lines 232, 236): Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before writing to $GITHUB_ENV in the 'Get and set token' step.
3. github-env-injection + static-unsanitized-env-write for TOKENLESS and CC_BRANCH (lines 253, 256): Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before writing to $GITHUB_ENV in the 'Override branch for forks' step.
4. github-env-injection + static-unsanitized-env-write for CC_SHA and CC_PR (lines 269, 270): Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before writing to $GITHUB_ENV in the 'Override commits and pr for pull requests' step.

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, broad-permissions

**Notes:**

1. unpinned-uses: Pinned all actions/checkout@v5.0.0 (6 occurrences in main.yml, 1 in codeql-analysis.yml, 1 in scorecards-analysis.yml) to SHA 08c6903cd8c0fde910a37f88322edcfb5dd907a8. Pinned github/codeql-action/init, autobuild, analyze, and upload-sarif @v3.30.0 to SHA 2d92b76c45b91eb80fc44c74ce3fce0ee94e8f9d. Tags preserved as inline comments.
2. script-injection: Fixed two run: blocks in main.yml (run-alpine-missing-deps and run-alpine-partial-deps jobs) that interpolated ${{ steps.codecov-upload.outcome }} directly in shell strings. Moved the expression to an env: block as CODECOV_OUTCOME and updated the shell script to reference $CODECOV_OUTCOME.
3. broad-permissions: Replaced top-level 'permissions: read-all' in scorecards-analysis.yml with specific minimal permissions (contents: read). The job already has its own permissions block with the specific scopes it needs.

