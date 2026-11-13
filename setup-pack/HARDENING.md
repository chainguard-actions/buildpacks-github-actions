<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-pack/v6.2.4

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-pack/v6.2.4** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Resolve pack version' runs `echo "version=${version}" >> "${GITHUB_OUTPUT}"`. When the pack-version input is set, `version` comes straight from PACK_VERSION (`${{ inputs.pack-version }}`) with no `printf '%s' ... | tr -d '\n\r'` step. A value containing a newline could add extra outputs to the step. Only the plain-file branch strips whitespace. The pack-version and .tool-versions branches do not.

Locations:

- `action.yml:82`

### github-env-injection (severity: high)

Step 'Install pack CLI' runs `echo "PATH=${HOME}/bin:${PATH}" >> "${GITHUB_ENV}"`. PATH and HOME come from the calling workflow's environment and are not set inside this script. They are written to $GITHUB_ENV without the `printf '%s' ... | tr -d '\n\r'` step, so a newline in either value could add extra environment variables. Sanitize the value before writing it, or write `${HOME}/bin` to $GITHUB_PATH after sanitizing it.

Locations:

- `action.yml:93`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

I fixed both findings in action.yml. I haven't run the action or any tests after the edits.

1. **Resolve pack version:** the version is now passed through `printf '%s' | tr -d '\n\r'` before it is written to $GITHUB_OUTPUT. This covers all three ways the version gets set: the pack-version input, the version file and the default.

2. **Install pack CLI:** the `${HOME}/bin:${PATH}` value is now cleaned the same way before it is written to $GITHUB_ENV.

When the values contain no newlines, the action behaves exactly as before.

