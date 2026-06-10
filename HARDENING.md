<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v6.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **codecov--codecov-action/v6.0.0** was hardened automatically. 15 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Check system dependencies' run: block directly interpolates ${{ inputs.skip_validation }} inside the shell command string: `if [ "${{ inputs.skip_validation }}" != "true" ]`. This allows an attacker-controlled input value to be injected into the shell before quoting takes effect.

Locations:

- `action.yml:175`

### script-injection (severity: high)

Sub-rule (a): The 'Set safe directory' run: block directly interpolates ${{ github.workspace }} inside the shell command string: `git config --global --add safe.directory "${{ github.workspace }}"`  Any ${{ ... }} expression inside a run: block is a script-injection risk regardless of context.

Locations:

- `action.yml:196`

### script-injection (severity: high)

Sub-rule (a): The 'Get and set token' run: block directly interpolates multiple expressions inside shell command strings: `if [ "${{ inputs.use_oidc }}" == 'true' ]`, `elif [ -n "${{ env.CODECOV_TOKEN }}" ]`, `echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV"`, `if [ -n "${{ inputs.token }}" ]`, and `CC_TOKEN=$(echo "${{ inputs.token }}" | tr -d '\n')`. All of these are direct expression interpolations inside a run: block and constitute script injection.

Locations:

- `action.yml:221`

### github-env-injection (severity: high)

The 'Get and set token' step writes ${{ env.CODECOV_TOKEN }} (a workflow-controlled env context value) directly to $GITHUB_ENV without the required sanitization (`printf '%s' ... | tr -d '\n\r'`): `echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV"`. A newline in the value would allow injection of arbitrary environment variables.

Locations:

- `action.yml:226`

### github-env-injection (severity: high)

The 'Get and set token' step applies only `tr -d '\n'` (strips newlines but NOT carriage returns) before writing inputs.token to $GITHUB_ENV: `CC_TOKEN=$(echo "${{ inputs.token }}" | tr -d '\n')` then `echo "CC_TOKEN=$CC_TOKEN" >> "$GITHUB_ENV"`. The required sanitization is `tr -d '\n\r'`; omitting \r leaves a carriage-return injection vector.

Locations:

- `action.yml:229`

### github-env-injection (severity: high)

The 'Override branch for forks' step writes env vars derived from github.event.pull_request.head.label (untrusted, PR-attacker-controlled) to $GITHUB_ENV without sanitization: `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"`. The env vars TOKENLESS and CC_BRANCH are set from $GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL which is mapped from ${{ github.event.pull_request.head.label }}.

Locations:

- `action.yml:247`

### github-env-injection (severity: high)

The 'Override commits and pr for pull requests' step writes env vars derived from inputs.override_commit and inputs.override_pr (caller-controlled inputs) to $GITHUB_ENV without sanitization: `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` and `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"`. These are set from ${{ inputs.override_commit }} and ${{ inputs.override_pr }} via the env: block, with no `tr -d '\n\r'` applied before the write.

Locations:

- `action.yml:267`

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

Fixed all 15 findings in action.yml:
1. 'Check system dependencies': Moved ${{ inputs.skip_validation }} to env block (SKIP_VALIDATION), replaced inline expression in shell with $SKIP_VALIDATION.
2. 'Set safe directory': Moved ${{ github.workspace }} to env block (GH_WORKSPACE), replaced inline expression in shell with $GH_WORKSPACE.
3. 'Get and set token': Moved ${{ inputs.use_oidc }}, ${{ env.CODECOV_TOKEN }}, and ${{ inputs.token }} to env block as INPUT_USE_OIDC, CODECOV_TOKEN_ENV, and INPUT_TOKEN. All three GITHUB_ENV writes (OIDC token, env token, input token) now sanitize with `printf '%s' "$VAR" | tr -d '\n\r'` before writing.
4. 'Override branch for forks': TOKENLESS and CC_BRANCH now sanitized with `printf '%s' ... | tr -d '\n\r'` before writing to GITHUB_ENV.
5. 'Override commits and pr for pull requests': CC_SHA and CC_PR now sanitized with `printf '%s' ... | tr -d '\n\r'` before writing to GITHUB_ENV.

