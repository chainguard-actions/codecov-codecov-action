<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v6.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v6.0.1** was hardened automatically. 9 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The expression `${{ github.workspace }}` is directly interpolated inside a `run:` shell command string in the 'Set safe directory' step. Before the shell ever sees the value, GitHub Actions performs YAML template substitution, meaning a crafted workspace path could inject shell metacharacters. The offending line is: `git config --global --add safe.directory "${{ github.workspace }}"`

Locations:

- `action.yml:192`

### github-env-injection (severity: high)

The 'Get and set token' step writes untrusted values to $GITHUB_ENV without the required sanitization (`printf '%s' ... | tr -d '\n\r'`). Three unsanitized writes occur: (1) `echo "CC_TOKEN=$CC_OIDC_TOKEN" >> "$GITHUB_ENV"` where CC_OIDC_TOKEN comes from steps.oidc.outputs.result (untrusted step output); (2) `echo "CC_TOKEN=$INPUT_CODECOV_TOKEN" >> "$GITHUB_ENV"` where INPUT_CODECOV_TOKEN is the inherited env var CODECOV_TOKEN (workflow-controlled); (3) `echo "CC_TOKEN=$CC_TOKEN" >> "$GITHUB_ENV"` where CC_TOKEN is derived from INPUT_TOKEN (inputs.token) — only `\n` is stripped via `tr -d '\n'`, but `\r` is not stripped, so the sanitization is incomplete.

Locations:

- `action.yml:224`
- `action.yml:228`
- `action.yml:233`

### github-env-injection (severity: high)

The 'Override branch for forks' step writes attacker-controlled values to $GITHUB_ENV without sanitization. `TOKENLESS` and `CC_BRANCH` are both derived from `$GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL` (mapped from `github.event.pull_request.head.label`, which is attacker-controlled on PRs from forks), and `CC_BRANCH` can also come from `inputs.override_branch`. Neither write applies the required `printf '%s' ... | tr -d '\n\r'` sanitization before writing to $GITHUB_ENV: `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"`.

Locations:

- `action.yml:250`
- `action.yml:253`

### github-env-injection (severity: high)

The 'Override commits and pr for pull requests' step writes attacker-controlled values to $GITHUB_ENV without sanitization. `CC_SHA` is derived from `inputs.override_commit` or `github.event.pull_request.head.sha`, and `CC_PR` is derived from `inputs.override_pr` or `github.event.number`. Neither write applies the required `printf '%s' ... | tr -d '\n\r'` sanitization: `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` and `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"`.

Locations:

- `action.yml:269`
- `action.yml:270`

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
1. script-injection: Moved `${{ github.workspace }}` from inline `run:` string to `env: SAFE_DIRECTORY:` block, referenced as `$SAFE_DIRECTORY` in shell.
2. github-env-injection (Get and set token): All three GITHUB_ENV writes now sanitize with `printf '%s' "$VAR" | tr -d '\n\r'` before writing — covers CC_OIDC_TOKEN, INPUT_CODECOV_TOKEN, and INPUT_TOKEN cases.
3. github-env-injection (Override branch for forks): Both TOKENLESS and CC_BRANCH writes now sanitize with printf/tr before writing to GITHUB_ENV.
4. github-env-injection (Override commits and pr for pull requests): Both CC_SHA and CC_PR writes now sanitize with printf/tr before writing to GITHUB_ENV.
All static-unsanitized-env-write findings are covered by the same fixes above.

