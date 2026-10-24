<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-tools/v6.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-tools/v6.2.1** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a), direct expression interpolation in a composite run block. Two lines put input expressions straight into the script: `CRANE_VERSION=${{ inputs.crane-version }}` (line 32) and `YJ_VERSION=${{ inputs.yj-version }}` (line 45). The values are substituted into the script before bash parses it, and they are not quoted, so a calling workflow can inject shell commands. Fix: pass them through env: (CRANE_VERSION: ${{ inputs.crane-version }}, YJ_VERSION: ${{ inputs.yj-version }}) and always expand them in double quotes.

Locations:

- `action.yml:32`
- `action.yml:45`

### github-env-injection (severity: high)

Case (e). The line `echo "PATH=${HOME}/bin:${PATH}" >> "${GITHUB_ENV}"` writes HOME and PATH to $GITHUB_ENV. Both are inherited from the calling workflow, not set in this script, and nothing strips newlines first. A value containing newlines could inject extra environment variables. Fix: build the value, run it through `printf '%s' ... | tr -d '\n\r'`, then write it. Writing `${HOME}/bin` to $GITHUB_PATH is also cleaner.

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

**Fixes applied:** script-injection, github-env-injection, static-inline-injection

**Notes:**

Moved inputs.crane-version and inputs.yj-version into the step's env: block as INPUT_CRANE_VERSION and INPUT_YJ_VERSION, and they are now expanded in double quotes. The action used to write PATH=${HOME}/bin:${PATH} to GITHUB_ENV. It now strips newlines from ${HOME}/bin and writes it to GITHUB_PATH, which puts the directory at the front of PATH for later steps, so later steps see the same PATH as before.

### Iteration 2

**Fixes applied:** script-injection, static-inline-injection

**Notes:**

In action.yml, replaced `YJ_VERSION=${{ inputs.yj-version }}` with `YJ_VERSION="${INPUT_YJ_VERSION}"`. The step's env: block already mapped that variable to the input.

