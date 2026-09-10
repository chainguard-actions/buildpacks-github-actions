<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-pack/v4.6.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-pack/v4.6.0** was hardened automatically. 6 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Four `${{ inputs.* }}` expressions are interpolated directly inside the `run:` shell script. GitHub Actions performs YAML template substitution before the shell ever sees the string, so an attacker-controlled input value can break out of the variable assignment and inject arbitrary shell commands. Offending lines:
  Line 35: `CRANE_VERSION=${{ inputs.crane-version }}`
  Line 43: `JQ_VERSION=${{ inputs.jq-version }}`
  Line 51: `PACK_VERSION=${{ inputs.pack-version }}`
  Line 59: `YJ_VERSION=${{ inputs.yj-version }}`

Sub-rule (b): The resulting shell variables `${CRANE_VERSION}`, `${JQ_VERSION}`, `${PACK_VERSION}`, and `${YJ_VERSION}` (all holding `inputs.*` values) are expanded unquoted inside curl URL strings (e.g. `"https://.../${CRANE_VERSION}/..."` — the outer quotes cover the whole string but the variable itself is not separately double-quoted, and the value was injected via direct expression interpolation). Fix: move each input into an `env:` block and reference it as a double-quoted shell variable, e.g. `env: { CRANE_VERSION: "${{ inputs.crane-version }}" }` and `"${CRANE_VERSION}"` in the script.

Locations:

- `action.yml:35`
- `action.yml:43`
- `action.yml:51`
- `action.yml:59`

### github-env-injection (severity: high)

Line 33 writes the inherited process environment variable `${PATH}` directly to `$GITHUB_ENV` without sanitization: `echo "PATH=${HOME}/bin:${PATH}" >> "${GITHUB_ENV}"`. Per the check rules, any process env var read inside a composite action's `run:` block that was not set immediately above by the same script is workflow-controlled and must be treated as untrusted. Writing it to GITHUB_ENV without `printf '%s' "$PATH" | tr -d '\n\r'` allows a newline-injection attack that can set arbitrary environment variables for subsequent steps. Fix: sanitize the value before writing, e.g.:
  `safe_path=$(printf '%s' "${HOME}/bin:${PATH}" | tr -d '\n\r')`
  `echo "PATH=${safe_path}" >> "${GITHUB_ENV}"`

Locations:

- `action.yml:33`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.crane-version }}" appears directly in run: block of step "Setup pack CLI"; move to env: map

Locations:

- `action.yml:36`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.jq-version }}" appears directly in run: block of step "Setup pack CLI"; move to env: map

Locations:

- `action.yml:45`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.pack-version }}" appears directly in run: block of step "Setup pack CLI"; move to env: map

Locations:

- `action.yml:55`

### static-inline-injection (severity: high)

shell injection: expression "${{ inputs.yj-version }}" appears directly in run: block of step "Setup pack CLI"; move to env: map

Locations:

- `action.yml:64`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

Fixed hardened/action/action.yml:
1. Moved all four ${{ inputs.* }} expressions (crane-version, jq-version, pack-version, yj-version) from inline run: shell script into a step-level env: block, eliminating script injection risk. The shell script now references them as plain env vars.
2. Sanitized the PATH value before writing to $GITHUB_ENV by using `safe_path=$(printf '%s' "${HOME}/bin:${PATH}" | tr -d '\n\r')` and then `echo "PATH=${safe_path}" >> "${GITHUB_ENV}"`, preventing newline-injection attacks.

