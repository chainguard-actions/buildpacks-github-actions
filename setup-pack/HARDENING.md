<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-pack/v4.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-pack/v4.1.0** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a) violation: Four `${{ inputs.* }}` expressions are directly interpolated inside the `run:` shell script in action.yml. This means the YAML template substitutes the raw input value into the shell command string before the shell ever sees it, allowing an attacker-controlled input to inject arbitrary shell commands via metacharacters. Offending lines:
- Line 35: `CRANE_VERSION=${{ inputs.crane-version }}`
- Line 43: `JQ_VERSION=${{ inputs.jq-version }}`
- Line 51: `PACK_VERSION=${{ inputs.pack-version }}`
- Line 59: `YJ_VERSION=${{ inputs.yj-version }}`

Fix: Move each input into the step's `env:` block (e.g. `CRANE_VERSION: ${{ inputs.crane-version }}`) and reference it as a quoted shell variable (`"$CRANE_VERSION"`) inside the `run:` script.

Locations:

- `action.yml:35`
- `action.yml:43`
- `action.yml:51`
- `action.yml:59`

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

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Moved all four ${{ inputs.* }} expressions (crane-version, jq-version, pack-version, yj-version) from the run: shell script into an env: block on the 'Setup pack CLI' step. The shell script now references them as environment variables (${CRANE_VERSION}, ${JQ_VERSION}, ${PACK_VERSION}, ${YJ_VERSION}) instead of inline template expressions, eliminating the shell injection risk.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in action.yml at line 36. The PATH environment variable is now sanitized before being written to $GITHUB_ENV. Added `safe_path=$(printf '%s' "${PATH}" | tr -d '\n\r')` before the echo command, and replaced `${PATH}` with `${safe_path}` in the GITHUB_ENV write. This prevents newline injection attacks where an attacker-controlled calling workflow could inject arbitrary key=value pairs into GITHUB_ENV via a crafted PATH value.

