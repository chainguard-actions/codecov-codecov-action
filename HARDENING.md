<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v5.5.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v5.5.5** was hardened automatically. 10 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple ${{ }} expressions are interpolated directly inside run: shell command strings in action.yml, violating rule (a). This allows an attacker who controls the calling workflow's inputs or github context to inject arbitrary shell commands.

1. 'Check system dependencies' step (line ~185): `if [ "${{ inputs.skip_validation }}" != "true" ]; then` — inputs.skip_validation interpolated directly into shell.

2. 'Set safe directory' step (line ~192): `git config --global --add safe.directory "${{ github.workspace }}"` — github.workspace interpolated directly into shell.

3. 'Get and set token' step (lines ~213–225): `if [ "${{ inputs.use_oidc }}" == 'true' ]`, `elif [ -n "${{ env.CODECOV_TOKEN }}" ]`, `echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV"`, `if [ -n "${{ inputs.token }}" ]`, and `CC_TOKEN=$(echo "${{ inputs.token }}" | tr -d '\n')` — inputs.use_oidc, env.CODECOV_TOKEN, and inputs.token all interpolated directly into shell commands.

All ${{ ... }} expressions inside run: blocks should be moved to env: variables and then referenced as quoted shell variables (e.g., "$VAR").

Locations:

- `action.yml:185`
- `action.yml:192`
- `action.yml:213`
- `action.yml:216`
- `action.yml:219`
- `action.yml:222`
- `action.yml:225`

### github-env-injection (severity: high)

In the 'Get and set token' step, the value of env.CODECOV_TOKEN is written directly to $GITHUB_ENV without sanitization: `echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV"`. The env.* context is an untrusted-input source — a calling workflow can set CODECOV_TOKEN to a value containing newlines, which would allow injection of arbitrary environment variables into subsequent steps. The required sanitization step (`safe=$(printf '%s' "$VAR" | tr -d '\n\r')`) is missing before the write.

Locations:

- `action.yml:219`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.skip_validation }}" appears directly in run: block of step "Check system dependencies"; move to env: map

Locations:

- `action.yml:191`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.use_oidc }}" appears directly in run: block of step "Get and set token"; move to env: map

Locations:

- `action.yml:248`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.token }}" appears directly in run: block of step "Get and set token"; move to env: map

Locations:

- `action.yml:256`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.token }}" appears directly in run: block of step "Get and set token"; move to env: map

Locations:

- `action.yml:259`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $CC_OIDC_TOKEN in step "Get and set token" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:250`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $CC_BRANCH in step "Override branch for forks" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:278`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $CC_SHA in step "Override commits and pr for pull requests" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:297`

### static-unsanitized-env-write (severity: medium)

unsanitized write to $GITHUB_ENV: variable $CC_PR in step "Override commits and pr for pull requests" comes from a ${{...}} expression and should be sanitized with printf/tr before writing

Locations:

- `action.yml:298`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection, static-unsanitized-env-write

**Notes:**

Fixed all security findings in hardened/action/action.yml:

1. 'Check system dependencies' step: Moved `${{ inputs.skip_validation }}` to env: block as INPUT_SKIP_VALIDATION, referenced as $INPUT_SKIP_VALIDATION in shell.

2. 'Set safe directory' step: Moved `${{ github.workspace }}` to env: block as GITHUB_WORKSPACE_INPUT, referenced as $GITHUB_WORKSPACE_INPUT in shell.

3. 'Get and set token' step: Moved `${{ inputs.use_oidc }}`, `${{ env.CODECOV_TOKEN }}`, and `${{ inputs.token }}` to env: block as INPUT_USE_OIDC, CODECOV_TOKEN_ENV, and INPUT_TOKEN respectively. All three GITHUB_ENV writes now sanitize values with `printf '%s' "$VAR" | tr -d '\n\r'` before writing.

4. 'Override branch for forks' step: Added sanitization `safe_branch=$(printf '%s' "$CC_BRANCH" | tr -d '\n\r')` before writing CC_BRANCH to GITHUB_ENV.

5. 'Override commits and pr for pull requests' step: Added sanitization for both CC_SHA and CC_PR using `printf '%s' "$VAR" | tr -d '\n\r'` before writing to GITHUB_ENV.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection vulnerability in the 'Override branch for forks' step of action.yml. Added sanitization for the TOKENLESS variable before writing it to $GITHUB_ENV: `safe_tokenless=$(printf '%s' "$TOKENLESS" | tr -d '\n\r')` followed by `echo "TOKENLESS=$safe_tokenless" >> "$GITHUB_ENV"`. This mirrors the existing sanitization pattern already applied to CC_BRANCH in the same block, preventing newline injection attacks from attacker-controlled PR head label values.

