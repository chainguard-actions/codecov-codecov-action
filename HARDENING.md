<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v7.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **codecov--codecov-action/v7.0.0** was hardened automatically. 9 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Set safe directory' step directly interpolates `${{ github.workspace }}` inside a `run:` shell command string: `git config --global --add safe.directory "${{ github.workspace }}"`. Any GitHub Actions expression inside a run: block is a script-injection risk because the value is substituted into the shell command before the shell parses it.

Locations:

- `action.yml:175`

### github-env-injection (severity: high)

The 'Get and set token' step writes `CC_TOKEN=$CC_OIDC_TOKEN` (sourced from `steps.oidc.outputs.result`) and `CC_TOKEN=$INPUT_CODECOV_TOKEN` (sourced from `env.CODECOV_TOKEN`) directly to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker-controlled newline in these values could inject arbitrary environment variables.

Locations:

- `action.yml:201`

### github-env-injection (severity: high)

The 'Override branch for forks' step writes `TOKENLESS=$TOKENLESS` and `CC_BRANCH=$CC_BRANCH` to `$GITHUB_ENV` without sanitization. These variables are derived from `github.event.pull_request.head.label` (an attacker-controlled value in pull request events), passed via the `GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL` env var. A newline injected into the label value could inject arbitrary environment variables.

Locations:

- `action.yml:221`

### github-env-injection (severity: high)

The 'Override commits and pr for pull requests' step writes `CC_SHA=$CC_SHA` and `CC_PR=$CC_PR` to `$GITHUB_ENV` without sanitization. These variables are sourced from `github.event.pull_request.head.sha` and `github.event.number` (workflow-controlled values), passed via env vars `GITHUB_EVENT_PULL_REQUEST_HEAD_SHA` and `GITHUB_EVENT_NUMBER`. No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the writes.

Locations:

- `action.yml:238`

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

Fixed 4 categories of findings in action.yml:
1. script-injection (line 175): Moved `${{ github.workspace }}` from the run: shell string into an env: block as WORKSPACE_PATH, eliminating direct expression interpolation in the shell command.
2. github-env-injection + static-unsanitized-env-write in 'Get and set token' step: All three branches that write CC_TOKEN to $GITHUB_ENV now sanitize the value using `printf '%s' "$VAR" | tr -d '\n\r'` before writing.
3. github-env-injection + static-unsanitized-env-write in 'Override branch for forks' step: Both TOKENLESS and CC_BRANCH are now sanitized with printf/tr before writing to $GITHUB_ENV.
4. github-env-injection + static-unsanitized-env-write in 'Override commits and pr for pull requests' step: Both CC_SHA and CC_PR are now sanitized with printf/tr before writing to $GITHUB_ENV.

