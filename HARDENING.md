<!-- markdownlint-disable -->

# Hardening Report: actions--setup-dotnet/v5.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **actions--setup-dotnet/v5.2.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files use mutable tag or branch references instead of pinned 40-character SHA commit hashes. Affected references include: `actions/checkout@v6` (tag) in e2e-tests.yml and test-dotnet.yml; `actions/reusable-workflows/...@main` (branch) in basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, and update-config-files.yml; `actions/publish-immutable-action@v0.0.4` (tag) in publish-immutable-actions.yml; `actions/publish-action@v0.4.0` (tag) in release-new-action-version.yml. These mutable refs can be silently redirected to malicious code.

Locations:

- `.github/workflows/e2e-tests.yml:30`
- `.github/workflows/test-dotnet.yml:28`
- `.github/workflows/publish-immutable-actions.yml:14`
- `.github/workflows/publish-immutable-actions.yml:17`
- `.github/workflows/release-new-action-version.yml:22`
- `.github/workflows/basic-validation.yml:14`
- `.github/workflows/check-dist.yml:14`
- `.github/workflows/codeql-analysis.yml:13`
- `.github/workflows/licensed.yml:12`
- `.github/workflows/update-config-files.yml:12`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate GitHub Actions expressions inside shell commands, violating rule (a). In e2e-tests.yml, `${{ runner.os }}` is interpolated in every 'Clear toolcache' step (e.g. `run: __tests__/clear-toolcache.ps1 ${{ runner.os }}`). Additionally, `${{ matrix.lower-version }}` and `${{ matrix.higher-version }}` are interpolated in run: blocks (e.g. `run: __tests__/verify-dotnet.ps1 -Patterns "^${{ matrix.lower-version }}$"`). In multiline run: blocks, `${{steps.step1.outputs.dotnet-version}}` and `${{steps.step2.outputs.dotnet-version}}` are interpolated directly. In test-dotnet.yml, `${{ matrix.dotnet-version }}` is interpolated in a run: block. All of these bypass shell quoting and allow expression values to be interpreted as shell commands.

Locations:

- `.github/workflows/e2e-tests.yml:33`
- `.github/workflows/e2e-tests.yml:299`
- `.github/workflows/e2e-tests.yml:305`
- `.github/workflows/e2e-tests.yml:280`
- `.github/workflows/e2e-tests.yml:315`
- `.github/workflows/test-dotnet.yml:36`

### missing-permissions (severity: medium)

Several workflow files have no top-level `permissions:` key and no job-level `permissions:` key on any job. Without explicit permissions, workflows inherit the default repository permissions (which may be broad). Affected files: e2e-tests.yml, basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, test-dotnet.yml, update-config-files.yml.

Locations:

- `.github/workflows/e2e-tests.yml:1`
- `.github/workflows/basic-validation.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/test-dotnet.yml:1`
- `.github/workflows/update-config-files.yml:1`

### hardcoded-credentials (severity: high)

The literal value `NOTATOKEN` is assigned to `NUGET_AUTH_TOKEN` in three places in e2e-tests.yml (e.g. `NUGET_AUTH_TOKEN: NOTATOKEN`). While this appears to be a test placeholder, it is a hardcoded non-expression literal assigned to a token-named variable, matching the hardcoded-credentials pattern.

Locations:

- `.github/workflows/e2e-tests.yml:79`
- `.github/workflows/e2e-tests.yml:330`
- `.github/workflows/e2e-tests.yml:358`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, missing-permissions, hardcoded-credentials

**Notes:**

Fixed all findings across 8 workflow files: (1) Pinned all mutable action refs to full SHA hashes: actions/checkout@v6 -> @df4cb1c069e1874edd31b4311f1884172cec0e10, actions/reusable-workflows@main -> @09976383aa8780d306ee271bd21bb77a54fad474, actions/publish-immutable-action@v0.0.4 -> @4bc8754ffc40f27910afb20287dbbbb675a4e978, actions/publish-action@v0.4.0 -> @23f4c6f12633a2da8f44938b71fde9afec138fb4. (2) Fixed script-injection in e2e-tests.yml and test-dotnet.yml by moving ${{ runner.os }}, ${{ matrix.lower-version }}, ${{ matrix.higher-version }}, ${{ steps.step1.outputs.dotnet-version }}, ${{ steps.step2.outputs.dotnet-version }}, and ${{ matrix.dotnet-version }} into env: blocks and referencing them as $env:VAR_NAME in PowerShell run: blocks. (3) Added top-level 'permissions: contents: read' to e2e-tests.yml, basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, test-dotnet.yml, and update-config-files.yml. (4) Replaced hardcoded NUGET_AUTH_TOKEN: NOTATOKEN with NUGET_AUTH_TOKEN: ${{ secrets.NUGET_AUTH_TOKEN }} in all three occurrences in e2e-tests.yml.

