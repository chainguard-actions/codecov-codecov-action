# Hardening Report: codecov--codecov-action/v7.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **codecov--codecov-action/v7.0.0** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Set safe directory' step directly interpolates `${{ github.workspace }}` inside a `run:` shell command: `git config --global --add safe.directory "${{ github.workspace }}"`.

This is a direct expression interpolation in a run block. An attacker who can influence `github.workspace` could inject arbitrary shell commands. The value should be passed via an `env:` variable and referenced as `$ENV_VAR` in the shell script instead.

Locations:

- `action.yml:171`

### github-env-injection (severity: high)

Multiple steps write attacker-controlled values (from `github.*` and `inputs.*` expressions) to `$GITHUB_ENV` via env vars without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). An attacker can inject newlines to add arbitrary environment variables.

1. 'Set fork' step: writes `CC_FORK` (derived from `github.event.pull_request.head.repo.full_name` / `github.repository`) to `$GITHUB_ENV` with `echo "CC_FORK=$CC_FORK" >> "$GITHUB_ENV"` — no sanitization.

2. 'Get and set token' step: writes `CC_TOKEN=$CC_OIDC_TOKEN` and `CC_TOKEN=$INPUT_CODECOV_TOKEN` to `$GITHUB_ENV` without sanitization (only the `$INPUT_TOKEN` path applies `tr -d '\n'`, but the OIDC and CODECOV_TOKEN paths do not).

3. 'Override branch for forks' step: writes `TOKENLESS` and `CC_BRANCH` (derived from `github.event.pull_request.head.label` and `inputs.override_branch`) to `$GITHUB_ENV` without sanitization.

4. 'Override commits and pr for pull requests' step: writes `CC_SHA` and `CC_PR` (derived from `inputs.override_commit`, `inputs.override_pr`, `github.event.pull_request.head.sha`, `github.event.number`) to `$GITHUB_ENV` without sanitization.

Locations:

- `action.yml:181`
- `action.yml:196`
- `action.yml:200`
- `action.yml:219`
- `action.yml:222`
- `action.yml:235`
- `action.yml:236`

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

Fixed all 7 findings in action.yml:
1. script-injection (line 171): Moved ${{ github.workspace }} from run: block to env: block as WORKSPACE_PATH, referenced as $WORKSPACE_PATH in shell.
2. github-env-injection + static-unsanitized-env-write: Added printf '%s' ... | tr -d '\n\r' sanitization before all writes to $GITHUB_ENV in four steps: 'Set fork' (CC_FORK), 'Get and set token' (CC_OIDC_TOKEN and INPUT_CODECOV_TOKEN paths), 'Override branch for forks' (TOKENLESS and CC_BRANCH), and 'Override commits and pr for pull requests' (CC_SHA and CC_PR).

