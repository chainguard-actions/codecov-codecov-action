<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v7.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v7.0.0** was hardened automatically. 9 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Set safe directory' step directly interpolates the GitHub Actions expression `${{ github.workspace }}` inside a `run:` shell command string. Any `${{ ... }}` expression interpolated directly into a shell script is a script-injection risk because the value is substituted by the Actions runner before the shell ever sees it, bypassing shell quoting. Offending line: `git config --global --add safe.directory "${{ github.workspace }}"`

Locations:

- `action.yml:172`

### github-env-injection (severity: high)

The 'Get and set token' step writes values from untrusted sources to $GITHUB_ENV without sanitization (no `printf '%s' ... | tr -d '\n\r'` applied before the write). Specifically: (1) `echo "CC_TOKEN=$CC_OIDC_TOKEN" >> "$GITHUB_ENV"` where CC_OIDC_TOKEN comes from `steps.oidc.outputs.result` (an untrusted step output); (2) `echo "CC_TOKEN=$INPUT_CODECOV_TOKEN" >> "$GITHUB_ENV"` where INPUT_CODECOV_TOKEN comes from `env.CODECOV_TOKEN` (a workflow-controlled env var). An attacker who can control these values could inject newlines to set arbitrary environment variables.

Locations:

- `action.yml:200`

### github-env-injection (severity: high)

The 'Override branch for forks' step writes attacker-controlled values to $GITHUB_ENV without sanitization. `TOKENLESS` and `CC_BRANCH` are derived from the env var `GITHUB_EVENT_PULL_REQUEST_HEAD_LABEL`, which maps to `${{ github.event.pull_request.head.label }}` — a value fully controlled by a pull request author. The writes `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"` lack the required `printf '%s' ... | tr -d '\n\r'` sanitization step, enabling environment variable injection via newline characters.

Locations:

- `action.yml:225`

### github-env-injection (severity: high)

The 'Override commits and pr for pull requests' step writes values from untrusted inputs to $GITHUB_ENV without sanitization. `CC_SHA` is sourced from `inputs.override_commit` and `CC_PR` from `inputs.override_pr` — both are caller-supplied inputs. The writes `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` and `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"` lack the required `printf '%s' ... | tr -d '\n\r'` sanitization, allowing an attacker to inject newlines and set arbitrary environment variables.

Locations:

- `action.yml:246`

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
1. script-injection (line 172): Moved `${{ github.workspace }}` into an env: block as WORKSPACE and referenced it as $WORKSPACE in the shell script.
2. github-env-injection + static-unsanitized-env-write in 'Get and set token' step: Added printf/tr sanitization for CC_OIDC_TOKEN and INPUT_CODECOV_TOKEN before writing to $GITHUB_ENV.
3. github-env-injection + static-unsanitized-env-write in 'Override branch for forks' step: Added printf/tr sanitization for TOKENLESS and CC_BRANCH before writing to $GITHUB_ENV.
4. github-env-injection + static-unsanitized-env-write in 'Override commits and pr for pull requests' step: Added printf/tr sanitization for CC_SHA and CC_PR before writing to $GITHUB_ENV.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Get and set token' step's else-branch in action.yml (line 247). Changed `CC_TOKEN=$(echo "$INPUT_TOKEN" | tr -d '\n')` to `CC_TOKEN=$(printf '%s' "$INPUT_TOKEN" | tr -d '\n\r')`. This ensures carriage returns are also stripped before writing to $GITHUB_ENV, preventing potential injection of additional key=value pairs. The fix is now consistent with all other GITHUB_ENV writes in the file which already used the correct `printf '%s' ... | tr -d '\n\r'` pattern.

