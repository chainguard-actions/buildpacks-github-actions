<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-tools/v6.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-tools/v6.1.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): Two `${{ inputs.* }}` expressions are directly interpolated into the `run:` shell script without going through an `env:` block. This allows an attacker who controls the `crane-version` or `yj-version` inputs to inject arbitrary shell commands.

Offending lines:
  Line 30: `CRANE_VERSION=${{ inputs.crane-version }}`
  Line 43: `YJ_VERSION=${{ inputs.yj-version }}`

Fix: Move the values into `env:` variables and reference them as quoted shell variables, e.g.:
```yaml
env:
  CRANE_VERSION: ${{ inputs.crane-version }}
  YJ_VERSION: ${{ inputs.yj-version }}
run: |
  crane_ver="$CRANE_VERSION"
  yj_ver="$YJ_VERSION"
  ...
```

Locations:

- `action.yml:30`
- `action.yml:43`

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

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

Moved ${{ inputs.crane-version }} and ${{ inputs.yj-version }} from inline shell assignments in the run: block to an env: block on the step. Removed the two inline assignments (CRANE_VERSION=${{ inputs.crane-version }} and YJ_VERSION=${{ inputs.yj-version }}) from the shell script. The script already referenced ${CRANE_VERSION} and ${YJ_VERSION} as shell variables throughout, so no other changes were needed.

