<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-tools/v6.2.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-tools/v6.2.3** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): a GitHub Actions expression is placed directly in the run: script on two lines: `CRANE_VERSION=${{ inputs.crane-version }}` (line 32) and `YJ_VERSION=${{ inputs.yj-version }}` (line 45). The values are substituted into the script text before bash runs, so input such as `1; malicious_cmd` would run as a command. Fix: add an env: block for the step (e.g. CRANE_VERSION: ${{ inputs.crane-version }}, YJ_VERSION: ${{ inputs.yj-version }}) and refer only to "$CRANE_VERSION" and "$YJ_VERSION", double-quoted, in the script.

Locations:

- `action.yml:32`
- `action.yml:45`

### github-env-injection (severity: high)

Case (e): `echo "PATH=${HOME}/bin:${PATH}" >> "${GITHUB_ENV}"` copies the inherited process env vars HOME and PATH into $GITHUB_ENV without first removing newlines. In a composite action, the calling workflow controls both variables. Fix: strip newlines first, e.g. `safe=$(printf '%s' "${HOME}/bin:${PATH}" | tr -d '\n\r')`, then write `echo "PATH=$safe" >> "$GITHUB_ENV"`.

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

Moved inputs.crane-version and inputs.yj-version into the step's env: block as INPUT_CRANE_VERSION and INPUT_YJ_VERSION. The script now reads them double-quoted, so the rest of the script is unchanged. The PATH value written to $GITHUB_ENV now has newlines and carriage returns stripped with tr -d '\n\r' first. Nothing was run to test these changes.

