<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v7.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v7.0.0** was hardened automatically. 9 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Set safe directory' step directly interpolates `${{ github.workspace }}` inside a `run:` shell command string: `git config --global --add safe.directory "${{ github.workspace }}"`. Any `${{ ... }}` expression interpolated directly into a run: block is a script injection risk because the value is substituted into the shell command before the shell parses it, allowing an attacker who can influence the value to inject arbitrary shell commands.

Locations:

- `action.yml:175`

### github-env-injection (severity: high)

The 'Get and set token' step writes untrusted values to $GITHUB_ENV without sanitization. (1) `echo "CC_TOKEN=$CC_OIDC_TOKEN" >> "$GITHUB_ENV"` — CC_OIDC_TOKEN is sourced from `steps.oidc.outputs.result` (an untrusted step output). (2) `echo "CC_TOKEN=$INPUT_CODECOV_TOKEN" >> "$GITHUB_ENV"` — INPUT_CODECOV_TOKEN is sourced from `env.CODECOV_TOKEN`, a workflow-controlled environment variable. Neither write is preceded by the required `printf '%s' ... | tr -d '\n\r'` sanitization, allowing newline injection into GITHUB_ENV.

Locations:

- `action.yml:203`
- `action.yml:207`

### github-env-injection (severity: high)

The 'Override branch for forks' step writes attacker-controlled values to $GITHUB_ENV without sanitization. (1) `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and (2) `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"` — both TOKENLESS and CC_BRANCH are derived from the env var GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL, which is set from `${{ github.event.pull_request.head.label }}` — an attacker-controlled value on pull requests. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before writing to GITHUB_ENV.

Locations:

- `action.yml:228`
- `action.yml:230`

### github-env-injection (severity: high)

The 'Override commits and pr for pull requests' step writes user-controlled input values to $GITHUB_ENV without sanitization. (1) `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` — CC_SHA is sourced from `inputs.override_commit` (user-supplied input). (2) `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"` — CC_PR is sourced from `inputs.override_pr` (user-supplied input). Neither write is preceded by the required `printf '%s' ... | tr -d '\n\r'` sanitization, allowing newline injection into GITHUB_ENV.

Locations:

- `action.yml:247`
- `action.yml:248`

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

Fixed all 9 findings in hardened/action/action.yml:
1. script-injection (line 175): Moved `${{ github.workspace }}` into an env var `WORKSPACE_PATH` and referenced it as `"$WORKSPACE_PATH"` in the shell script.
2. github-env-injection + static-unsanitized-env-write in 'Get and set token' step: Added `printf '%s' "$CC_OIDC_TOKEN" | tr -d '\n\r'` and `printf '%s' "$INPUT_CODECOV_TOKEN" | tr -d '\n\r'` sanitization before writing to GITHUB_ENV.
3. github-env-injection + static-unsanitized-env-write in 'Override branch for forks' step: Added `printf '%s' "$TOKENLESS" | tr -d '\n\r'` and `printf '%s' "$CC_BRANCH" | tr -d '\n\r'` sanitization before writing to GITHUB_ENV.
4. github-env-injection + static-unsanitized-env-write in 'Override commits and pr for pull requests' step: Added `printf '%s' "$CC_SHA" | tr -d '\n\r'` and `printf '%s' "$CC_PR" | tr -d '\n\r'` sanitization before writing to GITHUB_ENV.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Get and set token' step in action.yml: changed the INPUT_TOKEN sanitization from `echo "$INPUT_TOKEN" | tr -d '\n'` to `printf '%s' "$INPUT_TOKEN" | tr -d '\n\r'`. This ensures carriage returns are also stripped before writing to $GITHUB_ENV, preventing potential injection of additional key=value pairs. All three token paths (OIDC, CODECOV_TOKEN env var, and INPUT_TOKEN) now consistently use `printf '%s' "$VAR" | tr -d '\n\r'` for sanitization.

