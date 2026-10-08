<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-pack/v6.2.0

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-pack/v6.2.0** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Resolve pack version' step writes the `version` variable to $GITHUB_OUTPUT without sanitization. The value of `version` is derived from the untrusted inputs `inputs.pack-version` (via env var PACK_VERSION) or from a file path supplied by `inputs.pack-version-file` (via env var PACK_VERSION_FILE). An attacker-controlled value containing newlines could inject arbitrary key=value pairs into GITHUB_OUTPUT, potentially poisoning subsequent steps. The required sanitization step (`printf '%s' "$version" | tr -d '\n\r'`) is absent before the write: `echo "version=${version}" >> "${GITHUB_OUTPUT}"`.

Locations:

- `action.yml:72`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the 'Resolve pack version' step in action.yml (line 72) by sanitizing the `version` variable before writing it to $GITHUB_OUTPUT. Added `safe_version="$(printf '%s' "${version}" | tr -d '\n\r')"` and changed the echo to use `safe_version` instead of `version`. This prevents attacker-controlled values from `inputs.pack-version` or `inputs.pack-version-file` from injecting arbitrary key=value pairs into GITHUB_OUTPUT via embedded newlines.

