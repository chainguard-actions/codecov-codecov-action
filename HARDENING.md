# Hardening Report: codecov--codecov-action/v6.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **codecov--codecov-action/v6.0.1** was hardened automatically. 8 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Set safe directory' step directly interpolates `${{ github.workspace }}` inside a `run:` shell command: `git config --global --add safe.directory "${{ github.workspace }}"`. The github.workspace context value is embedded directly in the shell string rather than being assigned to an environment variable first. This constitutes script injection.

Locations:

- `action.yml:204`

### github-env-injection (severity: high)

The 'Override branch for forks' step writes attacker-controlled values to $GITHUB_ENV without sanitization. The variable TOKENLESS and CC_BRANCH are derived from GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL (mapped from `github.event.pull_request.head.label`, an attacker-controlled PR value), and written via `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"` without the required `printf '%s' ... | tr -d '\n\r'` sanitization step. A newline injection in the label value could allow an attacker to inject arbitrary environment variables.

Locations:

- `action.yml:268`
- `action.yml:271`

### github-env-injection (severity: high)

The 'Override commits and pr for pull requests' step writes attacker-controlled values to $GITHUB_ENV without sanitization. CC_SHA is derived from `inputs.override_commit` or `github.event.pull_request.head.sha`, and CC_PR is derived from `inputs.override_pr` or `github.event.number`. Both are written via `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` and `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"` without the required `printf '%s' ... | tr -d '\n\r'` sanitization step.

Locations:

- `action.yml:291`
- `action.yml:292`

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

Fixed all 8 findings in action.yml:
1. script-injection (line 204): Moved `${{ github.workspace }}` from the run shell string into an env var `WORKSPACE_PATH`, referenced as `$WORKSPACE_PATH` in the shell script.
2. static-unsanitized-env-write (CC_OIDC_TOKEN, line 252): Added `safe_oidc=$(printf '%s' "$CC_OIDC_TOKEN" | tr -d '\n\r')` before writing to GITHUB_ENV.
3. static-unsanitized-env-write (INPUT_CODECOV_TOKEN, line 256): Added `safe_codecov_token=$(printf '%s' "$INPUT_CODECOV_TOKEN" | tr -d '\n\r')` before writing to GITHUB_ENV.
4. github-env-injection (TOKENLESS, line 268): Added `safe_tokenless=$(printf '%s' "$TOKENLESS" | tr -d '\n\r')` before writing to GITHUB_ENV.
5. github-env-injection + static-unsanitized-env-write (CC_BRANCH, lines 271/283): Added `safe_branch=$(printf '%s' "$CC_BRANCH" | tr -d '\n\r')` before writing to GITHUB_ENV.
6. github-env-injection + static-unsanitized-env-write (CC_SHA, CC_PR, lines 291/292/302/303): Added `safe_sha` and `safe_pr` with `printf '%s' ... | tr -d '\n\r'` sanitization before writing to GITHUB_ENV.

