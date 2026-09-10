<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-pack/v6.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-pack/v6.1.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

In the 'Resolve pack version' step, the shell variable `version` is derived from `PACK_VERSION` (mapped from `inputs.pack-version`) or from reading a file at the path given by `PACK_VERSION_FILE` (mapped from `inputs.pack-version-file`). The resolved value is written directly to `$GITHUB_OUTPUT` via `echo "version=${version}" >> "${GITHUB_OUTPUT}"` without the required newline-stripping sanitization (`printf '%s' "$version" | tr -d '\n\r'`). An attacker who controls the `pack-version` input (or the contents of the file at `pack-version-file`) could inject newline characters to smuggle additional key=value pairs into `GITHUB_OUTPUT`, potentially overwriting subsequent step outputs or poisoning the environment of downstream steps.

Locations:

- `action.yml:76`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in action.yml at the 'Resolve pack version' step. The `version` variable (derived from user-controlled inputs `pack-version` or `pack-version-file`) was written directly to $GITHUB_OUTPUT without newline sanitization. Fixed by introducing `safe_version="$(printf '%s' "${version}" | tr -d '\n\r')"` and writing `safe_version` to $GITHUB_OUTPUT instead, preventing newline injection attacks that could smuggle additional key=value pairs into GITHUB_OUTPUT.

