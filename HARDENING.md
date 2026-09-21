<!-- markdownlint-disable -->

# Hardening Report: jenseng--dynamic-uses/v1.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **jenseng--dynamic-uses/v1.1.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: The 'Setup' step in action.yml contains `${{ ... }}` expressions directly interpolated inside a `run:` shell heredoc block. Specifically, `${{ '\$' }}` appears on line 29 and `${{ '$' }}`/`${{ '\$' }}` appear on line 39. Per the script-injection check, ANY `${{ ... }}` expression directly inside a `run:` shell command string is a violation — the GitHub Actions runner performs YAML template substitution before the shell ever sees the script, meaning these expressions are expanded inline in the shell command. Even though these particular expressions evaluate to literal characters (used to escape `${{` in generated YAML), the pattern is flagged because the rule prohibits all direct `${{ }}` interpolation in `run:` blocks without exception.

Locations:

- `action.yml:29`
- `action.yml:39`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Fixed script-injection finding in action.yml by moving the ${{ '\\$' }} and ${{ '$' }} expressions from the run: heredoc block into the env: block as named variables (DOLLAR='$' and BACKSLASH_DOLLAR='\$'). These are then referenced as shell environment variables (${DOLLAR} and ${BACKSLASH_DOLLAR}) in the run: script, eliminating all ${{ }} interpolation from the run: block while preserving identical runtime behavior. The remaining ${{ }} expressions in the outputs: section and env: block are legitimate and not subject to the script-injection rule.

