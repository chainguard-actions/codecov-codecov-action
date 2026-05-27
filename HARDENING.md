# Hardening Report: codecov--codecov-action/v6.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **codecov--codecov-action/v6.0.0** was hardened automatically. 14 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Check system dependencies' run: block directly interpolates ${{ inputs.skip_validation }} into the shell command string. An attacker-controlled input value is embedded directly in the shell script rather than being passed through an env: variable, enabling script injection.

Locations:

- `action.yml:175`

### script-injection (severity: high)

The 'Set safe directory' run: block directly interpolates ${{ github.workspace }} into the shell command string (git config --global --add safe.directory "${{ github.workspace }}"). The github.workspace context value should be passed via an env: variable instead.

Locations:

- `action.yml:196`

### script-injection (severity: high)

The 'Get and set token' run: block directly interpolates multiple attacker-controlled expressions into shell commands: ${{ inputs.use_oidc }} (used in an if condition), ${{ env.CODECOV_TOKEN }} (used in an if condition and written to GITHUB_ENV), and ${{ inputs.token }} (used in an if condition and in a command substitution). These should be passed via env: variables.

Locations:

- `action.yml:222`

### github-env-injection (severity: high)

The 'Get and set token' step writes attacker-controlled values to $GITHUB_ENV without proper sanitization. Specifically: (1) echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV" writes the CODECOV_TOKEN env context directly with no sanitization; (2) CC_TOKEN=$(echo "${{ inputs.token }}" | tr -d '\n') followed by echo "CC_TOKEN=$CC_TOKEN" >> "$GITHUB_ENV" strips only newlines but not carriage returns, and does not use the required printf '%s' ... | tr -d '\n\r' sanitization pipeline.

Locations:

- `action.yml:222`

### github-env-injection (severity: high)

The 'Override branch for forks' step writes attacker-controlled values to $GITHUB_ENV without sanitization. The env var GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL is set from ${{ github.event.pull_request.head.label }} (attacker-controlled via PR), then used unsanitized in: echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV" and echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV". Routing through an env: variable does not sanitize the value.

Locations:

- `action.yml:248`

### github-env-injection (severity: high)

The 'Override commits and pr for pull requests' step writes attacker-controlled values to $GITHUB_ENV without sanitization. CC_SHA (from ${{ inputs.override_commit }} / ${{ github.event.pull_request.head.sha }}) and CC_PR (from ${{ inputs.override_pr }} / ${{ github.event.number }}) are written via echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV" and echo "CC_PR=$CC_PR" >> "$GITHUB_ENV" with no sanitization pipeline applied.

Locations:

- `action.yml:265`

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

Fixed all security findings in action.yml:

1. 'Check system dependencies' step: Moved ${{ inputs.skip_validation }} to env: variable SKIP_VALIDATION, replaced inline expression with $SKIP_VALIDATION in shell.

2. 'Set safe directory' step: Moved ${{ github.workspace }} to env: variable ACTION_WORKSPACE, replaced inline expression with $ACTION_WORKSPACE in shell.

3. 'Get and set token' step: Moved ${{ inputs.use_oidc }}, ${{ env.CODECOV_TOKEN }}, and ${{ inputs.token }} to env: variables (INPUT_USE_OIDC, CODECOV_TOKEN, INPUT_TOKEN). All three GITHUB_ENV writes now use 'printf "%s" "$VAR" | tr -d "\n\r"' sanitization pipeline.

4. 'Override branch for forks' step: Both GITHUB_ENV writes (TOKENLESS and CC_BRANCH) now use printf/tr sanitization.

5. 'Override commits and pr for pull requests' step: Both GITHUB_ENV writes (CC_SHA and CC_PR) now use printf/tr sanitization.

