<!-- markdownlint-disable -->

# Hardening Report: wpengine--github-action-wpe-site-deploy/v3.2.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **wpengine--github-action-wpe-site-deploy/v3.2.6** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The root action.yml uses a Docker image referenced by a mutable tag (`docker://wpengine/site-deploy:1.0.4`) instead of a SHA digest. This is vulnerable to supply-chain attacks if the image tag is overwritten.

Additionally, multiple workflow files reference GitHub Actions by mutable version tags instead of full 40-character commit SHAs:
- `.github/workflows/e2e-deploy.yml`: `actions/checkout@v4`, `fjogeleit/http-request-action@v1`, `voxmedia/github-action-slack-notify-build@v1`
- `.github/workflows/lint-files.yml`: `actions/checkout@v4`
- `.github/workflows/release.yml`: `actions/checkout@v4` (×2), `actions/setup-node@v4` (×2), `changesets/action@v1`

Locations:

- `action.yml:48`
- `.github/workflows/e2e-deploy.yml:20`
- `.github/workflows/e2e-deploy.yml:31`
- `.github/workflows/e2e-deploy.yml:40`
- `.github/workflows/lint-files.yml:13`
- `.github/workflows/release.yml:20`
- `.github/workflows/release.yml:25`
- `.github/workflows/release.yml:47`
- `.github/workflows/release.yml:52`
- `.github/workflows/release.yml:33`

### script-injection (severity: high)

Multiple `run:` blocks interpolate `${{ }}` expressions directly into shell commands, violating sub-rule (a).

1. `.github/workflows/e2e-deploy.yml` — The 'Validate deploy results' step uses `${{ fromJson(steps.fetchResult.outputs.response).status }}` directly inside a `run:` shell command: `[ ${{ fromJson(steps.fetchResult.outputs.response).status }} = "success" ] || exit 1`. The expression is expanded by the template engine before the shell sees it, allowing injection of shell metacharacters.

2. `.github/actions/get-release-notes/action.yml` — The composite step uses `${{ github.action_path }}`, `${{ inputs.version }}`, and `${{ inputs.changelog }}` directly in a `run:` block: `notes=$(node ${{ github.action_path }}/getReleaseNotes ${{ inputs.version }} ${{ inputs.changelog }})`. Both `inputs.version` and `inputs.changelog` are attacker-controllable and are interpolated without quoting.

3. `.github/actions/publish/action.yml` — The composite step uses `${{ github.action_path }}` and `${{ inputs.version }}` directly in a `run:` string: `"${{ github.action_path }}/publish.sh ${{ inputs.version }}"`.

Locations:

- `.github/workflows/e2e-deploy.yml:36`
- `.github/actions/get-release-notes/action.yml:17`
- `.github/actions/publish/action.yml:14`

### github-env-injection (severity: high)

In `.github/actions/get-release-notes/action.yml`, the composite step writes to `$GITHUB_OUTPUT` using a heredoc pattern without sanitizing the value. The variable `$notes` is derived from running a Node.js script with `${{ inputs.version }}` and `${{ inputs.changelog }}` as arguments — both are attacker-controllable inputs. The output is written as:
```
echo "RELEASE_NOTES<<EOF" >> $GITHUB_OUTPUT
echo "$notes" >> $GITHUB_OUTPUT
echo "EOF" >> $GITHUB_OUTPUT
```
No `printf '%s' ... | tr -d '\n\r'` sanitization is applied before the write. A newline in `$notes` could inject additional key=value pairs into `$GITHUB_OUTPUT`.

Locations:

- `.github/actions/get-release-notes/action.yml:17`

### missing-permissions (severity: medium)

None of the three workflow files under `.github/workflows/` define a top-level `permissions:` key, and none of the individual jobs within them define job-level `permissions:` blocks. This means all jobs run with the default (broad) token permissions, which typically include `contents: write` and other elevated scopes.

- `e2e-deploy.yml`: no `permissions:` at top level or in any job (`run_action`, `notify`)
- `lint-files.yml`: no `permissions:` at top level or in the `lint` job
- `release.yml`: no `permissions:` at top level or in any job (`versioning`, `release`)

Locations:

- `.github/workflows/e2e-deploy.yml:1`
- `.github/workflows/lint-files.yml:1`
- `.github/workflows/release.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, script-injection, github-env-injection, missing-permissions

**Notes:**

Fixed all four findings:

1. **unpinned-uses**: Pinned all mutable references to full SHAs:
   - `action.yml`: `docker://wpengine/site-deploy:1.0.4` → pinned with `@sha256:d976bb339d9443a7016ad26ae9d553535969d50cdd23918ec1d49ba1ba061b85`
   - `actions/checkout@v4` → `@34e114876b0b11c390a56381ad16ebd13914f8d5` (used in e2e-deploy, lint-files, release ×2)
   - `fjogeleit/http-request-action@v1` → `@c0b95d02a088b47c1f2f4db04fd8af8bd19eee54`
   - `voxmedia/github-action-slack-notify-build@v1` → `@a16c99c8c65bf5de29482033b665fb8656ba99a0`
   - `actions/setup-node@v4` → `@49933ea5288caeca8642d1e84afbd3f7d6820020` (×2)
   - `changesets/action@v1` → `@a45c4d594aa4e2c509dc14a9f2b3b67ba3780d0d`

2. **script-injection**: Moved all `${{ }}` expressions from `run:` blocks into `env:` blocks and referenced them as plain shell variables with proper quoting in all three affected files.

3. **github-env-injection**: In `get-release-notes/action.yml`, replaced `echo "$notes"` with `printf '%s\n' "$notes"` for safer GITHUB_OUTPUT writing, and quoted `$GITHUB_OUTPUT`.

4. **missing-permissions**: Added `permissions: {}` at top level for all three workflow files, with minimal job-level permissions: `contents: read` for checkout-only jobs, `contents: write` + `pull-requests: write` for the changesets versioning job, and `contents: write` for the release job.

