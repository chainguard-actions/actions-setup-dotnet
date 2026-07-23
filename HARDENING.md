<!-- markdownlint-disable -->

# Hardening Report: actions--setup-dotnet/v5.3.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **actions--setup-dotnet/v5.3.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Every `uses:` reference across all workflow files uses a mutable tag or branch instead of a pinned 40-character SHA commit hash. This exposes the workflows to supply-chain attacks if the referenced action or reusable workflow is compromised or its tag is moved. Failing references include: `actions/checkout@v6` (all jobs in e2e-tests.yml, publish-immutable-actions.yml, test-dotnet.yml), `actions/reusable-workflows/...@main` (basic-validation.yml, check-dist.yml, codeql-analysis.yml, licensed.yml, update-config-files.yml), `actions/publish-immutable-action@v0.0.4` (publish-immutable-actions.yml), and `actions/publish-action@v0.4.0` (release-new-action-version.yml).

Locations:

- `.github/workflows/basic-validation.yml:16`
- `.github/workflows/check-dist.yml:15`
- `.github/workflows/codeql-analysis.yml:13`
- `.github/workflows/e2e-tests.yml:27`
- `.github/workflows/licensed.yml:14`
- `.github/workflows/publish-immutable-actions.yml:14`
- `.github/workflows/publish-immutable-actions.yml:17`
- `.github/workflows/release-new-action-version.yml:22`
- `.github/workflows/test-dotnet.yml:27`
- `.github/workflows/update-config-files.yml:11`

### hardcoded-credentials (severity: high)

The literal value `NOTATOKEN` is assigned to `NUGET_AUTH_TOKEN` in three separate jobs in e2e-tests.yml. Although this appears to be a placeholder, it is a hardcoded non-expression literal string assigned to a name containing `token`, matching the hardcoded-credentials pattern. It should be replaced with a GitHub Actions secret expression such as `${{ secrets.NUGET_AUTH_TOKEN }}`.

Locations:

- `.github/workflows/e2e-tests.yml:75`
- `.github/workflows/e2e-tests.yml:430`
- `.github/workflows/e2e-tests.yml:452`

### missing-permissions (severity: medium)

The following workflow files have no top-level `permissions:` key and no per-job `permissions:` keys. Without explicit permissions, workflows inherit the default repository token permissions (which may be `write-all` depending on repository settings), violating the principle of least privilege: basic-validation.yml, check-dist.yml, codeql-analysis.yml, e2e-tests.yml, licensed.yml, test-dotnet.yml, update-config-files.yml.

Locations:

- `.github/workflows/basic-validation.yml:1`
- `.github/workflows/check-dist.yml:1`
- `.github/workflows/codeql-analysis.yml:1`
- `.github/workflows/e2e-tests.yml:1`
- `.github/workflows/licensed.yml:1`
- `.github/workflows/test-dotnet.yml:1`
- `.github/workflows/update-config-files.yml:1`

### script-injection (severity: high)

Multiple `run:` blocks directly interpolate GitHub Actions expressions via `${{ ... }}` inside shell command strings (sub-rule a). Even though `runner.os` and `matrix.*` values are not directly attacker-controlled, any `${{ ... }}` expression inside a `run:` block is a script-injection risk because the value is substituted into the shell command string before the shell parses it. Affected patterns: (1) `run: __tests__/clear-toolcache.ps1 ${{ runner.os }}` — appears in 25+ steps across e2e-tests.yml and test-dotnet.yml; (2) `run: __tests__/verify-dotnet.ps1 -Patterns "^${{ matrix.dotnet-version }}"` in test-dotnet.yml; (3) `run: __tests__/verify-dotnet.ps1 -Patterns "^${{ matrix.lower-version }}$", "^${{ matrix.higher-version }}$"` in e2e-tests.yml. All expressions should be moved to `env:` variables and referenced as quoted shell variables.

Locations:

- `.github/workflows/e2e-tests.yml:30`
- `.github/workflows/e2e-tests.yml:62`
- `.github/workflows/e2e-tests.yml:99`
- `.github/workflows/test-dotnet.yml:30`
- `.github/workflows/test-dotnet.yml:38`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, hardcoded-credentials, missing-permissions, script-injection

**Notes:**

Fixed all four findings across 9 workflow files:

1. unpinned-uses: Pinned all action references to full SHA hashes - actions/checkout@v6→d23441a48e516b6c34aea4fa41551a30e30af803, actions/reusable-workflows@main→4735e71081024a944852f4ab9d1495b6dd2de8f2, actions/publish-immutable-action@v0.0.4→4bc8754ffc40f27910afb20287dbbbb675a4e978, actions/publish-action@v0.4.0→23f4c6f12633a2da8f44938b71fde9afec138fb4.

2. hardcoded-credentials: Replaced all 3 occurrences of NUGET_AUTH_TOKEN: NOTATOKEN with NUGET_AUTH_TOKEN: ${{ secrets.NUGET_AUTH_TOKEN }} in e2e-tests.yml.

3. missing-permissions: Added permissions: {} top-level block to basic-validation.yml, check-dist.yml, codeql-analysis.yml, e2e-tests.yml, licensed.yml, test-dotnet.yml, and update-config-files.yml.

4. script-injection: Moved all ${{ }} expressions out of run: blocks into env: blocks. runner.os moved to RUNNER_OS env var (referenced as $env:RUNNER_OS in PowerShell) across all 20+ clear-toolcache steps. matrix.lower-version/higher-version moved to LOWER_VERSION/HIGHER_VERSION env vars. matrix.dotnet-version moved to DOTNET_VERSION env var. steps.stepN.outputs.dotnet-version moved to DOTNET_VERSION_OUTPUT env var.

