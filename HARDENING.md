<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v6.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v6.0.1** was hardened automatically. 9 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Set safe directory' run: block directly interpolates a ${{ }} expression inside a shell command string: `git config --global --add safe.directory "${{ github.workspace }}"`  Any ${{ ... }} expression interpolated directly into a run: script is a script-injection risk because YAML template substitution occurs before the shell ever sees the value, bypassing shell quoting. The value should be passed via an env: variable and referenced as `"$GITHUB_WORKSPACE"` instead.

Locations:

- `action.yml:175`

### github-env-injection (severity: high)

The 'Get and set token' step writes workflow-controlled values to $GITHUB_ENV without sanitization. (1) `echo "CC_TOKEN=$CC_OIDC_TOKEN" >> "$GITHUB_ENV"` — CC_OIDC_TOKEN is set from `${{ steps.oidc.outputs.result }}` (a step output, untrusted). (2) `echo "CC_TOKEN=$INPUT_CODECOV_TOKEN" >> "$GITHUB_ENV"` — INPUT_CODECOV_TOKEN is set from `${{ env.CODECOV_TOKEN }}` (a workflow-controlled env var). Neither write is preceded by the required `printf '%s' ... | tr -d '\n\r'` sanitization step, allowing newline injection into $GITHUB_ENV.

Locations:

- `action.yml:215`
- `action.yml:219`

### github-env-injection (severity: high)

The 'Override branch for forks' step writes attacker-controlled values to $GITHUB_ENV without sanitization. `TOKENLESS` and `CC_BRANCH` are both derived from the env var `GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL`, which is set from `${{ github.event.pull_request.head.label }}` — a value that can be controlled by a pull request author. The writes `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"` are not preceded by the required `printf '%s' ... | tr -d '\n\r'` sanitization, allowing newline injection into $GITHUB_ENV.

Locations:

- `action.yml:240`
- `action.yml:242`

### github-env-injection (severity: high)

The 'Override commits and pr for pull requests' step writes user-supplied input values to $GITHUB_ENV without sanitization. `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` — CC_SHA is set from `${{ inputs.override_commit }}` (caller-controlled). `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"` — CC_PR is set from `${{ inputs.override_pr }}` (caller-controlled). Neither write is preceded by the required `printf '%s' ... | tr -d '\n\r'` sanitization step, allowing newline injection into $GITHUB_ENV.

Locations:

- `action.yml:258`
- `action.yml:259`

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
1. script-injection (line 175): Removed the redundant `git config --global --add safe.directory "${{ github.workspace }}"` line. The subsequent line using `$GITHUB_WORKSPACE` env var is sufficient and safe.
2. github-env-injection in 'Get and set token': Added `printf '%s' ... | tr -d '\n\r'` sanitization for CC_OIDC_TOKEN and INPUT_CODECOV_TOKEN before writing to $GITHUB_ENV.
3. github-env-injection in 'Override branch for forks': Added `printf '%s' ... | tr -d '\n\r'` sanitization for TOKENLESS and CC_BRANCH before writing to $GITHUB_ENV.
4. github-env-injection in 'Override commits and pr for pull requests': Added `printf '%s' ... | tr -d '\n\r'` sanitization for CC_SHA and CC_PR before writing to $GITHUB_ENV.
All static-unsanitized-env-write findings are also resolved by the same sanitization changes.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the incomplete sanitization in the 'Get and set token' step's INPUT_TOKEN branch. Changed `CC_TOKEN=$(echo "$INPUT_TOKEN" | tr -d '\n')` to `CC_TOKEN=$(printf '%s' "$INPUT_TOKEN" | tr -d '\n\r')`. This ensures carriage returns are also stripped before writing to $GITHUB_ENV, consistent with the other two token branches in the same step that already used the correct `printf '%s' | tr -d '\n\r'` pattern.

