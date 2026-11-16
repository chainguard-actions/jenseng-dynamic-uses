<!-- markdownlint-disable -->

# Hardening Report: jenseng--dynamic-uses/v1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **jenseng--dynamic-uses/v1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) direct expression interpolation: the 'Setup' composite step's run: block contains a GitHub Actions expression directly in the shell script: `value: ${{ '\$' }}{{ toJSON(steps.run.outputs) }}`. The expression is a constant string literal and cannot be controlled by an attacker. It is still reported because the check flags every ${{ }} expression inside a run: script. To fix it, write the literal directly in the heredoc (for example an escaped `\${{`) or pass it in through an env var. The other workflow-controlled values ($uses, $with) are double-quoted inside command substitutions, so they pass rule (b).

Locations:

- `action.yml:27`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection

**Notes:**

Removed GitHub expressions from the Setup step's run: script while keeping the generated file's text the same. Line 27: replaced `${{ '\$' }}{{` with `$(printf '%s' '$'){{` (the heredoc is unquoted, so the command substitution prints a literal $ and the generated action.yml still gets `${{ toJSON(steps.run.outputs) }}`; writing `\${{` directly would have let GitHub evaluate the expression during Setup). Line 38 (sed escaping of `with`): replaced `\${{'$'}}{{'\$'}}{{` with `\$""{{'\$'}}{{`, which the shell turns into the same `${{'$'}}{{` replacement. No env var holding a bare metacharacter was added. Not run; equivalence checked by reading the shell quoting rules.

