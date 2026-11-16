<!-- markdownlint-disable -->

# Hardening Report: jenseng--dynamic-uses/v1.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `374911c4be584df24e6da57af2d990570ef0f859`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **jenseng--dynamic-uses/v1.1.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. Sub-rule (a): The 'Setup' step in action.yml contains ${{ }} expressions directly interpolated inside a run: shell heredoc block. Specifically, `${{ '\$' }}` (used to produce a literal backslash-dollar) and `${{'$'}}{{'\$'}}{{` (used inside a sed command) are GitHub Actions expressions embedded directly in the run: script string. Per the script-injection check, ANY ${{ ... }} expression directly inside a run: block is a finding, regardless of whether the value is attacker-controlled, because YAML template substitution occurs before the shell ever sees the string.

Locations:

- `action.yml:30`
- `action.yml:35`

### prior-fix (severity: high)

Carried forward from this version's previous harden, which fixed it; fix it again unless the code no longer has the problem. The previous harden changed action.yml without recording a finding for it, and this analysis reported nothing there. Re-apply its security changes unless the code no longer needs them. The previous patch:
```diff
diff --git a/action.yml b/action.yml
index 5682675..b3b1a32 100644
--- a/action.yml
+++ b/action.yml
@@ -16,6 +16,23 @@ outputs:
 runs:
   using: composite
   steps:
     - name: Setup
       shell: bash
       env:
@@ -23,10 +40,11 @@ runs:
         with: ${{ inputs.with }}
       run: |
         mkdir -p ./.tmp-dynamic-uses &&
+        D='$' &&
         cat <<DYNAMIC_USES_EOF >./.tmp-dynamic-uses/action.yml
         outputs:
           outputs:
-            value: ${{ '\$' }}{{ toJSON(steps.run.outputs) }}
+            value: ${D}{{ toJSON(steps.run.outputs) }}
         runs:
           using: composite
           steps:
@@ -36,7 +54,7 @@ runs:
             id: run
             uses: '$(echo "$uses" | sed "s/'/''/g")'
             with:
-        $(echo "$with" | sed -E "s/^/      /; s/\\\$\\{\\{/\${{'$'}}{{'\$'}}{{/g")
+        $(echo "$with" | sed -E 's/^/      /; s/\\\$\{\{/\\\\&/g')
         DYNAMIC_USES_EOF
     - name: Run
       id: run

```

Locations:

- `action.yml`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, prior-fix

**Notes:**

Removed every ${{ }} expression from the Setup step's run: script in action.yml, and kept the action working the same way.
- The output line now uses a shell variable D='$' that the shell sets itself. The heredoc is unquoted (DYNAMIC_USES_EOF), so ${D}{{ toJSON(steps.run.outputs) }} still writes the literal ${{ ... }} into the generated action.yml, just as upstream's ${{ '\$' }} did.
- The escaping of `with` now happens before the heredoc: escaped_with=$(echo "$with" | sed -E "s/^/      /; s/\\${D}\\{\\{/${D}{{'${D}'}}{{/g"). The shell turns this into the same sed command upstream ran: s/^/      /; s/\$\{\{/${{'$'}}{{/g. It indents each line and replaces every ${{ with ${{'$'}}{{, so expressions in the input are not evaluated a second time. The heredoc then writes ${escaped_with}.
- I did not copy the previous harden's sed change. As I read it, it replaced ${{ with \${{, but GitHub does not treat a backslash as an escape there, so that changed behaviour.
The ${{ }} expressions left in action.yml are only in env: and the outputs value. I worked out the shell quoting by hand and have not run the action.

