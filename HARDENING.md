<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v5.5.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v5.5.5** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Multiple `${{ }}` expressions are interpolated directly inside `run:` shell command strings in action.yml, allowing script injection.

1. "Check system dependencies" step: `if [ "${{ inputs.skip_validation }}" != "true" ]` — the input value is substituted directly into the shell command before the shell parses it.

2. "Set safe directory" step: `git config --global --add safe.directory "${{ github.workspace }}"` — the github context value is interpolated directly into the shell command.

3. "Get and set token" step (multiple lines):
   - `if [ "${{ inputs.use_oidc }}" == 'true' ]`
   - `elif [ -n "${{ env.CODECOV_TOKEN }}" ]`
   - `echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV"`
   - `if [ -n "${{ inputs.token }}" ]`
   - `CC_TOKEN=$(echo "${{ inputs.token }}" | tr -d '\n')`

All of these allow an attacker-controlled value to be injected into the shell command string before the shell evaluates it. The correct fix is to pass values via `env:` variables and reference them as `"$VAR"` in the shell.

Locations:

- `action.yml:168`
- `action.yml:183`
- `action.yml:207`
- `action.yml:210`
- `action.yml:213`
- `action.yml:216`
- `action.yml:219`

### github-env-injection (severity: high)

Multiple `run:` steps write untrusted input values to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

1. "Get and set token" step: `echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV"` — the `env.CODECOV_TOKEN` context value (workflow-controlled) is written directly to GITHUB_ENV without newline sanitization, enabling environment variable injection.

2. "Override branch for forks" step: `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"` — both variables are sourced from `github.event.pull_request.head.label` (attacker-controlled via a PR), passed through `env:` without sanitization before being written to GITHUB_ENV.

3. "Override commits and pr for pull requests" step: `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` and `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"` — `CC_SHA` is sourced from `inputs.override_commit` and `CC_PR` from `inputs.override_pr`, both written to GITHUB_ENV without sanitization.

An attacker can inject newlines into these values to set arbitrary environment variables for subsequent steps.

Locations:

- `action.yml:213`
- `action.yml:233`
- `action.yml:237`
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

Fixed all script injection and GITHUB_ENV injection findings in hardened/action/action.yml:

1. 'Check system dependencies' step: Added env block with SKIP_VALIDATION=${{ inputs.skip_validation }} and replaced the inline ${{ inputs.skip_validation }} expression in the shell with $SKIP_VALIDATION.

2. 'Set safe directory' step: Added env block with GIT_WORKSPACE=${{ github.workspace }} and replaced the inline ${{ github.workspace }} expression with $GIT_WORKSPACE.

3. 'Get and set token' step: Moved inputs.use_oidc, inputs.token, and env.CODECOV_TOKEN to the env: block as INPUT_USE_OIDC, INPUT_TOKEN, and CODECOV_TOKEN respectively. Updated all shell references to use env vars. Added printf '%s' ... | tr -d '\n\r' sanitization before all GITHUB_ENV writes (CC_OIDC_TOKEN, CODECOV_TOKEN, and INPUT_TOKEN paths).

4. 'Override branch for forks' step: Added printf '%s' ... | tr -d '\n\r' sanitization before writing TOKENLESS and CC_BRANCH to GITHUB_ENV.

5. 'Override commits and pr for pull requests' step: Added printf '%s' ... | tr -d '\n\r' sanitization before writing CC_SHA and CC_PR to GITHUB_ENV.

