<!-- markdownlint-disable -->

# Hardening Report: actions--setup-dotnet/v5.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **actions--setup-dotnet/v5.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

All workflow files reference actions using mutable tags or branch names instead of pinned 40-character SHA digests, making them vulnerable to supply-chain attacks. Failing references: basic-validation.yml uses actions/reusable-workflows/...@main; check-dist.yml uses actions/reusable-workflows/...@main; codeql-analysis.yml uses actions/reusable-workflows/...@main; e2e-tests.yml uses actions/checkout@v6 (repeated across all 25+ jobs); licensed.yml uses actions/reusable-workflows/...@main; publish-immutable-actions.yml uses actions/checkout@v6 and actions/publish-immutable-action@v0.0.4; release-new-action-version.yml uses actions/publish-action@v0.4.0; test-dotnet.yml uses actions/checkout@v6; update-config-files.yml uses actions/reusable-workflows/...@main.

Locations:

- `.github/workflows/basic-validation.yml:15`
- `.github/workflows/check-dist.yml:14`
- `.github/workflows/codeql-analysis.yml:12`
- `.github/workflows/e2e-tests.yml:30`
- `.github/workflows/licensed.yml:13`
- `.github/workflows/publish-immutable-actions.yml:15`
- `.github/workflows/publish-immutable-actions.yml:17`
- `.github/workflows/release-new-action-version.yml:21`
- `.github/workflows/test-dotnet.yml:22`
- `.github/workflows/update-config-files.yml:9`

### missing-permissions (severity: medium)

Seven workflow files have no top-level permissions: key and no job-level permissions: key on any job, meaning they run with the default (potentially broad write) GITHUB_TOKEN permissions. Affected files: basic-validation.yml, check-dist.yml, codeql-analysis.yml, e2e-tests.yml, licensed.yml, test-dotnet.yml, update-config-files.yml.

Locations:

- `.github/workflows/basic-validation.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/e2e-tests.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/test-dotnet.yml:1`
- `.github/workflows/update-config-files.yml:1`

### script-injection (severity: high)

Multiple run: blocks directly interpolate ${{ }} expressions into shell commands (sub-rule a), allowing template substitution before the shell parses the string. In e2e-tests.yml: every job's 'Clear toolcache' step uses `run: __tests__/clear-toolcache.ps1 ${{ runner.os }}` (25 occurrences); the 'Verify value of the dotnet-version output' steps embed `${{steps.step1.outputs.dotnet-version}}` and `${{steps.step2.outputs.dotnet-version}}` directly in run: blocks; the 'Verify dotnet (lower/higher version)' steps embed `${{ matrix.lower-version }}` and `${{ matrix.higher-version }}` directly. In test-dotnet.yml: `run: __tests__/clear-toolcache.ps1 ${{ runner.os }}` and `run: __tests__/verify-dotnet.ps1 -Patterns "^${{ matrix.dotnet-version }}"` directly interpolate expressions.

Locations:

- `.github/workflows/e2e-tests.yml:33`
- `.github/workflows/e2e-tests.yml:393`
- `.github/workflows/e2e-tests.yml:430`
- `.github/workflows/e2e-tests.yml:462`
- `.github/workflows/e2e-tests.yml:466`
- `.github/workflows/test-dotnet.yml:27`
- `.github/workflows/test-dotnet.yml:35`

### hardcoded-credentials (severity: high)

e2e-tests.yml contains three occurrences of a hardcoded literal token value: `NUGET_AUTH_TOKEN: NOTATOKEN`. The name contains 'token' and the value is a non-expression alphanumeric literal (not a ${{ secrets.* }} reference). This should be replaced with a proper secrets expression such as `${{ secrets.NUGET_AUTH_TOKEN }}`.

Locations:

- `.github/workflows/e2e-tests.yml:88`
- `.github/workflows/e2e-tests.yml:388`
- `.github/workflows/e2e-tests.yml:406`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, hardcoded-credentials

**Notes:**

Fixed all four findings across 9 workflow files:

1. unpinned-uses: Pinned all mutable action references to full 40-char SHA digests: actions/checkout@v6→df4cb1c, actions/reusable-workflows@main→09976383, actions/publish-immutable-action@v0.0.4→4bc8754, actions/publish-action@v0.4.0→23f4c6f.

2. missing-permissions: Added `permissions: {}` at the top level of basic-validation.yml, check-dist.yml, codeql-analysis.yml, e2e-tests.yml, licensed.yml, test-dotnet.yml, and update-config-files.yml.

3. script-injection: Moved all ${{ }} expressions out of run: shell strings into step env: blocks. In e2e-tests.yml: all 25+ clear-toolcache calls now use RUNNER_OS env var ($env:RUNNER_OS in pwsh); dotnet-version output checks use DOTNET_VERSION_OUTPUT env var; sequential version verification uses LOWER_VERSION/HIGHER_VERSION env vars. In test-dotnet.yml: clear-toolcache uses RUNNER_OS env var; verify-dotnet uses DOTNET_VERSION env var.

4. hardcoded-credentials: Replaced all 3 occurrences of `NUGET_AUTH_TOKEN: NOTATOKEN` with `NUGET_AUTH_TOKEN: ${{ secrets.NUGET_AUTH_TOKEN }}` in e2e-tests.yml (test-setup-full-version, test-proxy, test-bypass-proxy jobs).

