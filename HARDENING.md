<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v6.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v6.0.1** was hardened automatically. 9 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The 'Set safe directory' step directly interpolates `${{ github.workspace }}` inside a `run:` shell command string. Before the shell executes the command, GitHub Actions performs template substitution, so an attacker who can influence `github.workspace` (or any future context value) can inject arbitrary shell commands. The offending line is: `git config --global --add safe.directory "${{ github.workspace }}"`  — this should be replaced with the safe env-var form `"$GITHUB_WORKSPACE"` (which is already used on the very next line).

Locations:

- `action.yml:177`

### github-env-injection (severity: high)

The 'Get and set token' step writes untrusted values to $GITHUB_ENV without sanitization. Specifically: (1) `echo "CC_TOKEN=$CC_OIDC_TOKEN" >> "$GITHUB_ENV"` — CC_OIDC_TOKEN is sourced from `steps.oidc.outputs.result` (an untrusted step output); (2) `echo "CC_TOKEN=$INPUT_CODECOV_TOKEN" >> "$GITHUB_ENV"` — INPUT_CODECOV_TOKEN is the inherited process env var CODECOV_TOKEN set by the calling workflow. Neither write is preceded by the required `printf '%s' ... | tr -d '\n\r'` sanitization, allowing newline injection into $GITHUB_ENV to set arbitrary environment variables.

Locations:

- `action.yml:205`
- `action.yml:209`

### github-env-injection (severity: high)

The 'Override branch for forks' step writes attacker-controlled values to $GITHUB_ENV without sanitization. `TOKENLESS` and `CC_BRANCH` are both derived from the env var `GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL`, which is set from `${{ github.event.pull_request.head.label }}` — a value fully controlled by a pull request author. The writes `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"` are not preceded by the required `printf '%s' ... | tr -d '\n\r'` sanitization, enabling newline injection to set arbitrary environment variables.

Locations:

- `action.yml:230`
- `action.yml:233`

### github-env-injection (severity: high)

The 'Override commits and pr for pull requests' step writes untrusted values to $GITHUB_ENV without sanitization. `CC_SHA` is sourced from `inputs.override_commit` or `github.event.pull_request.head.sha`, and `CC_PR` from `inputs.override_pr` or `github.event.number` — all caller-controlled. The writes `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` and `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"` are not preceded by the required `printf '%s' ... | tr -d '\n\r'` sanitization, enabling newline injection to set arbitrary environment variables.

Locations:

- `action.yml:250`
- `action.yml:251`

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
1. script-injection: Replaced `${{ github.workspace }}` with `$GITHUB_WORKSPACE` in the 'Set safe directory' step (line 177).
2. github-env-injection + static-unsanitized-env-write: Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before all writes to $GITHUB_ENV in three steps: 'Get and set token' (CC_OIDC_TOKEN and INPUT_CODECOV_TOKEN), 'Override branch for forks' (TOKENLESS and CC_BRANCH), and 'Override commits and pr for pull requests' (CC_SHA and CC_PR).

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the INPUT_TOKEN branch in the 'Get and set token' step (action.yml ~line 248). Changed `CC_TOKEN=$(echo "$INPUT_TOKEN" | tr -d '\n')` to `safe_token=$(printf '%s' "$INPUT_TOKEN" | tr -d '\n\r')` and updated the echo to use `$safe_token`. This ensures carriage returns (\r) are also stripped before writing to $GITHUB_ENV, consistent with the other two branches in the same step.

