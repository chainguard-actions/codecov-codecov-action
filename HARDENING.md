<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v7.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v7.0.0** was hardened automatically. 9 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The 'Set safe directory' run block directly interpolates `${{ github.workspace }}` inside the shell command string. Before the shell executes the command, GitHub Actions performs template substitution, so an attacker who can influence the workspace path could inject shell metacharacters. The offending line is: `git config --global --add safe.directory "${{ github.workspace }}"`

Locations:

- `action.yml:196`

### github-env-injection (severity: high)

The 'Get and set token' step writes inherited/workflow-controlled env vars to $GITHUB_ENV without sanitization. Specifically: (1) `echo "CC_TOKEN=$CC_OIDC_TOKEN" >> "$GITHUB_ENV"` — CC_OIDC_TOKEN comes from `steps.oidc.outputs.result`, a workflow-controlled value; (2) `echo "CC_TOKEN=$INPUT_CODECOV_TOKEN" >> "$GITHUB_ENV"` — INPUT_CODECOV_TOKEN comes from `env.CODECOV_TOKEN`, an inherited process env var set by the calling workflow. Neither write is preceded by the required `printf '%s' ... | tr -d '\n\r'` sanitization step.

Locations:

- `action.yml:218`

### github-env-injection (severity: high)

The 'Override branch for forks' step writes attacker-controllable values to $GITHUB_ENV without sanitization. The env var GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL is sourced from `${{ github.event.pull_request.head.label }}` (attacker-controlled via PR). The script writes `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"` where both variables derive from GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before either write.

Locations:

- `action.yml:243`

### github-env-injection (severity: high)

The 'Override commits and pr for pull requests' step writes attacker-controllable values to $GITHUB_ENV without sanitization. CC_SHA derives from GITHUB_EVENT_PULL_REQUEST_HEAD_SHA (sourced from `${{ github.event.pull_request.head.sha }}`), and CC_PR derives from GITHUB_EVENT_NUMBER (sourced from `${{ github.event.number }}`). Both are written via `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` and `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"` without the required `printf '%s' ... | tr -d '\n\r'` sanitization.

Locations:

- `action.yml:264`

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

Fixed 4 categories of findings in hardened/action/action.yml:
1. script-injection: Moved `${{ github.workspace }}` from the 'Set safe directory' run block into an env var (WORKSPACE_PATH) to prevent shell metacharacter injection.
2. github-env-injection + static-unsanitized-env-write in 'Get and set token': Added `printf '%s' ... | tr -d '\n\r'` sanitization for CC_OIDC_TOKEN and INPUT_CODECOV_TOKEN before writing to $GITHUB_ENV.
3. github-env-injection + static-unsanitized-env-write in 'Override branch for forks': Added sanitization for TOKENLESS and CC_BRANCH before writing to $GITHUB_ENV.
4. github-env-injection + static-unsanitized-env-write in 'Override commits and pr for pull requests': Added sanitization for CC_SHA and CC_PR before writing to $GITHUB_ENV.

