<!-- markdownlint-disable -->

# Hardening Report: wpengine--github-action-wpe-site-deploy/v3.2.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **wpengine--github-action-wpe-site-deploy/v3.2.7** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

Multiple workflow files reference actions using mutable version tags instead of pinned 40-character SHA commits, making them vulnerable to supply-chain attacks. Unpinned references found:
- e2e-deploy.yml: `actions/checkout@v4` (line 21), `fjogeleit/http-request-action@v1` (line 35), `voxmedia/github-action-slack-notify-build@v1` (line 50)
- lint-files.yml: `actions/checkout@v4` (line 14)
- release.yml: `actions/checkout@v4` (lines 22, 40), `actions/setup-node@v4` (lines 26, 55), `changesets/action@v1` (line 30)

Additionally, action.yml uses a mutable Docker image tag `docker://wpengine/site-deploy:1.0.5` instead of a SHA digest (e.g. `docker://wpengine/site-deploy@sha256:<digest>`).

Locations:

- `.github/workflows/e2e-deploy.yml:21`
- `.github/workflows/e2e-deploy.yml:35`
- `.github/workflows/e2e-deploy.yml:50`
- `.github/workflows/lint-files.yml:14`
- `.github/workflows/release.yml:22`
- `.github/workflows/release.yml:26`
- `.github/workflows/release.yml:30`
- `.github/workflows/release.yml:40`
- `.github/workflows/release.yml:55`
- `action.yml:50`

### permissions (severity: medium)

None of the three workflow files define a top-level `permissions:` block, and no individual job within them defines a `permissions:` block either. Without explicit permissions, workflows run with the repository's default token permissions, which may be overly broad (write access to contents, pull-requests, etc.).

Locations:

- `.github/workflows/e2e-deploy.yml:1`
- `.github/workflows/lint-files.yml:1`
- `.github/workflows/release.yml:1`

### script-injection (severity: high)

GitHub Actions expressions are interpolated directly inside `run:` shell command strings, allowing an attacker to inject arbitrary shell commands.

(a) `.github/workflows/e2e-deploy.yml` line 40: `${{ fromJson(steps.fetchResult.outputs.response).status }}` is interpolated directly into a shell `[` test command. The value comes from an external HTTP response and is not sanitized before shell evaluation.

(b) `.github/actions/get-release-notes/action.yml` line 19: `${{ inputs.version }}` and `${{ inputs.changelog }}` are interpolated directly into a `run:` shell command (`node ... ${{ inputs.version }} ${{ inputs.changelog }}`). A caller can supply a value containing shell metacharacters.

(c) `.github/actions/publish/action.yml` line 14: `${{ inputs.version }}` is interpolated directly as the entire `run:` value (`"${{ github.action_path }}/publish.sh ${{ inputs.version }}"`), allowing shell injection via the version input.

Locations:

- `.github/workflows/e2e-deploy.yml:40`
- `.github/actions/get-release-notes/action.yml:19`
- `.github/actions/publish/action.yml:14`

### github-env-injection (severity: high)

In `.github/actions/get-release-notes/action.yml`, the `run:` block writes the variable `$notes` to `$GITHUB_OUTPUT` without sanitization. `$notes` is the output of a Node.js command whose arguments are `${{ inputs.version }}` and `${{ inputs.changelog }}` — both untrusted caller-controlled inputs. A malicious input could embed newlines to inject arbitrary key=value pairs into `$GITHUB_OUTPUT`, potentially overwriting other step outputs. The required sanitization step (`printf '%s' "$notes" | tr -d '\n\r'`) is absent before the `echo "$notes" >> $GITHUB_OUTPUT` write.

Locations:

- `.github/actions/get-release-notes/action.yml:20`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings:

1. **unpinned-uses**: Pinned all action references to full SHA commits:
   - actions/checkout@v4 → @34e114876b0b11c390a56381ad16ebd13914f8d5
   - fjogeleit/http-request-action@v1 → @c0b95d02a088b47c1f2f4db04fd8af8bd19eee54
   - voxmedia/github-action-slack-notify-build@v1 → @a16c99c8c65bf5de29482033b665fb8656ba99a0
   - actions/setup-node@v4 → @49933ea5288caeca8642d1e84afbd3f7d6820020
   - changesets/action@v1 → @a45c4d594aa4e2c509dc14a9f2b3b67ba3780d0d
   - Docker image wpengine/site-deploy:1.0.5 → pinned with @sha256:9ca1f817b908acc0e8508ba26e3f8fdc927cb162d4b3c3f8a61882b150e0dd68 (docker:// scheme preserved)

2. **permissions**: Added `permissions: {}` at top-level to all three workflow files. Added job-level permissions with minimum required access (contents: read for checkout-only jobs, contents: write + pull-requests: write for the versioning job).

3. **script-injection**: Moved all ${{ }} expressions out of run: shell strings into env: blocks in e2e-deploy.yml, get-release-notes/action.yml, and publish/action.yml. Referenced as plain environment variables in shell scripts.

4. **github-env-injection**: In get-release-notes/action.yml, sanitized the notes output with `printf '%s' "$notes" | tr -d '\n\r'` before writing to GITHUB_OUTPUT, and switched from heredoc format to simple key=value format to prevent newline injection attacks.

### Iteration 2

**Fixes applied:** script-injection

**Notes:**

Fixed two script-injection findings by replacing `${{ github.action_path }}` with the built-in `$GITHUB_ACTION_PATH` environment variable:
1. `.github/actions/get-release-notes/action.yml` line 22: Changed `node "${{ github.action_path }}/getReleaseNotes"` to `node "$GITHUB_ACTION_PATH/getReleaseNotes"`
2. `.github/actions/publish/action.yml` line 17: Changed `"${{ github.action_path }}/publish.sh"` to `"$GITHUB_ACTION_PATH/publish.sh"`

The `$GITHUB_ACTION_PATH` environment variable is set automatically by GitHub Actions and is safe to use directly in shell scripts without YAML template interpolation risk.

