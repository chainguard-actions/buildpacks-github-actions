<!-- markdownlint-disable -->

# Hardening Report: buildpacks--github-actions--setup-tools/v6.1.1

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **buildpacks--github-actions--setup-tools/v6.1.1** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Rule (a): Two `${{ inputs.* }}` expressions are interpolated directly into the `run:` shell script in action.yml. GitHub Actions substitutes these values into the script text before the shell executes it, so an attacker-controlled input containing shell metacharacters (`;`, `$(...)`, `|`, backticks, etc.) can inject arbitrary commands.

Offending lines:
- `CRANE_VERSION=${{ inputs.crane-version }}` — the crane version input is spliced raw into the shell script. The value is then used unquoted in a URL: `"https://...v${CRANE_VERSION}/..."` (inside double-quotes, but the injection already occurred at assignment).
- `YJ_VERSION=${{ inputs.yj-version }}` — same pattern for the yj version input.

Fix: pass inputs via `env:` and reference them as quoted shell variables, e.g.:
```yaml
env:
  CRANE_VERSION: ${{ inputs.crane-version }}
run: |
  case "$CRANE_VERSION" in
    [0-9]*) ;; *) echo 'invalid version' >&2; exit 1 ;;
  esac
  curl ... "https://.../v${CRANE_VERSION}/..."
```

Locations:

- `action.yml:32`
- `action.yml:45`

### github-env-injection (severity: high)

The `run:` block writes `${PATH}` — an inherited process environment variable set by the calling workflow — directly to `$GITHUB_ENV` without sanitization:

```bash
echo "PATH=${HOME}/bin:${PATH}" >> "${GITHUB_ENV}"
```

A calling workflow can set `PATH` to a value containing embedded newlines (`\n`), which would inject additional `KEY=VALUE` entries into `GITHUB_ENV`, allowing environment variable poisoning for subsequent steps. The required sanitization step (`printf '%s' "$PATH" | tr -d '\n\r'`) is absent.

Fix:
```bash
safe_path=$(printf '%s' "${HOME}/bin:${PATH}" | tr -d '\n\r')
echo "PATH=${safe_path}" >> "${GITHUB_ENV}"
```

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

Fixed all findings in action.yml:
1. Moved ${{ inputs.crane-version }} and ${{ inputs.yj-version }} from the run: shell script into a new env: block on the step. The shell script now references them as $CRANE_VERSION and $YJ_VERSION, preventing shell metacharacter injection.
2. Sanitized the PATH value before writing to $GITHUB_ENV using `safe_path=$(printf '%s' "${HOME}/bin:${PATH}" | tr -d '\n\r')` to prevent newline-based environment variable injection.

