<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-pack/v4.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-pack/v4.1.0** was hardened automatically. 5 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a) violation: Four `${{ inputs.* }}` expressions are directly interpolated inside the `run:` shell script in action.yml. The Actions runner substitutes these expressions before the shell executes them, so a caller supplying a malicious value (e.g. containing `;`, `$(...)`, or backticks) can inject arbitrary shell commands. Offending lines:
- `CRANE_VERSION=${{ inputs.crane-version }}`
- `JQ_VERSION=${{ inputs.jq-version }}`
- `PACK_VERSION=${{ inputs.pack-version }}`
- `YJ_VERSION=${{ inputs.yj-version }}`

Fix: move each input into an `env:` block and reference it as a quoted shell variable, e.g. `"$CRANE_VERSION"`.

Locations:

- `action.yml:34`
- `action.yml:43`
- `action.yml:52`
- `action.yml:61`

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

Moved all four ${{ inputs.* }} expressions (crane-version, jq-version, pack-version, yj-version) from the run: shell script into an env: block on the 'Setup pack CLI' step in action.yml. The shell script now references them as plain environment variables (${CRANE_VERSION}, ${JQ_VERSION}, ${PACK_VERSION}, ${YJ_VERSION}), eliminating the risk of shell injection via malicious input values.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

Fixed the GITHUB_ENV injection vulnerability in action.yml at line 37. The PATH value is now sanitized before being written to GITHUB_ENV by using `safe_path=$(printf '%s' "${HOME}/bin:${PATH}" | tr -d '\n\r')` to strip any embedded newlines or carriage returns that could allow injection of arbitrary key=value pairs into the environment.

