<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-tools/v6.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-tools/v6.2.2** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a), an expression written directly into the script: in the composite step 'Install additional buildpack management tools', the inputs are placed straight into the run script. The offending lines are `CRANE_VERSION=${{ inputs.crane-version }}` and `YJ_VERSION=${{ inputs.yj-version }}`. A caller can pass shell metacharacters in these inputs and run any command they like. Fix: pass the inputs through env: (for example CRANE_VERSION: ${{ inputs.crane-version }}) and always use them double-quoted.

Locations:

- `action.yml:32`
- `action.yml:46`

### github-env-injection (severity: high)

The composite step writes `echo "PATH=${HOME}/bin:${PATH}" >> "${GITHUB_ENV}"`. HOME and PATH come from the calling workflow's environment and are not set in this script. They go into GITHUB_ENV without the required `printf '%s' ... | tr -d '\n\r'` step, so a value containing a newline could inject extra environment variables. Fix: sanitize the value before writing it, or append "${HOME}/bin" to GITHUB_PATH after sanitizing it.

Locations:

- `action.yml:26`

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

**Fixes applied:** script-injection, static-inline-injection, github-env-injection

**Notes:**

Moved inputs.crane-version and inputs.yj-version into the step's env: block as INPUT_CRANE_VERSION and INPUT_YJ_VERSION. The script now reads them as double-quoted shell variables. The PATH value written to GITHUB_ENV now goes through printf '%s' | tr -d '\n\r' first. Nothing else changed, so the action works as before.

### Iteration 2

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

action.yml line 49: replaced YJ_VERSION=${{ inputs.yj-version }} with YJ_VERSION="${INPUT_YJ_VERSION}", using the existing env: mapping.

