<!-- markdownlint-disable -->

# Hardening Report: codecov--codecov-action/v6.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **codecov--codecov-action/v6.0.1** was hardened automatically. 9 finding(s) were identified and resolved across 3 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): A ${{ }} expression is interpolated directly inside a run: shell command string. In action.yml, the 'Set safe directory' step uses `${{ github.workspace }}` directly in a git config command. In .github/workflows/main.yml, two 'Verify dependency check failed' steps use `${{ steps.codecov-upload.outcome }}` directly inside an if-condition in a shell script. Any ${{ }} expression in a run: block is a script-injection risk because YAML template substitution happens before the shell ever sees the value.

Locations:

- `action.yml:196`
- `.github/workflows/main.yml:148`
- `.github/workflows/main.yml:213`

### github-env-injection (severity: high)

Multiple run: steps in action.yml write values derived from untrusted inputs or github context to $GITHUB_ENV without the required sanitization step (printf '%s' ... | tr -d '\n\r'). (1) 'Get and set token' step writes `echo "CC_TOKEN=$CC_OIDC_TOKEN" >> "$GITHUB_ENV"` where CC_OIDC_TOKEN comes from steps.oidc.outputs.result (untrusted), and `echo "CC_TOKEN=$INPUT_CODECOV_TOKEN" >> "$GITHUB_ENV"` where INPUT_CODECOV_TOKEN comes from env.CODECOV_TOKEN (workflow-controlled). (2) 'Override branch for forks' step writes `echo "TOKENLESS=$TOKENLESS" >> "$GITHUB_ENV"` and `echo "CC_BRANCH=$CC_BRANCH" >> "$GITHUB_ENV"` where the values derive from github.event.pull_request.head.label (attacker-controlled on PRs). (3) 'Override commits and pr for pull requests' step writes `echo "CC_SHA=$CC_SHA" >> "$GITHUB_ENV"` and `echo "CC_PR=$CC_PR" >> "$GITHUB_ENV"` where values derive from inputs.override_commit, inputs.override_pr, github.event.pull_request.head.sha, and github.event.number — all untrusted. None of these writes are preceded by the required newline-stripping sanitization.

Locations:

- `action.yml:233`
- `action.yml:237`
- `action.yml:241`
- `action.yml:261`
- `action.yml:264`
- `action.yml:279`
- `action.yml:280`

### unpinned-uses (severity: high)

Multiple workflow files and action.yml reference external actions using mutable version tags instead of full 40-character commit SHAs. Unpinned references are vulnerable to supply-chain attacks if the tag is moved. Affected references: main.yml — actions/checkout@v5.0.0 (used 6 times). codeql-analysis.yml — actions/checkout@v5.0.0, github/codeql-action/init@v3.30.0, github/codeql-action/autobuild@v3.30.0, github/codeql-action/analyze@v3.30.0. scorecards-analysis.yml — actions/checkout@v5.0.0, github/codeql-action/upload-sarif@v3.30.0.

Locations:

- `.github/workflows/main.yml:10`
- `.github/workflows/codeql-analysis.yml:30`
- `.github/workflows/codeql-analysis.yml:35`
- `.github/workflows/codeql-analysis.yml:42`
- `.github/workflows/codeql-analysis.yml:52`
- `.github/workflows/scorecards-analysis.yml:21`
- `.github/workflows/scorecards-analysis.yml:52`

### missing-permissions (severity: medium)

The workflow file enforce-license-compliance.yml has no top-level permissions: key and no job-level permissions: key on any of its jobs. Without explicit permissions, the workflow inherits the default repository permissions (which may be write-all for private repos), granting the GITHUB_TOKEN broader access than necessary.

Locations:

- `.github/workflows/enforce-license-compliance.yml:1`

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

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions, static-unsanitized-env-write

**Notes:**

Fixed all findings:

1. script-injection: Moved ${{ github.workspace }} in action.yml 'Set safe directory' step into env var WORKSPACE. Moved ${{ steps.codecov-upload.outcome }} in both main.yml 'Verify dependency check failed' steps into env var CODECOV_OUTCOME.

2. github-env-injection / static-unsanitized-env-write: Added printf '%s' ... | tr -d '\n\r' sanitization before all GITHUB_ENV writes in action.yml: CC_OIDC_TOKEN and INPUT_CODECOV_TOKEN in 'Get and set token'; TOKENLESS and CC_BRANCH in 'Override branch for forks'; CC_SHA and CC_PR in 'Override commits and pr for pull requests'.

3. unpinned-uses: Pinned all unpinned action references to full 40-char SHAs with tag comments: actions/checkout@v5.0.0 → @08c6903cd8c0fde910a37f88322edcfb5dd907a8 (8 total occurrences across main.yml, codeql-analysis.yml, scorecards-analysis.yml); github/codeql-action/{init,autobuild,analyze,upload-sarif}@v3.30.0 → @2d92b76c45b91eb80fc44c74ce3fce0ee94e8f9d.

4. missing-permissions: Added permissions: {} to enforce-license-compliance.yml.

### Iteration 2

**Fixes applied:** broad-permissions

**Notes:**

Replaced top-level `permissions: read-all` with `permissions: {}` in .github/workflows/scorecards-analysis.yml. The `analysis` job already had explicit minimal job-level permissions (`security-events: write`, `id-token: write`, `actions: read`, `contents: read`), so those remain unchanged and provide the necessary access without the overly broad top-level grant.

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the incomplete sanitization in the 'Get and set token' step's else branch in action.yml (line 227). Changed `CC_TOKEN=$(echo "$INPUT_TOKEN" | tr -d '\n')` to `safe_token=$(printf '%s' "$INPUT_TOKEN" | tr -d '\n\r')` and updated the subsequent echo to use `safe_token`. This uses printf '%s' instead of echo (avoiding escape sequence interpretation) and strips both \n and \r characters to prevent carriage return injection into $GITHUB_ENV. The fix is now consistent with the sanitization pattern used in the other branches of the same conditional block.

