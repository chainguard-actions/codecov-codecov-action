<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v7.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v7.1.1** was hardened automatically. 9 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A GitHub Actions expression `${{ github.workspace }}` is directly interpolated inside a `run:` shell command in the 'Set safe directory' step. This causes YAML template substitution to inject the value directly into the shell command string before the shell ever sees it, enabling script injection if the value contains shell metacharacters. The offending line is: `git config --global --add safe.directory "${{ github.workspace }}"`

Locations:

- `action.yml:182`

### github-env-injection (severity: high)

The 'Get and set token' step writes unsanitized values to $GITHUB_ENV. (1) `echo "CC_TOKEN=$CC_OIDC_TOKEN" >> "$GITHUB_ENV"` — CC_OIDC_TOKEN is sourced from `${{ steps.oidc.outputs.result }}` with no newline sanitization. (2) `echo "CC_TOKEN=$INPUT_CODECOV_TOKEN" >> "$GITHUB_ENV"` — INPUT_CODECOV_TOKEN is sourced from `${{ env.CODECOV_TOKEN }}` (an inherited workflow env var) with no sanitization. An attacker-controlled newline in these values can inject arbitrary environment variables into subsequent steps. The required sanitization (`printf '%s' "$VAR" | tr -d '\n\r'`) is absent for these two branches.

Locations:

- `action.yml:211`
- `action.yml:215`

### github-env-injection (severity: high)

The 'Override branch for forks' step writes unsanitized values derived from `${{ github.event.pull_request.head.label }}` to $GITHUB_ENV. The env var `GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL` (set from `${{ github.event.pull_request.head.label }}`) is assigned to `TOKENLESS` and `CC_BRANCH`, then written: `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"`. A PR head label containing a newline can inject arbitrary environment variables. No `tr -d '\n\r'` sanitization is applied.

Locations:

- `action.yml:234`
- `action.yml:237`

### github-env-injection (severity: high)

The 'Override commits and pr for pull requests' step writes unsanitized input-derived values to $GITHUB_ENV. `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` — CC_SHA is sourced from `${{ inputs.override_commit }}`. `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"` — CC_PR is sourced from `${{ inputs.override_pr }}`. Both are caller-controlled inputs written directly to the environment file without the required `printf '%s' ... | tr -d '\n\r'` sanitization, allowing newline injection to set arbitrary environment variables.

Locations:

- `action.yml:251`
- `action.yml:252`

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

Fixed all 9 findings in hardened/action/action.yml:

1. script-injection (line 182): Moved `${{ github.workspace }}` from the `run:` shell string into an `env:` block as `SAFE_DIRECTORY`, then referenced it as `"$SAFE_DIRECTORY"` in the shell script.

2. github-env-injection + static-unsanitized-env-write (lines 211, 215, 256, 260): In 'Get and set token', sanitized `CC_OIDC_TOKEN` and `INPUT_CODECOV_TOKEN` with `printf '%s' "$VAR" | tr -d '\n\r'` before writing to $GITHUB_ENV.

3. github-env-injection + static-unsanitized-env-write (lines 234, 237, 287): In 'Override branch for forks', sanitized `TOKENLESS` and `CC_BRANCH` with `printf '%s' "$VAR" | tr -d '\n\r'` before writing to $GITHUB_ENV.

4. github-env-injection + static-unsanitized-env-write (lines 251, 252, 306, 307): In 'Override commits and pr for pull requests', sanitized `CC_SHA` and `CC_PR` with `printf '%s' "$VAR" | tr -d '\n\r'` before writing to $GITHUB_ENV.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the else-branch in the 'Get and set token' step (action.yml ~line 230). Changed `CC_TOKEN=$(echo "$INPUT_TOKEN" | tr -d '\n')` to `safe_token=$(printf '%s' "$INPUT_TOKEN" | tr -d '\n\r')` to strip both newlines and carriage returns before writing to $GITHUB_ENV. This prevents carriage-return-based environment variable injection and makes the else-branch consistent with the other two branches in the same step.

