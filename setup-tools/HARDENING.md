<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-tools/v6.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-tools/v6.2.0** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `${{ inputs.* }}` expressions are interpolated directly inside the `run:` shell script. On line 31, `CRANE_VERSION=${{ inputs.crane-version }}` and on line 44, `YJ_VERSION=${{ inputs.yj-version }}` are substituted by the Actions template engine before the shell parses the script. An attacker supplying a crafted version string (e.g. containing `;`, `$(...)`, or backticks) can execute arbitrary commands. These values must be passed via `env:` variables and then referenced as quoted shell variables (e.g. `"$CRANE_VERSION"`) — never interpolated directly with `${{ }}` inside a `run:` block. Sub-rule (b): `${CRANE_VERSION}` is also used unquoted inside a URL string on the curl command line, and `${YJ_VERSION}` is used unquoted in a `[[ ]]` comparison, both of which allow shell metacharacter injection from the (already-injected) value.

Locations:

- `action.yml:31`
- `action.yml:44`

### github-env-injection (severity: high)

Line 25 writes `echo "PATH=${HOME}/bin:${PATH}" >> "${GITHUB_ENV}"` without sanitizing `${PATH}`. In a composite action, `PATH` is an inherited process environment variable that can be set to any value by the calling workflow. If `PATH` contains embedded newline characters, the write to `$GITHUB_ENV` can inject arbitrary additional key=value pairs into the runner's environment (e.g. `MALICIOUS_VAR=evil`). The required sanitization step — `safe=$(printf '%s' "$PATH" | tr -d '\n\r')` — must be applied before the write.

Locations:

- `action.yml:25`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.crane-version }}" appears directly in run: block of step "Install additional buildpack management tools"; move to env: map

Locations:

- `action.yml:32`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.yj-version }}" appears directly in run: block of step "Install additional buildpack management tools"; move to env: map

Locations:

- `action.yml:45`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Fixed three categories of findings in hardened/action/action.yml:
1. script-injection / static-inline-injection (lines 31, 44): Moved `${{ inputs.crane-version }}` and `${{ inputs.yj-version }}` out of the `run:` shell script and into the step's `env:` block as `CRANE_VERSION` and `YJ_VERSION`. The shell script now references them as `${CRANE_VERSION}` and `${YJ_VERSION}` (already quoted in URL strings and comparisons), eliminating the template-engine injection vector.
2. github-env-injection (line 25): Added `safe_path=$(printf '%s' "${PATH}" | tr -d '\n\r')` before the `echo "PATH=..." >> "${GITHUB_ENV}"` line, so that any embedded newlines in the inherited PATH environment variable are stripped before being written to GITHUB_ENV, preventing injection of additional key=value pairs.

