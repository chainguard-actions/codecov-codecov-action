<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v5.5.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v5.5.5** was hardened automatically. 10 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Multiple `${{ }}` expressions are interpolated directly inside `run:` shell command strings in action.yml, violating rule (a). This allows template substitution to inject arbitrary shell metacharacters before the shell parses the string.

1. Step 'Check system dependencies': `if [ "${{ inputs.skip_validation }}" != "true" ]` — inputs.skip_validation is interpolated directly into a shell `if` condition.
2. Step 'Set safe directory': `git config --global --add safe.directory "${{ github.workspace }}"` — github.workspace is interpolated directly into a shell command.
3. Step 'Get and set token': `if [ "${{ inputs.use_oidc }}" == 'true' ]` — inputs.use_oidc is interpolated directly into a shell `if` condition.
4. Step 'Get and set token': `elif [ -n "${{ env.CODECOV_TOKEN }}" ]` and `echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV"` — env.CODECOV_TOKEN is interpolated directly into shell conditions and an echo command.
5. Step 'Get and set token': `if [ -n "${{ inputs.token }}" ]` — inputs.token is interpolated directly into a shell `if` condition.

Locations:

- `action.yml:163`
- `action.yml:179`
- `action.yml:199`
- `action.yml:202`
- `action.yml:204`
- `action.yml:206`

### github-env-injection (severity: high)

Multiple `run:` steps write values derived from untrusted GitHub context inputs to `$GITHUB_ENV` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). An attacker can inject newlines into these values to poison subsequent environment variables.

1. Step 'Set fork' (rule c/e): `echo "CC_FORK=$CC_FORK" >> "$GITHUB_ENV"` — CC_FORK is computed from `$GITHUB_EVENT_PULL_REQUEST_HEAD_REPO_FULL_NAME` (sourced from `github.event.pull_request.head.repo.full_name`) without sanitization before the write.
2. Step 'Get and set token' (rule a): `echo "CC_TOKEN=${{ env.CODECOV_TOKEN }}" >> "$GITHUB_ENV"` — env.CODECOV_TOKEN is written directly to $GITHUB_ENV with a ${{ }} expression, with no sanitization.
3. Step 'Override branch for forks' (rule c/e): `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"` — both variables are derived from `$GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL` (sourced from `github.event.pull_request.head.label`) without sanitization.
4. Step 'Override commits and pr' (rule c/e): `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` and `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"` — CC_SHA is derived from `github.event.pull_request.head.sha` and CC_PR from `github.event.number`, both written to $GITHUB_ENV without sanitization.

Locations:

- `action.yml:191`
- `action.yml:204`
- `action.yml:220`
- `action.yml:221`
- `action.yml:233`
- `action.yml:234`

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

1. script-injection / static-inline-injection: Moved all ${{ }} expressions out of run: shell blocks into env: blocks:
   - 'Check system dependencies': ${{ inputs.skip_validation }} → SKIP_VALIDATION env var
   - 'Set safe directory': ${{ github.workspace }} → INPUT_WORKSPACE env var
   - 'Get and set token': ${{ inputs.use_oidc }}, ${{ env.CODECOV_TOKEN }}, ${{ inputs.token }} → INPUT_USE_OIDC, INPUT_CODECOV_TOKEN, INPUT_TOKEN env vars

2. github-env-injection / static-unsanitized-env-write: Added printf '%s' ... | tr -d '\n\r' sanitization before all $GITHUB_ENV writes:
   - 'Set fork': CC_FORK sanitized
   - 'Get and set token': CC_TOKEN (from OIDC, env, or input) sanitized
   - 'Override branch for forks': TOKENLESS and CC_BRANCH sanitized
   - 'Override commits and pr for pull requests': CC_SHA and CC_PR sanitized

### Iteration 2

**Fixes applied:** script-injection, unpinned-uses, broad-permissions, missing-permissions

**Notes:**

Fixed all 4 findings: (1) Script injection in main.yml lines 131 and 195 - moved ${{ steps.codecov-upload.outcome }} to env blocks as CODECOV_UPLOAD_OUTCOME; (2) Pinned all unpinned actions to full SHAs: actions/checkout@v5.0.0→08c6903cd8c0fde910a37f88322edcfb5dd907a8 (8 occurrences across 3 files), github/codeql-action/{init,autobuild,analyze,upload-sarif}@v3.30.0→2d92b76c45b91eb80fc44c74ce3fce0ee94e8f9d; (3) Replaced broad 'permissions: read-all' with 'permissions: {}' in scorecards-analysis.yml (job-level permissions already set explicitly); (4) Added 'permissions: contents: read' to enforce-license-compliance.yml.

