<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v7.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v7.0.0** was hardened automatically. 7 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The 'Set safe directory' step's run: block directly interpolates the GitHub Actions expression `${{ github.workspace }}` inside the shell command string. Before the shell executes the command, GitHub Actions performs template substitution on the expression, meaning any value in github.workspace is injected verbatim into the shell command without quoting or escaping. The offending line is: `git config --global --add safe.directory "${{ github.workspace }}"`  — this should use the environment variable `$GITHUB_WORKSPACE` instead (already present on the next line), or route through an env: block and reference the env var.

Locations:

- `action.yml:204`

### github-env-injection (severity: high)

Multiple steps write untrusted values to $GITHUB_ENV without the required sanitization step (`printf '%s' "$VAR" | tr -d '\n\r'`). An attacker can inject newlines into these values to poison subsequent environment variables or outputs.

(1) 'Get and set token' step: `echo "CC_TOKEN=$CC_OIDC_TOKEN" >> "$GITHUB_ENV"` — CC_OIDC_TOKEN is sourced from steps.oidc.outputs.result (untrusted step output). Also `echo "CC_TOKEN=$INPUT_CODECOV_TOKEN" >> "$GITHUB_ENV"` — INPUT_CODECOV_TOKEN is sourced from env.CODECOV_TOKEN (inherited workflow env var, untrusted). Neither write is preceded by newline sanitization.

(2) 'Override branch for forks' step: `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"` — both values derive from GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL which is set from `github.event.pull_request.head.label`, an attacker-controlled value via pull request.

(3) 'Override commits and pr for pull requests' step: `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` and `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"` — CC_SHA derives from inputs.override_commit and github.event.pull_request.head.sha; CC_PR derives from inputs.override_pr and github.event.number — all untrusted sources.

Locations:

- `action.yml:213`
- `action.yml:217`
- `action.yml:238`
- `action.yml:241`
- `action.yml:257`
- `action.yml:258`

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

Fixed all 7 findings in hardened/action/action.yml:
1. script-injection (line 204): Replaced `${{ github.workspace }}` with `$GITHUB_WORKSPACE` in the 'Set safe directory' step — the env var is already available and avoids template injection.
2. github-env-injection + static-unsanitized-env-write: Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization before all writes to $GITHUB_ENV:
   - 'Get and set token' step: sanitized CC_OIDC_TOKEN (from steps.oidc.outputs.result) and INPUT_CODECOV_TOKEN (from env.CODECOV_TOKEN) via safe_token variable.
   - 'Override branch for forks' step: sanitized TOKENLESS and CC_BRANCH (both derived from github.event.pull_request.head.label) via safe_tokenless and safe_branch variables.
   - 'Override commits and pr for pull requests' step: sanitized CC_SHA (from inputs.override_commit / github.event.pull_request.head.sha) and CC_PR (from inputs.override_pr / github.event.number) via safe_sha and safe_pr variables.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the incomplete sanitization in the INPUT_TOKEN branch of the 'Get and set token' step in action.yml. Changed `CC_TOKEN=$(echo "$INPUT_TOKEN" | tr -d '\n')` to `safe_token=$(printf '%s' "$INPUT_TOKEN" | tr -d '\n\r')` and updated the echo line to use `safe_token`. This now uses `printf '%s'` instead of `echo` and strips both `\n` and `\r` characters, consistent with the OIDC and CODECOV_TOKEN branches in the same step.

