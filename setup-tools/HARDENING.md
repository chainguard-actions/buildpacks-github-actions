<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-tools/v6.2.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-tools/v6.2.4** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): workflow expressions are placed directly into the composite step's run: script. Line 32: `CRANE_VERSION=${{ inputs.crane-version }}`. Line 45: `YJ_VERSION=${{ inputs.yj-version }}`. Each value is pasted into the bash script before it runs, and it is not quoted, so a caller-supplied input could run commands. Fix: pass the inputs through env: (for example CRANE_VERSION: ${{ inputs.crane-version }}) and refer to them as "$CRANE_VERSION" in double quotes.

Locations:

- `action.yml:32`
- `action.yml:45`

### github-env-injection (severity: high)

Case (e): `echo "PATH=${HOME}/bin:${PATH}" >> "${GITHUB_ENV}"` writes the inherited process env vars HOME and PATH to $GITHUB_ENV. The calling workflow controls these values, and the step does not first remove newlines with `printf '%s' ... | tr -d '\n\r'`. A newline in either value could inject extra environment variables. Fix: clean the value before the write, for example `safe=$(printf '%s' "${HOME}/bin:${PATH}" | tr -d '\n\r')`. Better still, add the directory with `printf '%s\n' "$safe_dir" >> "$GITHUB_PATH"`, using a newline-stripped value, instead of rewriting PATH.

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

All changes are in action.yml. Not tested: I made the edits but have not run the action.

- **Script injection:** inputs.crane-version and inputs.yj-version now go through the step's env: block as INPUT_CRANE_VERSION and INPUT_YJ_VERSION. The script reads them in double quotes, so they are no longer pasted into the script text.
- **PATH write:** the step no longer rewrites the whole PATH into $GITHUB_ENV. It strips newlines from "$HOME/bin" and adds just that directory to $GITHUB_PATH. Later steps still find $HOME/bin first on PATH, as before, but the current PATH value is no longer copied into $GITHUB_ENV.

