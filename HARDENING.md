<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v6.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v6.0.0** was hardened automatically. 10 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Multiple `${{ }}` expressions are interpolated directly inside `run:` shell command strings, allowing script injection.

1. Step "Check system dependencies": `if [ "${{ inputs.skip_validation }}" != "true" ]` — the `inputs.skip_validation` value is injected directly into the shell before quoting can protect it.

2. Step "Set safe directory": `git config --global --add safe.directory "${{ github.workspace }}"` — `github.workspace` is interpolated directly into the shell command.

3. Step "Get and set token": Three direct interpolations in the run block:
   - `if [ "${{ inputs.use_oidc }}" == 'true' ]`
   - `elif [ -n "${{ env.CODECOV_TOKEN }}" ]`
   - `echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV"`
   - `if [ -n "${{ inputs.token }}" ]`
   - `CC_TOKEN=$(echo "${{ inputs.token }}" | tr -d '\n')`

All of these allow an attacker-controlled value to be interpreted as shell syntax before the shell ever sees it.

Locations:

- `action.yml:163`
- `action.yml:183`
- `action.yml:213`
- `action.yml:217`
- `action.yml:220`
- `action.yml:222`
- `action.yml:224`

### github-env-injection (severity: high)

Multiple `run:` steps write values derived from untrusted inputs or github context to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`).

1. Step "Get and set token": `echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV"` — directly writes an expression-interpolated value to GITHUB_ENV with no newline sanitization.

2. Step "Override branch for forks": `TOKENLESS` and `CC_BRANCH` are assigned from `$GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL` (which is `${{ github.event.pull_request.head.label }}` via the `env:` block) and then written to `$GITHUB_ENV` via `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"` — no sanitization applied.

3. Step "Override commits and pr": `CC_SHA` (from `${{ inputs.override_commit }}` / `${{ github.event.pull_request.head.sha }}`) and `CC_PR` (from `${{ inputs.override_pr }}` / `${{ github.event.number }}`) are written to `$GITHUB_ENV` via `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` and `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"` — no sanitization applied.

An attacker can inject newlines into these values to set arbitrary environment variables for subsequent steps.

Locations:

- `action.yml:220`
- `action.yml:237`
- `action.yml:239`
- `action.yml:253`
- `action.yml:254`

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

Fixed all findings in hardened/action/action.yml:

1. **Check system dependencies**: Moved `${{ inputs.skip_validation }}` to `env: INPUT_SKIP_VALIDATION` and referenced it as `$INPUT_SKIP_VALIDATION` in the shell script.

2. **Set safe directory**: Moved `${{ github.workspace }}` to `env: INPUT_GITHUB_WORKSPACE` and referenced it as `$INPUT_GITHUB_WORKSPACE` in the shell script.

3. **Get and set token**: Moved `${{ inputs.use_oidc }}`, `${{ env.CODECOV_TOKEN }}`, and `${{ inputs.token }}` to env block as `INPUT_USE_OIDC`, `INPUT_CODECOV_TOKEN`, and `INPUT_TOKEN`. Also added `printf '%s' ... | tr -d '\n\r'` sanitization for all values written to `$GITHUB_ENV` (CC_OIDC_TOKEN, CODECOV_TOKEN, and inputs.token paths).

4. **Override branch for forks**: Added `printf '%s' ... | tr -d '\n\r'` sanitization for `TOKENLESS` and `CC_BRANCH` before writing to `$GITHUB_ENV`.

5. **Override commits and pr for pull requests**: Added `printf '%s' ... | tr -d '\n\r'` sanitization for `CC_SHA` and `CC_PR` before writing to `$GITHUB_ENV`.

