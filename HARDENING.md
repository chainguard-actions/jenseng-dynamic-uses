<!-- markdownlint-disable -->

# Hardening Report: jenseng--dynamic-uses/v1.0.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **jenseng--dynamic-uses/v1.0.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `${{ inputs.* }}` expressions are directly interpolated inside a `run:` shell command block in the 'Setup' step. GitHub Actions performs template substitution on the entire `run:` string before the shell executes it, so even though these expressions appear inside a quoted heredoc (`cat <<'DYNAMIC_USES_EOF'`), the runner substitutes them first. An attacker-controlled `inputs.uses` or `inputs.with` value can inject arbitrary shell content into the script.

Offending lines:
  Line 34: `            uses: ${{ inputs.uses }}`
  Line 35: `            with: ${{ inputs.with || '{}' }}`

Fix: pass the inputs via environment variables and reference them as `$ENV_VAR` inside the heredoc, e.g.:
```yaml
env:
  USES_INPUT: ${{ inputs.uses }}
  WITH_INPUT: ${{ inputs.with || '{}' }}
run: |
  cat <<'EOF' >./.tmp-dynamic-uses/action.yml
  ...
            uses: ${USES_INPUT}
            with: ${WITH_INPUT}
  EOF
```

Locations:

- `action.yml:34`
- `action.yml:35`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.uses }}" appears directly in run: block of step "Setup"; move to env: map

Locations:

- `action.yml:34`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Moved ${{ inputs.uses }} and ${{ inputs.with || '{}' }} from the run: block into an env: block (USES_INPUT and WITH_INPUT). The quoted heredoc (<<'DYNAMIC_USES_EOF') writes the static parts of the generated action.yml, and then two printf statements append the dynamic 'uses:' and 'with:' lines using properly double-quoted shell variables ($USES_INPUT and $WITH_INPUT). This eliminates the script injection risk while preserving the action's functionality. The ${{ '$' }}{{ toJSON(steps.run.outputs) }} literal escape trick inside the heredoc was left unchanged as it is not an injection risk.

