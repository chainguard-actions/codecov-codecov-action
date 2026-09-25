<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v6.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v6.0.0** was hardened automatically. 14 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Check system dependencies' run: block directly interpolates ${{ inputs.skip_validation }} into a shell command string: `if [ "${{ inputs.skip_validation }}" != "true" ]`. This allows an attacker-controlled value to be injected into the shell before quoting takes effect.

Locations:

- `action.yml:160`

### script-injection (severity: high)

Sub-rule (a): The 'Set safe directory' run: block directly interpolates ${{ github.workspace }} into a shell command string: `git config --global --add safe.directory "${{ github.workspace }}"`  Any expression inside a run: block is a script-injection risk regardless of context.

Locations:

- `action.yml:176`

### script-injection (severity: high)

Sub-rule (a): The 'Get and set token' run: block directly interpolates multiple ${{ }} expressions into shell command strings: `if [ "${{ inputs.use_oidc }}" == 'true' ]`, `elif [ -n "${{ env.CODECOV_TOKEN }}" ]`, `echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV"`, `if [ -n "${{ inputs.token }}" ]`, and `CC_TOKEN=$(echo "${{ inputs.token }}" | tr -d '\n')`. Attacker-controlled values from inputs.* and env.* are interpolated directly into shell before the shell parses them.

Locations:

- `action.yml:205`

### github-env-injection (severity: high)

The 'Get and set token' step writes ${{ env.CODECOV_TOKEN }} directly to $GITHUB_ENV without sanitization: `echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV"`. The env.CODECOV_TOKEN value is workflow-controlled and can contain newlines that inject additional environment variables. The required sanitization step (printf '%s' ... | tr -d '\n\r') is absent.

Locations:

- `action.yml:210`

### github-env-injection (severity: high)

The 'Override branch for forks' step writes unsanitized values to $GITHUB_ENV. The variables TOKENLESS and CC_BRANCH are derived from GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL (set from github.event.pull_request.head.label, an attacker-controlled value on PRs) and written without sanitization: `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"`. No printf '%s' | tr -d '\n\r' sanitization is applied.

Locations:

- `action.yml:228`

### github-env-injection (severity: high)

The 'Override commits and pr for pull requests' step writes unsanitized values to $GITHUB_ENV. CC_SHA comes from GITHUB_EVENT_PULL_REQUEST_HEAD_SHA (set from github.event.pull_request.head.sha) and CC_PR from GITHUB_EVENT_NUMBER (set from github.event.number). Both are written without sanitization: `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` and `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"`. No printf '%s' | tr -d '\n\r' sanitization is applied.

Locations:

- `action.yml:248`

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

Fixed all 14 findings in hardened/action/action.yml:

1. 'Check system dependencies' step: Moved ${{ inputs.skip_validation }} to env block as SKIP_VALIDATION; referenced as $SKIP_VALIDATION in shell.

2. 'Set safe directory' step: Moved ${{ github.workspace }} to env block as GIT_WORKSPACE; referenced as $GIT_WORKSPACE in shell.

3. 'Get and set token' step: Moved ${{ inputs.use_oidc }}, ${{ env.CODECOV_TOKEN }}, and ${{ inputs.token }} to env block as INPUT_USE_OIDC, INPUT_CODECOV_TOKEN, INPUT_TOKEN. All GITHUB_ENV writes now use printf '%s' | tr -d '\n\r' sanitization (for CC_OIDC_TOKEN, CODECOV_TOKEN, and inputs.token paths).

4. 'Override branch for forks' step: Added printf/tr sanitization for TOKENLESS and CC_BRANCH before writing to GITHUB_ENV.

5. 'Override commits and pr for pull requests' step: Added printf/tr sanitization for CC_SHA and CC_PR before writing to GITHUB_ENV. Moved env block before run block to fix YAML structure (GITHUB_EVENT_NAME, GITHUB_EVENT_NUMBER, GITHUB_EVENT_PULL_REQUEST_HEAD_SHA were originally after the run block).

