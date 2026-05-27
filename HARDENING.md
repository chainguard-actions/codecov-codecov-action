# Hardening Report: codecov--codecov-action/v6.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **codecov--codecov-action/v6.0.1** was hardened automatically. 9 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Set safe directory' step directly interpolates `${{ github.workspace }}` inside the `run:` shell command string. Attacker-controlled github.* expressions must be passed through an `env:` variable and referenced as `$ENV_VAR` in the run block, not interpolated directly. The line reads: `git config --global --add safe.directory "${{ github.workspace }}"`

Locations:

- `action.yml:213`

### github-env-injection (severity: high)

The 'Override branch for forks' step writes attacker-controlled values to $GITHUB_ENV without sanitization. The env vars TOKENLESS and CC_BRANCH are derived from `github.event.pull_request.head.label` (via GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL) and `inputs.override_branch` (via CC_BRANCH), and are echoed directly to $GITHUB_ENV with no `printf '%s' ... | tr -d '\n\r'` sanitization step. A newline injection in these values could allow an attacker to set arbitrary environment variables.

Locations:

- `action.yml:282`
- `action.yml:285`

### github-env-injection (severity: high)

The 'Override commits and pr for pull requests' step writes attacker-controlled values to $GITHUB_ENV without sanitization. CC_SHA is derived from `inputs.override_commit` and `github.event.pull_request.head.sha`; CC_PR is derived from `inputs.override_pr` and `github.event.number`. Both are echoed directly to $GITHUB_ENV with no `printf '%s' ... | tr -d '\n\r'` sanitization, enabling newline injection to set arbitrary environment variables.

Locations:

- `action.yml:303`
- `action.yml:304`

### github-env-injection (severity: high)

The 'Get and set token' step writes CC_TOKEN to $GITHUB_ENV from two unsanitized sources: (1) `echo "CC_TOKEN=$CC_OIDC_TOKEN" >> "$GITHUB_ENV"` where CC_OIDC_TOKEN comes from `steps.oidc.outputs.result`, and (2) `echo "CC_TOKEN=$INPUT_CODECOV_TOKEN" >> "$GITHUB_ENV"` where INPUT_CODECOV_TOKEN comes from `env.CODECOV_TOKEN`. Neither write applies the required `printf '%s' ... | tr -d '\n\r'` sanitization before writing to $GITHUB_ENV. (Note: the INPUT_TOKEN path does apply `tr -d '\n'` and is safe.)

Locations:

- `action.yml:258`
- `action.yml:261`

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
1. script-injection (line 213): Moved `${{ github.workspace }}` into an `env:` block as `WORKSPACE` and referenced it as `$WORKSPACE` in the run block.
2. github-env-injection + static-unsanitized-env-write (Get and set token step): Added `safe_oidc=$(printf '%s' "$CC_OIDC_TOKEN" | tr -d '\n\r')` and `safe_codecov=$(printf '%s' "$INPUT_CODECOV_TOKEN" | tr -d '\n\r')` before writing to GITHUB_ENV.
3. github-env-injection + static-unsanitized-env-write (Override branch for forks step): Added sanitization for TOKENLESS and CC_BRANCH using printf/tr before writing to GITHUB_ENV.
4. github-env-injection + static-unsanitized-env-write (Override commits and pr for pull requests step): Added sanitization for CC_SHA and CC_PR using printf/tr before writing to GITHUB_ENV.

