<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v6.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v6.0.0** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Direct ${{ }} expression interpolation inside run: shell commands. In the 'Check system dependencies' step, `${{ inputs.skip_validation }}` is interpolated directly into a shell conditional: `if [ "${{ inputs.skip_validation }}" != "true" ]`. In the 'Set safe directory' step, `${{ github.workspace }}` is interpolated directly into a git config command: `git config --global --add safe.directory "${{ github.workspace }}"`. In the 'Get and set token' step, multiple expressions are interpolated directly into shell: `if [ "${{ inputs.use_oidc }}" == 'true' ]`, `elif [ -n "${{ env.CODECOV_TOKEN }}" ]`, `echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV"`, and `CC_TOKEN=$(echo "${{ inputs.token }}" | tr -d '\n')`. All of these allow an attacker-controlled value to be injected into the shell command string before the shell parses it, enabling command injection.

Locations:

- `action.yml:167`
- `action.yml:176`
- `action.yml:198`
- `action.yml:201`
- `action.yml:203`
- `action.yml:206`

### github-env-injection (severity: high)

Unsanitized values derived from untrusted inputs are written to $GITHUB_ENV without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). In the 'Override branch for forks' step: `TOKENLESS` (set from `$GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL`, which is `github.event.pull_request.head.label`) and `CC_BRANCH` (set from `inputs.override_branch` via env var) are written with `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"` without sanitization. In the 'Override commits and pr' step: `CC_SHA` (from `inputs.override_commit`) and `CC_PR` (from `inputs.override_pr`) are written with `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` and `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"` without sanitization. An attacker can inject newlines into these values to set arbitrary environment variables for subsequent steps.

Locations:

- `action.yml:215`
- `action.yml:218`
- `action.yml:230`
- `action.yml:231`

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

Fixed all script-injection findings by moving ${{ }} expressions from run: blocks to env: blocks and referencing them as plain environment variables. Fixed all github-env-injection and static-unsanitized-env-write findings by sanitizing values with `printf '%s' "$VAR" | tr -d '\n\r'` before writing to $GITHUB_ENV. Specific changes: (1) 'Check system dependencies': moved inputs.skip_validation to env as INPUT_SKIP_VALIDATION; (2) 'Set safe directory': replaced ${{ github.workspace }} with $GITHUB_WORKSPACE (standard env var); (3) 'Get and set token': moved inputs.use_oidc, env.CODECOV_TOKEN, and inputs.token to env block, sanitized all GITHUB_ENV writes; (4) 'Override branch for forks': sanitized TOKENLESS and CC_BRANCH before writing to GITHUB_ENV; (5) 'Override commits and pr for pull requests': sanitized CC_SHA and CC_PR before writing to GITHUB_ENV.

