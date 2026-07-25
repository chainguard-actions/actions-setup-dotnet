<!-- markdownlint-disable -->

# Hardening Report: actions--setup-dotnet/v5.4.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--setup-dotnet/v5.4.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference reusable workflows using the mutable '@main' branch ref instead of a pinned 40-character commit SHA. This exposes the action to supply-chain attacks if the upstream repository is compromised. Affected references: 'actions/reusable-workflows/.github/workflows/basic-validation.yml@main', 'actions/reusable-workflows/.github/workflows/check-dist.yml@main', 'actions/reusable-workflows/.github/workflows/codeql-analysis.yml@main', 'actions/reusable-workflows/.github/workflows/licensed.yml@main', 'actions/reusable-workflows/.github/workflows/update-config-files.yml@main'.

Locations:

- `.github/workflows/basic-validation.yml:15`
- `.github/workflows/check-dist.yml:15`
- `.github/workflows/codeql-analysis.yml:13`
- `.github/workflows/licensed.yml:14`
- `.github/workflows/update-config-files.yml:10`

### hardcoded-credentials (severity: high)

The workflow file contains a hardcoded literal token value 'NOTATOKEN' assigned to 'NUGET_AUTH_TOKEN'. Even though this appears to be a placeholder used for testing, it is a non-expression literal value assigned to a name containing 'token', which matches the hardcoded-credentials pattern. It should be replaced with a GitHub Actions secret expression (e.g. '${{ secrets.NUGET_AUTH_TOKEN }}').

Locations:

- `.github/workflows/e2e-tests.yml:68`
- `.github/workflows/e2e-tests.yml:453`
- `.github/workflows/e2e-tests.yml:476`

### missing-permissions (severity: medium)

These workflow files have no top-level 'permissions:' key and no job-level 'permissions:' keys on any of their jobs. Without explicit permissions, workflows run with the default (potentially broad) token permissions. Each file should declare minimal required permissions.

Locations:

- `.github/workflows/basic-validation.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/e2e-tests.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/test-dotnet.yml:1`
- `.github/workflows/update-config-files.yml:1`

### script-injection (severity: high)

Multiple 'run:' blocks directly interpolate GitHub Actions expressions (${{ ... }}) into shell command strings. This violates rule (a): any ${{ ... }} expression inside a run: block is a script-injection risk because the value is substituted into the shell command string before the shell parses it. Affected patterns include: (1) '${{ runner.os }}' interpolated directly into PowerShell script invocations (e.g. 'run: __tests__/clear-toolcache.ps1 ${{ runner.os }}') in e2e-tests.yml (24 occurrences) and test-dotnet.yml; (2) '${{ matrix.lower-version }}' and '${{ matrix.higher-version }}' interpolated into run: blocks in e2e-tests.yml; (3) '${{ matrix.dotnet-version }}' interpolated into a run: block in test-dotnet.yml; (4) '${{ steps.step1.outputs.dotnet-version }}' and '${{ steps.step2.outputs.dotnet-version }}' interpolated into run: blocks in e2e-tests.yml. These should be passed via env: variables and referenced as shell variables (e.g. $RUNNER_OS, $MATRIX_DOTNET_VERSION).

Locations:

- `.github/workflows/e2e-tests.yml:30`
- `.github/workflows/e2e-tests.yml:509`
- `.github/workflows/e2e-tests.yml:534`
- `.github/workflows/test-dotnet.yml:33`
- `.github/workflows/test-dotnet.yml:42`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, hardcoded-credentials, missing-permissions, script-injection

**Notes:**

Fixed all four findings across 7 workflow files:

1. unpinned-uses: Pinned all 5 reusable workflow references from '@main' to SHA '4735e71081024a944852f4ab9d1495b6dd2de8f2 # main' in basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, and update-config-files.yml.

2. hardcoded-credentials: Replaced all 3 occurrences of 'NUGET_AUTH_TOKEN: NOTATOKEN' with 'NUGET_AUTH_TOKEN: ${{ secrets.NUGET_AUTH_TOKEN }}' in e2e-tests.yml (lines 68, 453, 476).

3. missing-permissions: Added 'permissions: {}' top-level block to all 7 workflow files: basic-validation.yml, check-dist.yml, codeql-analysis.yml, e2e-tests.yml, licensed.yml, test-dotnet.yml, update-config-files.yml.

4. script-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks:
   - 24 occurrences of ${{ runner.os }} in e2e-tests.yml → env: RUNNER_OS: ${{ runner.os }}, referenced as $env:RUNNER_OS in PowerShell
   - 1 occurrence of ${{ runner.os }} in test-dotnet.yml → same pattern
   - ${{ matrix.lower-version }} and ${{ matrix.higher-version }} in run: blocks → env: LOWER_VERSION / HIGHER_VERSION
   - ${{ matrix.dotnet-version }} in run: block in test-dotnet.yml → env: MATRIX_DOTNET_VERSION
   - ${{steps.step1.outputs.dotnet-version}} and ${{steps.step2.outputs.dotnet-version}} → env: STEP1_DOTNET_VERSION / STEP2_DOTNET_VERSION

