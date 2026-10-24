<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-pack/v6.2.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-pack/v6.2.1** was hardened automatically. 2 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

The 'Resolve pack version' step copies the untrusted input inputs.pack-version (passed in as the env var PACK_VERSION) straight into a shell variable (version="${PACK_VERSION}", line 41). It then writes that variable to the step output with `echo "version=${version}" >> "${GITHUB_OUTPUT}"` (line 76). Newlines and carriage returns are never stripped first, and routing the value through env: does not clean it. The pack-version-file branch also lacks the required printf '%s' | tr -d '\n\r' step: the plain-file branch only strips whitespace with tr -d '[:space:]', and the .tool-versions branch (awk) has no stripping at all. As a result, a value containing a newline can add extra outputs. Fix: run safe=$(printf '%s' "$version" | tr -d '\n\r') right before the write and write $safe instead.

Locations:

- `action.yml:76`
- `action.yml:41`

### github-env-injection (severity: high)

The 'Install pack CLI' step runs `echo "PATH=${HOME}/bin:${PATH}" >> "${GITHUB_ENV}"`. HOME and PATH are not set by this script; the composite action inherits them from the calling workflow. Under this check, inherited env vars count as workflow-controlled, so this write needs sanitizing. It goes to GITHUB_ENV without printf '%s' ... | tr -d '\n\r', so a newline in either variable could inject extra environment variables. Fix: sanitize first, e.g. safe=$(printf '%s' "${HOME}/bin:${PATH}" | tr -d '\n\r'), then write PATH=$safe. A simpler option is to append "${HOME}/bin", sanitized, to GITHUB_PATH.

Locations:

- `action.yml:88`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

Before each write to GITHUB_OUTPUT (the version) and GITHUB_ENV (PATH), the value is now passed through printf '%s' | tr -d '\n\r' to strip newlines and carriage returns. Neither write was a command substitution wrapped inside another function's argument, so a failing command still stops the step under errexit.

### Iteration 2

**Fixes applied:** github-env-injection

**Notes:**

In the 'Install pack CLI' step of action.yml, the HOME/PATH value now has CR and LF stripped (printf '%s' ... | tr -d '\n\r') before it is written to $GITHUB_ENV. I kept the PATH=... write to GITHUB_ENV so the action behaves as upstream does.

