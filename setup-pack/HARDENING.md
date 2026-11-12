<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-pack/v6.2.3

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-pack/v6.2.3** was hardened automatically. 2 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### github-env-injection (severity: high)

Step 'Resolve pack version' writes a value taken from the inputs pack-version (via env PACK_VERSION) or pack-version-file (file contents) to $GITHUB_OUTPUT without sanitizing it first: `echo "version=${version}" >> "${GITHUB_OUTPUT}"`. When the value comes straight from PACK_VERSION it is copied as-is, so an embedded newline can inject more outputs. Fix: run `version=$(printf '%s' "$version" | tr -d '\n\r')` immediately before the write.

Locations:

- `action.yml:76`

### github-env-injection (severity: high)

Step 'Install pack CLI' writes the inherited process env vars HOME and PATH to $GITHUB_ENV without sanitizing them: `echo "PATH=${HOME}/bin:${PATH}" >> "${GITHUB_ENV}"`. In a composite action these values come from the calling workflow and are workflow-controlled. Fix: pass the value through `printf '%s' ... | tr -d '\n\r'` before writing it.

Locations:

- `action.yml:88`

## Iteration Notes

### Iteration 1

**Fixes applied:** github-env-injection

**Notes:**

action.yml: the resolved version now has CR/LF stripped before it is written to GITHUB_OUTPUT. The PATH value (HOME/bin:PATH) also has CR/LF stripped before it is written to GITHUB_ENV. Nothing else changed.

