<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v5.5.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v5.5.5** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `${{ }}` expressions are interpolated directly inside `run:` shell command strings in action.yml, allowing an attacker to inject arbitrary shell commands.

1. 'Check system dependencies' step: `if [ "${{ inputs.skip_validation }}" != "true" ]` — the `inputs.skip_validation` value is expanded by the template engine before the shell sees it, enabling shell metacharacter injection.

2. 'Set safe directory' step: `git config --global --add safe.directory "${{ github.workspace }}"` — `github.workspace` is interpolated directly into the shell command.

3. 'Get and set token' step: `if [ "${{ inputs.use_oidc }}" == 'true' ]`, `elif [ -n "${{ env.CODECOV_TOKEN }}" ]`, `echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV"`, `if [ -n "${{ inputs.token }}" ]`, and `CC_TOKEN=$(echo "${{ inputs.token }}" | tr -d '\n')` — all directly interpolate untrusted context values into shell commands.

Locations:

- `action.yml:181`
- `action.yml:196`
- `action.yml:224`
- `action.yml:227`
- `action.yml:230`
- `action.yml:232`
- `action.yml:234`

### github-env-injection (severity: high)

Multiple `run:` steps write values derived from untrusted inputs/github context to `$GITHUB_ENV` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`).

1. 'Get and set token' step: `echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV"` — writes `env.CODECOV_TOKEN` (a workflow-controlled value) directly to GITHUB_ENV with no newline sanitization. (Note: the `inputs.token` path uses `tr -d '\n'` but omits `\r`, which is insufficient.)

2. 'Override branch for forks' step: `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"` — `TOKENLESS` and `CC_BRANCH` are derived from the `GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL` env var (sourced from `github.event.pull_request.head.label`) and `CC_BRANCH` from `inputs.override_branch`, both written to GITHUB_ENV without sanitization.

3. 'Override commits and pr for pull requests' step: `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` and `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"` — `CC_SHA` comes from `inputs.override_commit` / `github.event.pull_request.head.sha` and `CC_PR` from `inputs.override_pr` / `github.event.number`, all written to GITHUB_ENV without sanitization.

Locations:

- `action.yml:230`
- `action.yml:248`
- `action.yml:250`
- `action.yml:265`
- `action.yml:266`

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

**Fixes applied:** script-injection, github-env-injection, static-inline-injection, static-unsanitized-env-write

**Notes:**

Fixed all findings in action.yml:

1. script-injection / static-inline-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks for three steps:
   - 'Check system dependencies': inputs.skip_validation → INPUT_SKIP_VALIDATION env var
   - 'Set safe directory': github.workspace → GITHUB_WORKSPACE_PATH env var
   - 'Get and set token': inputs.use_oidc → INPUT_USE_OIDC, env.CODECOV_TOKEN → INPUT_CODECOV_TOKEN, inputs.token → INPUT_TOKEN

2. github-env-injection / static-unsanitized-env-write: Added printf '%s' ... | tr -d '\n\r' sanitization before all GITHUB_ENV writes in:
   - 'Get and set token': all three token paths (OIDC, env token, input token) now sanitized
   - 'Override branch for forks': TOKENLESS and CC_BRANCH now sanitized
   - 'Override commits and pr for pull requests': CC_SHA and CC_PR now sanitized

The 'Set fork' step's CC_FORK write was not flagged (it's a locally-computed 'true'/'false' value, not from external context) and was left as-is.

