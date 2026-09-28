<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v6.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v6.0.0** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `run:` blocks in action.yml directly interpolate `${{ ... }}` expressions into shell commands, enabling script injection.

1. 'Check system dependencies' step: `if [ "${{ inputs.skip_validation }}" != "true" ]` — the inputs.skip_validation value is interpolated directly into the shell condition before the shell parses it.

2. 'Set safe directory' step: `git config --global --add safe.directory "${{ github.workspace }}"` — github.workspace is interpolated directly into the shell command.

3. 'Get and set token' step contains multiple direct interpolations:
   - `if [ "${{ inputs.use_oidc }}" == 'true' ]`
   - `elif [ -n "${{ env.CODECOV_TOKEN }}" ]`
   - `echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV"`
   - `if [ -n "${{ inputs.token }}" ]`
   - `CC_TOKEN=$(echo "${{ inputs.token }}" | tr -d '\n')`

All of these allow an attacker-controlled value to be injected into the shell command string before the shell interprets it, enabling arbitrary command execution.

Locations:

- `action.yml:168`
- `action.yml:183`
- `action.yml:207`
- `action.yml:211`
- `action.yml:213`
- `action.yml:215`
- `action.yml:217`

### github-env-injection (severity: high)

Multiple `run:` steps write values derived from untrusted inputs to `$GITHUB_ENV` without the required sanitization (`printf '%s' ... | tr -d '\n\r'`), allowing newline injection to set arbitrary environment variables for subsequent steps.

1. 'Get and set token' step: `echo "CC_TOKEN=$CC_OIDC_TOKEN" >> "$GITHUB_ENV"` — CC_OIDC_TOKEN is set from `${{ steps.oidc.outputs.result }}` (a workflow-controlled value) with no sanitization.

2. 'Get and set token' step: `echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV"` — directly writes the env.CODECOV_TOKEN expression (workflow-controlled) to GITHUB_ENV without sanitization.

3. 'Override branch for forks' step: `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"` — both TOKENLESS and CC_BRANCH are derived from GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL (= `${{ github.event.pull_request.head.label }}`), which is attacker-controlled on PRs, with no sanitization before the write.

4. 'Override commits and pr for pull requests' step: `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` and `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"` — CC_SHA and CC_PR are sourced from `inputs.override_commit`, `inputs.override_pr`, `github.event.pull_request.head.sha`, and `github.event.number` (all workflow-controllable), with no sanitization.

Locations:

- `action.yml:209`
- `action.yml:213`
- `action.yml:231`
- `action.yml:234`
- `action.yml:249`
- `action.yml:250`

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

Fixed all security findings in hardened/action/action.yml:

1. script-injection / static-inline-injection (Check system dependencies): Moved `${{ inputs.skip_validation }}` to env block as SKIP_VALIDATION; shell now references $SKIP_VALIDATION.

2. script-injection (Set safe directory): Replaced `${{ github.workspace }}` with `$GITHUB_WORKSPACE` (standard GitHub Actions env var, always available, no injection risk).

3. script-injection / static-inline-injection (Get and set token): Moved all inline expressions to env block: inputs.use_oidc→USE_OIDC, env.CODECOV_TOKEN→CODECOV_TOKEN_ENV, inputs.token→CODECOV_TOKEN_INPUT. Shell script now references only env vars.

4. github-env-injection / static-unsanitized-env-write (Get and set token): All three token paths (OIDC, env, input) now sanitize with `printf '%s' "$VAR" | tr -d '\n\r'` before writing to GITHUB_ENV.

5. github-env-injection / static-unsanitized-env-write (Override branch for forks): TOKENLESS and CC_BRANCH writes to GITHUB_ENV now sanitized with printf/tr.

6. github-env-injection / static-unsanitized-env-write (Override commits and pr for pull requests): CC_SHA and CC_PR writes to GITHUB_ENV now sanitized with printf/tr.

