# Hardening Report: codecov--codecov-action/v6.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **codecov--codecov-action/v6.0.0** was hardened automatically. 14 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Check system dependencies' run block directly interpolates ${{ inputs.skip_validation }} inside a shell `if` condition. Attacker-controlled input values should be passed via env: variables, not interpolated directly into run: scripts.

Locations:

- `action.yml:165`

### script-injection (severity: high)

The 'Set safe directory' run block directly interpolates ${{ github.workspace }} inside a `git config` shell command. GitHub context values should be passed via env: variables, not interpolated directly into run: scripts.

Locations:

- `action.yml:186`

### script-injection (severity: high)

The 'Get and set token' run block directly interpolates ${{ inputs.use_oidc }}, ${{ env.CODECOV_TOKEN }}, and ${{ inputs.token }} inside shell conditionals and echo commands. Attacker-controlled values should be passed via env: variables, not interpolated directly into run: scripts.

Locations:

- `action.yml:225`

### github-env-injection (severity: high)

The 'Get and set token' step writes ${{ env.CODECOV_TOKEN }} directly to $GITHUB_ENV without any sanitization (no `printf '%s' | tr -d '\n\r'` applied). An attacker who can set CODECOV_TOKEN to a value containing newlines could inject arbitrary environment variables. Additionally, ${{ inputs.token }} is only partially sanitized with `tr -d '\n'` (missing `\r` stripping and the required `printf '%s'` wrapper).

Locations:

- `action.yml:231`

### github-env-injection (severity: high)

The 'Override branch for forks' step writes $CC_BRANCH (sourced from inputs.override_branch via env:) and $GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL (sourced from github.event.pull_request.head.label via env:) to $GITHUB_ENV without sanitization. An attacker-controlled value containing newlines could inject arbitrary environment variables.

Locations:

- `action.yml:255`

### github-env-injection (severity: high)

The 'Override commits and pr for pull requests' step writes $CC_SHA (sourced from inputs.override_commit via env:) and $CC_PR (sourced from inputs.override_pr via env:) to $GITHUB_ENV without sanitization (no `printf '%s' | tr -d '\n\r'` applied). Attacker-controlled values containing newlines could inject arbitrary environment variables.

Locations:

- `action.yml:270`

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

Fixed all 14 findings in action.yml:
1. 'Check system dependencies': moved ${{ inputs.skip_validation }} to CC_SKIP_VALIDATION_INPUT env var to prevent shell injection.
2. 'Set safe directory': moved ${{ github.workspace }} to CC_GITHUB_WORKSPACE env var to prevent shell injection.
3. 'Get and set token': moved ${{ inputs.use_oidc }}, ${{ env.CODECOV_TOKEN }}, and ${{ inputs.token }} to env vars (CC_USE_OIDC_INPUT, CC_CODECOV_TOKEN_ENV, CC_TOKEN_INPUT); all GITHUB_ENV writes now sanitized with printf '%s' "$VAR" | tr -d '\n\r'.
4. 'Override branch for forks': CC_BRANCH and TOKENLESS writes to GITHUB_ENV now sanitized with printf/tr.
5. 'Override commits and pr for pull requests': CC_SHA and CC_PR writes to GITHUB_ENV now sanitized with printf/tr.

