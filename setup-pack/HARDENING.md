<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-pack/v6.2.2

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-pack/v6.2.2** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Resolve pack version': the pack-version input (env PACK_VERSION from ${{ inputs.pack-version }}) is copied to $version and written with `echo "version=${version}" >> "${GITHUB_OUTPUT}"`. Nothing strips newlines first (`printf '%s' ... | tr -d '\n\r'`). A value with a newline in it could add extra step outputs. The version-file path only removes whitespace on the plain-file branch, and the explicit-input branch doesn't sanitize at all.

Locations:

- `action.yml:76`

### github-env-injection (severity: high)

Step 'Install pack CLI': `echo "PATH=${HOME}/bin:${PATH}" >> "${GITHUB_ENV}"` writes the inherited HOME and PATH variables to GITHUB_ENV without `printf '%s' ... | tr -d '\n\r'`. Composite actions get their environment from the calling workflow, so this check treats these values as workflow-controlled. A newline in either one could inject more environment variables.

Locations:

- `action.yml:88`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

In action.yml: the 'Resolve pack version' step now runs the resolved version through printf '%s' | tr -d '\n\r' before writing version= to GITHUB_OUTPUT. The 'Install pack CLI' step does the same to ${HOME}/bin:${PATH} before writing PATH= to GITHUB_ENV. Nothing else changed.

