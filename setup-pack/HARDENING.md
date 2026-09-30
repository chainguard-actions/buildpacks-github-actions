<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-pack/v6.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-pack/v6.1.1** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Resolve pack version': the shell variable `version` is derived from `inputs.pack-version` (via env var `$PACK_VERSION`) or from reading a file at a path supplied by `inputs.pack-version-file` (via env var `$PACK_VERSION_FILE`), both of which are untrusted caller-controlled inputs. The value is written directly to `$GITHUB_OUTPUT` with `echo "version=${version}" >> "${GITHUB_OUTPUT}"` without the required sanitization step (`printf '%s' "$version" | tr -d '\n\r'`) applied immediately before the write. An attacker can inject newlines into the version string to poison subsequent steps that read this output.

Locations:

- `action.yml:79`

### github-env-injection (severity: high)

Step 'Install pack CLI': `echo "PATH=${HOME}/bin:${PATH}" >> "${GITHUB_ENV}"` writes the inherited process env vars `$HOME` and `$PATH` to `$GITHUB_ENV` without sanitization. In a composite action, `$HOME` and `$PATH` are inherited from the calling workflow and are therefore workflow-controlled (untrusted). A calling workflow could set `$PATH` to a value containing newlines, allowing injection of arbitrary key=value pairs into the GitHub environment. The required sanitization (`printf '%s' ... | tr -d '\n\r'`) is absent.

Locations:

- `action.yml:90`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Fixed two github-env-injection findings in hardened/action/action.yml:
1. 'Resolve pack version' step (line 79): Sanitized the `version` variable with `printf '%s' "${version}" | tr -d '\n\r'` before writing to GITHUB_OUTPUT.
2. 'Install pack CLI' step (line 90): Sanitized both `$HOME` and `$PATH` with `printf '%s' ... | tr -d '\n\r'` before writing the PATH entry to GITHUB_ENV.

