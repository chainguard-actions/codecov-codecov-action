# Hardening Report: codecov--codecov-action/v7.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **codecov--codecov-action/v7.0.0** was hardened automatically. 9 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

The 'Set safe directory' step directly interpolates `${{ github.workspace }}` inside the `run:` shell command string rather than assigning it to an environment variable first. An attacker who can influence the workspace path could inject shell commands. The vulnerable line is: `git config --global --add safe.directory "${{ github.workspace }}"`

Locations:

- `action.yml:174`

### github-env-injection (severity: high)

The 'Override branch for forks' step writes attacker-controlled values to $GITHUB_ENV without sanitization. The env var GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL is set from `${{ github.event.pull_request.head.label }}` (attacker-controlled via PR branch name) and CC_BRANCH from `${{ inputs.override_branch }}`. Both TOKENLESS and CC_BRANCH are written to $GITHUB_ENV via `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"` without the required `printf '%s' ... | tr -d '\n\r'` sanitization. A newline in the label or input could inject arbitrary environment variables.

Locations:

- `action.yml:228`

### github-env-injection (severity: high)

The 'Override commits and pr for pull requests' step writes attacker-controlled input values to $GITHUB_ENV without sanitization. CC_SHA is set from `${{ inputs.override_commit }}` and CC_PR from `${{ inputs.override_pr }}`, then written to $GITHUB_ENV via `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` and `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"` without the required `printf '%s' ... | tr -d '\n\r'` sanitization. A newline in either input could inject arbitrary environment variables.

Locations:

- `action.yml:250`

### github-env-injection (severity: high)

The 'Get and set token' step writes token values to $GITHUB_ENV without full sanitization on two of three code paths. The OIDC path writes `echo "CC_TOKEN=$CC_OIDC_TOKEN" >> "$GITHUB_ENV"` and the CODECOV_TOKEN env path writes `echo "CC_TOKEN=$INPUT_CODECOV_TOKEN" >> "$GITHUB_ENV"` — neither applies the required `printf '%s' ... | tr -d '\n\r'` sanitization before writing to $GITHUB_ENV. Only the `inputs.token` path applies a partial `tr -d '\n'` (missing the full sanitization pipeline). A newline embedded in the token value could inject arbitrary environment variables.

Locations:

- `action.yml:205`

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

Fixed 4 categories of findings in action.yml:
1. script-injection (line 174): Moved `${{ github.workspace }}` from the run: shell string into an env: block as SAFE_DIRECTORY, referenced as $SAFE_DIRECTORY in the shell.
2. github-env-injection + static-unsanitized-env-write in 'Get and set token' (lines 205/252/256): All three token code paths now use `safe_token=$(printf '%s' "$VAR" | tr -d '\n\r')` before writing CC_TOKEN to $GITHUB_ENV.
3. github-env-injection + static-unsanitized-env-write in 'Override branch for forks' (lines 228/283): TOKENLESS and CC_BRANCH are now sanitized with printf/tr before writing to $GITHUB_ENV.
4. github-env-injection + static-unsanitized-env-write in 'Override commits and pr for pull requests' (lines 250/302/303): CC_SHA and CC_PR are now sanitized with printf/tr before writing to $GITHUB_ENV.

