<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-pack/v4.1.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-pack/v4.1.0** was hardened automatically. 7 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Four `${{ inputs.* }}` expressions are directly interpolated inside the `run:` shell script. GitHub Actions substitutes these template expressions into the shell script text before execution, so a caller supplying a version string containing shell metacharacters (e.g., `; malicious_cmd #`) achieves command injection. Offending lines:
- `CRANE_VERSION=${{ inputs.crane-version }}` (line 36)
- `JQ_VERSION=${{ inputs.jq-version }}` (line 46)
- `PACK_VERSION=${{ inputs.pack-version }}` (line 55)
- `YJ_VERSION=${{ inputs.yj-version }}` (line 64)

Fix: Move each input into an `env:` block and reference the env var (double-quoted) inside the script instead of using `${{ }}` directly in the `run:` block.

Locations:

- `action.yml:36`
- `action.yml:46`
- `action.yml:55`
- `action.yml:64`

### script-injection (severity: high)

Sub-rule (b): The shell variables `${CRANE_VERSION}`, `${PACK_VERSION}` (populated from `${{ inputs.* }}`) are used unquoted inside URL strings passed to `curl`. Unquoted variable expansions allow the shell to parse metacharacters (`;`, `|`, `&`, whitespace, globs) out of the value, enabling command injection even after the value is stored in a variable. For example:
- `"https://.../${CRANE_VERSION}/go-containerregistry..."` — CRANE_VERSION is inside double-quotes here but was set via direct `${{ }}` interpolation (sub-rule a applies too).
- The `pack` URL uses `${PACK_VERSION}` twice inside a double-quoted string.
All four version variables should be double-quoted wherever expanded, and the `${{ }}` interpolation must be replaced with `env:` routing.

Locations:

- `action.yml:40`
- `action.yml:49`
- `action.yml:58`
- `action.yml:67`

### github-env-injection (severity: high)

The `run:` block writes the inherited process environment variable `$PATH` to `$GITHUB_ENV` without sanitization: `echo "PATH=${HOME}/bin:${PATH}" >> "${GITHUB_ENV}"`. The `$PATH` variable is not a literal computed in this script — it is inherited from the calling workflow's environment and is therefore workflow-controlled (untrusted). A calling workflow could set `PATH` to a value containing newline characters, which would inject additional key=value pairs into `$GITHUB_ENV`. The required sanitization step (`printf '%s' "$PATH" | tr -d '\n\r'`) must be applied before the write.

Locations:

- `action.yml:34`

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

Fixed all findings in action.yml:
1. Moved all four ${{ inputs.* }} expressions (crane-version, jq-version, pack-version, yj-version) from the run: block into a step-level env: block as INPUT_CRANE_VERSION, INPUT_JQ_VERSION, INPUT_PACK_VERSION, INPUT_YJ_VERSION. Inside the shell script they are now referenced as double-quoted env vars, eliminating script injection.
2. Sanitized the $PATH value before writing to $GITHUB_ENV using `printf '%s' ... | tr -d '\n\r'` to prevent newline-based env injection.

