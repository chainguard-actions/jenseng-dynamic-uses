<!-- markdownlint-disable -->

# Hardening Report: jenseng--dynamic-uses/v1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **jenseng--dynamic-uses/v1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow steps use actions/checkout@v6, which is a mutable tag reference rather than a pinned 40-character commit SHA. If the tag is moved or compromised, the action will silently execute different code. All uses: references should be pinned to a full SHA (e.g. actions/checkout@11bd71901bbe5b1630ceea73d27597364c9af683 # v6).

Locations:

- `.github/workflows/release.yml:27`
- `.github/workflows/test.yml:16`
- `.github/workflows/test.yml:57`

### missing-permissions (severity: medium)

These workflow files have no top-level permissions: key and no job-level permissions: block on any of their jobs. Without explicit permissions, the GITHUB_TOKEN is granted its default (potentially broad) access. Each workflow should declare minimal required permissions explicitly.

Locations:

- `.github/workflows/cicd.yml:1`
- `.github/workflows/pull-request.yml:1`
- `.github/workflows/test.yml:1`

### script-injection (severity: high)

Sub-rule (b): In the 'Determine version' step of release.yml, the env var $version (sourced from inputs.version via `env: version: ${{ inputs.version }}`) is expanded unquoted inside the run: shell script. Unquoted expansions like `[[ $version == patch ]]`, `echo $last_version`, and `full_version="$version"` allow an attacker-controlled value containing shell metacharacters (`;`, `|`, `&`, `$(...)`, etc.) to be interpreted by the shell. The variable must be double-quoted everywhere it is used: `[[ "$version" == patch ]]`, `echo "$last_version"`, etc.

Locations:

- `.github/workflows/release.yml:35`

### github-env-injection (severity: high)

In the 'Determine version' step of release.yml, the variable full_version (derived from $version, which is set from the untrusted workflow_dispatch input inputs.version) is written directly to $GITHUB_ENV without sanitization: `echo "full_version=$full_version" >> "$GITHUB_ENV"`. An attacker-controlled value containing newlines could inject additional environment variable assignments (e.g. `ACTIONS_RUNTIME_TOKEN=...`). The required sanitization step `safe=$(printf '%s' "$full_version" | tr -d '\n\r')` must be applied before every write to $GITHUB_ENV.

Locations:

- `.github/workflows/release.yml:46`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, missing-permissions, script-injection, github-env-injection

**Notes:**

1. unpinned-uses: Pinned all three actions/checkout@v6 references to full SHA d23441a48e516b6c34aea4fa41551a30e30af803 # v6 in release.yml and test.yml. 2. missing-permissions: Added `permissions: {}` top-level blocks to cicd.yml, pull-request.yml, and test.yml (release.yml already had a job-level `permissions: contents: write`). 3. script-injection: Double-quoted all uses of $version and $last_version in the 'Determine version' step of release.yml to prevent shell metacharacter injection. 4. github-env-injection: Added sanitization step `safe_full_version=$(printf '%s' "$full_version" | tr -d '\n\r')` before writing to $GITHUB_ENV to prevent newline injection attacks.

