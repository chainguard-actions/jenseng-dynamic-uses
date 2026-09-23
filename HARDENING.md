<!-- markdownlint-disable -->

# Hardening Report: jenseng--dynamic-uses/v1.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **jenseng--dynamic-uses/v1.1.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): The 'Setup' step in action.yml contains ${{ }} expressions directly interpolated inside a run: shell heredoc block. Specifically, `${{ '\$' }}` (used to produce a literal backslash-dollar) and `${{'$'}}{{'\$'}}{{` (used inside a sed command) are GitHub Actions expressions embedded directly in the run: script string. Per the script-injection check, ANY ${{ ... }} expression directly inside a run: block is a finding, regardless of whether the value is attacker-controlled, because YAML template substitution occurs before the shell ever sees the string.

Locations:

- `action.yml:30`
- `action.yml:35`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings in action.yml:

1. Line 30: Replaced `${{ '\\$' }}{{ toJSON(steps.run.outputs) }}` with `${D}{{ toJSON(steps.run.outputs) }}`. Added `D='$' &&` before the heredoc to define a shell variable. GHA does not process `${D}{{` as an expression (not `${{`), and bash expands `${D}` to `$` in the unquoted heredoc, producing `${{ toJSON(steps.run.outputs) }}` in the generated action.yml file.

2. Line 35: Replaced the double-quoted sed command containing `${{ '$' }}` and `${{ '\\$' }}` GHA expressions with a single-quoted sed command `'s/\\\$\{\{/\\\\&/g'`. After heredoc processing, bash executes `sed -E 's/\$\{\{/\\&/g'` in single-quote context, where pattern `\$\{\{` matches `${{` and replacement `\\&` outputs `\${{` (backslash + matched text).

The three remaining `${{ }}` occurrences are in the outputs: section and env: block (not in run: blocks), which are appropriate locations.

