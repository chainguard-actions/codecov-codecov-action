<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v6.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **codecov--codecov-action/v6.0.1** was hardened automatically. 9 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Set safe directory' step directly interpolates `${{ github.workspace }}` inside a `run:` shell command string. The expression is substituted by the YAML template engine before the shell executes it, allowing an attacker who controls the workspace path to inject shell metacharacters. Offending line: git config --global --add safe.directory "${{ github.workspace }}"

Locations:

- `action.yml:192`

### github-env-injection (severity: high)

The 'Get and set token' step writes CC_OIDC_TOKEN (sourced from steps.oidc.outputs.result) and INPUT_CODECOV_TOKEN (sourced from env.CODECOV_TOKEN) to $GITHUB_ENV without the required printf '%s' ... | tr -d '\n\r' sanitization. An attacker who can influence these values could inject arbitrary environment variable definitions via newline characters. Offending lines: echo "CC_TOKEN=$CC_OIDC_TOKEN" >> "$GITHUB_ENV" and echo "CC_TOKEN=$INPUT_CODECOV_TOKEN" >> "$GITHUB_ENV"

Locations:

- `action.yml:222`
- `action.yml:226`

### github-env-injection (severity: high)

The 'Override branch for forks' step writes TOKENLESS and CC_BRANCH to $GITHUB_ENV without sanitization. Both variables are derived from github.event.pull_request.head.label (an attacker-controlled PR head label) and inputs.override_branch, passed through env vars but never sanitized with tr -d '\n\r' before the write. Offending lines: echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV" and echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"

Locations:

- `action.yml:248`
- `action.yml:251`

### github-env-injection (severity: high)

The 'Override commits and pr for pull requests' step writes CC_SHA (sourced from inputs.override_commit / github.event.pull_request.head.sha) and CC_PR (sourced from inputs.override_pr / github.event.number) to $GITHUB_ENV without the required printf '%s' ... | tr -d '\n\r' sanitization. Offending lines: echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV" and echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"

Locations:

- `action.yml:267`
- `action.yml:268`

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

Fixed all 9 findings in action.yml:
1. script-injection (line 192): Moved `${{ github.workspace }}` from the run: shell string into an env: block as WORKSPACE_PATH, referenced as "$WORKSPACE_PATH" in the shell.
2. github-env-injection + static-unsanitized-env-write in 'Get and set token' step: Added `safe_token=$(printf '%s' "$CC_OIDC_TOKEN" | tr -d '\n\r')` and `safe_token=$(printf '%s' "$INPUT_CODECOV_TOKEN" | tr -d '\n\r')` before writing to $GITHUB_ENV.
3. github-env-injection + static-unsanitized-env-write in 'Override branch for forks' step: Added sanitization for TOKENLESS and CC_BRANCH using printf/tr before writing to $GITHUB_ENV.
4. github-env-injection + static-unsanitized-env-write in 'Override commits and pr for pull requests' step: Added sanitization for CC_SHA and CC_PR using printf/tr before writing to $GITHUB_ENV.

