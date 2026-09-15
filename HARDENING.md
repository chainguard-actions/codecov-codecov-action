<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v7.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v7.1.0** was hardened automatically. 9 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The 'Set safe directory' step directly interpolates `${{ github.workspace }}` inside a `run:` shell command string. Before the shell executes the command, GitHub Actions performs template substitution on the expression, allowing an attacker who controls the workspace path to inject shell metacharacters. The offending line is: `git config --global --add safe.directory "${{ github.workspace }}"`

Locations:

- `action.yml:220`

### github-env-injection (severity: high)

The 'Get and set token' step writes untrusted values to $GITHUB_ENV without sanitization. (1) `echo "CC_TOKEN=$CC_OIDC_TOKEN" >> "$GITHUB_ENV"` — CC_OIDC_TOKEN is sourced from `steps.oidc.outputs.result` (an untrusted step output) with no newline stripping. (2) `echo "CC_TOKEN=$INPUT_CODECOV_TOKEN" >> "$GITHUB_ENV"` — INPUT_CODECOV_TOKEN is sourced from `env.CODECOV_TOKEN`, an inherited process env var set by the calling workflow, with no sanitization. An attacker-controlled value containing newlines could inject arbitrary environment variables into subsequent steps. The required sanitization (`printf '%s' "$VAR" | tr -d '\n\r'`) is absent for both paths.

Locations:

- `action.yml:256`
- `action.yml:260`

### github-env-injection (severity: high)

The 'Override branch for forks' step writes attacker-controlled values to $GITHUB_ENV without sanitization. (1) `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` — TOKENLESS is derived from GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL, which is set from `${{ github.event.pull_request.head.label }}` (attacker-controlled via PR). (2) `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"` — CC_BRANCH is set from `${{ inputs.override_branch }}` (caller-controlled input). Neither write is preceded by the required `printf '%s' ... | tr -d '\n\r'` sanitization, allowing newline injection into $GITHUB_ENV.

Locations:

- `action.yml:284`
- `action.yml:287`

### github-env-injection (severity: high)

The 'Override commits and pr for pull requests' step writes untrusted input values to $GITHUB_ENV without sanitization. (1) `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` — CC_SHA is sourced from `${{ inputs.override_commit }}` (caller-controlled). (2) `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"` — CC_PR is sourced from `${{ inputs.override_pr }}` (caller-controlled). Neither write is preceded by the required `printf '%s' ... | tr -d '\n\r'` sanitization, allowing a malicious caller to inject arbitrary environment variables into subsequent steps via embedded newlines.

Locations:

- `action.yml:306`
- `action.yml:307`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $CC_OIDC_TOKEN in step "Get and set token" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:256`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $INPUT_CODECOV_TOKEN in step "Get and set token" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:260`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $CC_BRANCH in step "Override branch for forks" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:287`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $CC_SHA in step "Override commits and pr for pull requests" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:306`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $CC_PR in step "Override commits and pr for pull requests" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:307`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-unsanitized-env-write

**Notes:**

Fixed all 9 findings in action.yml:
1. script-injection (line 220): Moved `${{ github.workspace }}` into an env var `WORKSPACE_PATH` and referenced it as `$WORKSPACE_PATH` in the shell script.
2. github-env-injection + static-unsanitized-env-write (lines 256, 260): Added `printf '%s' "$VAR" | tr -d '\n\r'` sanitization for CC_OIDC_TOKEN and INPUT_CODECOV_TOKEN before writing to $GITHUB_ENV in the 'Get and set token' step.
3. github-env-injection + static-unsanitized-env-write (lines 284, 287): Added sanitization for TOKENLESS and CC_BRANCH before writing to $GITHUB_ENV in the 'Override branch for forks' step.
4. github-env-injection + static-unsanitized-env-write (lines 306, 307): Added sanitization for CC_SHA and CC_PR before writing to $GITHUB_ENV in the 'Override commits and pr for pull requests' step.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the token sanitization in the 'Get and set token' step's else branch in hardened/action/action.yml. Changed `CC_TOKEN=$(echo "$INPUT_TOKEN" | tr -d '\n')` to `CC_TOKEN=$(printf '%s' "$INPUT_TOKEN" | tr -d '\n\r')` to strip both newlines and carriage returns before writing to $GITHUB_ENV, preventing potential environment variable injection via carriage returns.

