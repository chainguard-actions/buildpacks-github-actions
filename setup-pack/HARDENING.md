<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-pack/v6.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-pack/v6.1.1** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Resolve pack version' step writes the `version` variable to `$GITHUB_OUTPUT` without sanitization. The `version` value is derived from `${PACK_VERSION}` (which holds `inputs.pack-version`) or from reading a file at the path `${PACK_VERSION_FILE}` (which holds `inputs.pack-version-file`). Both are attacker-controlled inputs. The write `echo "version=${version}" >> "${GITHUB_ENV}"` is not preceded by the required `printf '%s' ... | tr -d '\n\r'` sanitization step, allowing an attacker to inject newlines and poison subsequent `$GITHUB_OUTPUT` entries (e.g., by supplying a version string containing `\nsome_key=injected_value`).

Locations:

- `action.yml:88`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed the github-env-injection finding in hardened/action/action.yml at line 88. The 'Resolve pack version' step now sanitizes the `version` variable before writing it to $GITHUB_OUTPUT. Added `safe_version="$(printf '%s' "${version}" | tr -d '\n\r')"` and changed the echo to use `safe_version` instead of `version`. This prevents an attacker from injecting newlines via the `pack-version` input or a malicious `pack-version-file` to poison subsequent $GITHUB_OUTPUT entries.

