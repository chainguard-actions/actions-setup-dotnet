<!-- markdownlint-disable -->

# Hardening Report: actions--setup-dotnet/v6.0.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--setup-dotnet/v6.0.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Five workflow files reference `actions/reusable-workflows` using the mutable `@main` branch ref instead of a pinned 40-character commit SHA. This exposes the workflows to supply-chain attacks if the upstream repository is compromised or the branch is force-pushed. Failing references: `actions/reusable-workflows/.github/workflows/basic-validation.yml@main`, `actions/reusable-workflows/.github/workflows/check-dist.yml@main`, `actions/reusable-workflows/.github/workflows/codeql-analysis.yml@main`, `actions/reusable-workflows/.github/workflows/licensed.yml@main`, `actions/reusable-workflows/.github/workflows/update-config-files.yml@main`.

Locations:

- `.github/workflows/basic-validation.yml:17`
- `.github/workflows/check-dist.yml:16`
- `.github/workflows/codeql-analysis.yml:14`
- `.github/workflows/licensed.yml:14`
- `.github/workflows/update-config-files.yml:12`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate GitHub Actions expressions (`${{ ... }}`) into shell command strings (sub-rule a). This allows template substitution to inject arbitrary shell metacharacters before the shell ever parses the command. Affected expressions include: `${{ runner.os }}` (passed as a positional argument to a PowerShell script in every job's 'Clear toolcache' step), `${{ matrix.dotnet-version }}` and `${{ matrix.lower-version }}`/`${{ matrix.higher-version }}` (interpolated into PowerShell -Patterns arguments), and `${{steps.step1.outputs.dotnet-version}}`/`${{steps.step2.outputs.dotnet-version}}` (interpolated inside a PowerShell string comparison). All of these should be passed via `env:` variables and referenced as `$ENV_VAR` (double-quoted) in the shell script.

Locations:

- `.github/workflows/e2e-tests.yml:27`
- `.github/workflows/e2e-tests.yml:57`
- `.github/workflows/e2e-tests.yml:338`
- `.github/workflows/e2e-tests.yml:349`
- `.github/workflows/e2e-tests.yml:374`
- `.github/workflows/e2e-tests.yml:383`
- `.github/workflows/test-dotnet.yml:28`
- `.github/workflows/test-dotnet.yml:38`

### hardcoded-credentials (severity: high)

The workflow file `e2e-tests.yml` contains the literal hardcoded value `NUGET_AUTH_TOKEN: NOTATOKEN` in three separate steps. Although labeled as a placeholder, this is a non-expression literal string assigned to a token-named environment variable. It should be replaced with a GitHub Actions secret expression such as `${{ secrets.NUGET_AUTH_TOKEN }}` or removed if not needed.

Locations:

- `.github/workflows/e2e-tests.yml:68`
- `.github/workflows/e2e-tests.yml:452`
- `.github/workflows/e2e-tests.yml:474`

### missing-permissions (severity: medium)

Seven workflow files have no top-level `permissions:` block and no job-level `permissions:` blocks on any of their jobs. Without explicit permissions, workflows inherit the repository's default token permissions (which may be `write-all`), granting unnecessarily broad access. Affected files: `basic-validation.yml`, `check-dist.yml`, `codeql-analysis.yml`, `e2e-tests.yml`, `licensed.yml`, `test-dotnet.yml`, `update-config-files.yml`. Each should declare minimal required permissions (e.g. `permissions: read-all` or specific scopes).

Locations:

- `.github/workflows/basic-validation.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/e2e-tests.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/test-dotnet.yml:1`
- `.github/workflows/update-config-files.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, hardcoded-credentials, missing-permissions

**Notes:**

Fixed all four findings across 7 workflow files:

1. **unpinned-uses**: Pinned all 5 `actions/reusable-workflows@main` references to SHA `d468c63c53c1184242904d1a3ac74fd1081f36c8` with `# main` comment in basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, and update-config-files.yml.

2. **script-injection**: Moved all ${{ }} expressions out of `run:` blocks into `env:` blocks:
   - 24 occurrences of `${{ runner.os }}` in e2e-tests.yml clear-toolcache steps → `RUNNER_OS` env var, using `$env:RUNNER_OS` in PowerShell
   - `${{steps.step1.outputs.dotnet-version}}` and `${{steps.step2.outputs.dotnet-version}}` → `DOTNET_VERSION_OUTPUT` env var
   - `${{ matrix.lower-version }}` and `${{ matrix.higher-version }}` in verify-dotnet.ps1 calls → `LOWER_VERSION`/`HIGHER_VERSION` env vars
   - `${{ runner.os }}` and `${{ matrix.dotnet-version }}` in test-dotnet.yml → `RUNNER_OS`/`DOTNET_VERSION` env vars

3. **hardcoded-credentials**: Replaced all 3 occurrences of `NUGET_AUTH_TOKEN: NOTATOKEN` with `NUGET_AUTH_TOKEN: ${{ secrets.NUGET_AUTH_TOKEN }}`.

4. **missing-permissions**: Added `permissions: contents: read` top-level block to all 7 affected workflow files.

