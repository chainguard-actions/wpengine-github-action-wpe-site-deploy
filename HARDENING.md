<!-- markdownlint-disable -->

# Hardening Report: wpengine--github-action-wpe-site-deploy/v3.2.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **wpengine--github-action-wpe-site-deploy/v3.2.8** was hardened automatically. 4 finding(s) were identified and resolved across 2 iteration(s).

## Findings Fixed

### script-injection (severity: high)

Sub-rule (a): `${{ ... }}` expressions are interpolated directly inside `run:` shell command strings. In `.github/actions/publish/action.yml` line 15, `${{ github.action_path }}` and `${{ inputs.version }}` are embedded directly in the run command: `"${{ github.action_path }}/publish.sh ${{ inputs.version }}"`— an attacker controlling `inputs.version` can inject arbitrary shell commands. In `.github/actions/get-release-notes/action.yml` line 20, `${{ github.action_path }}`, `${{ inputs.version }}`, and `${{ inputs.changelog }}` are all interpolated directly: `notes=$(node ${{ github.action_path }}/getReleaseNotes ${{ inputs.version }} ${{ inputs.changelog }})`. In `.github/workflows/e2e-deploy.yml` line 33, `${{ fromJson(steps.fetchResult.outputs.response).status }}` is interpolated directly in a run block: `[ ${{ fromJson(steps.fetchResult.outputs.response).status }} = "success" ] || exit 1`.

Locations:

- `.github/actions/publish/action.yml:15`
- `.github/actions/get-release-notes/action.yml:20`
- `.github/workflows/e2e-deploy.yml:33`

### github-env-injection (severity: high)

In `.github/actions/get-release-notes/action.yml`, the variable `$notes` is populated via command substitution that directly interpolates `${{ inputs.version }}` and `${{ inputs.changelog }}` (attacker-controlled inputs). The result is then written to `$GITHUB_OUTPUT` on lines 21-23 (`echo "$notes" >> $GITHUB_OUTPUT`) without the required sanitization step (`printf '%s' "$notes" | tr -d '\n\r'`). A newline embedded in the inputs could inject arbitrary key=value pairs into the GitHub output context.

Locations:

- `.github/actions/get-release-notes/action.yml:21`

### unpinned-uses (severity: high)

Multiple workflow files and the root action.yml reference actions and Docker images by mutable tags or branch names instead of immutable 40-character commit SHAs or SHA digests, making them vulnerable to supply-chain attacks:

- `e2e-deploy.yml`: `actions/checkout@v4` (tag), `fjogeleit/http-request-action@v1` (tag)
- `lint-files.yml`: `actions/checkout@v4` (tag)
- `release.yml`: `actions/checkout@v4` (tag), `actions/setup-node@v4` (tag), `changesets/action@v1` (tag)
- `sonar.yml`: `sonarsource/sonarqube-scan-action@master` (branch), `sonarsource/sonarqube-quality-gate-action@master` (branch)
- `action.yml` (root): `image: docker://wpengine/site-deploy:1.0.6` (mutable Docker tag, not a SHA digest)

Locations:

- `.github/workflows/e2e-deploy.yml:20`
- `.github/workflows/e2e-deploy.yml:29`
- `.github/workflows/lint-files.yml:13`
- `.github/workflows/release.yml:14`
- `.github/workflows/release.yml:18`
- `.github/workflows/release.yml:24`
- `.github/workflows/sonar.yml:14`
- `.github/workflows/sonar.yml:20`
- `action.yml:47`

### missing-permissions (severity: medium)

None of the four workflow files define a top-level `permissions:` key, and none of their jobs define job-level `permissions:` blocks. This means all jobs run with the default (often broad) token permissions. Affected files: `e2e-deploy.yml`, `lint-files.yml`, `release.yml`, and `sonar.yml`.

Locations:

- `.github/workflows/e2e-deploy.yml:1`
- `.github/workflows/lint-files.yml:1`
- `.github/workflows/release.yml:1`
- `.github/workflows/sonar.yml:1`

## Iteration Notes

### Iteration 1

**Fixes applied:** script-injection, github-env-injection, unpinned-uses, missing-permissions

**Notes:**

Fixed all 4 findings across 7 files:

1. script-injection: Moved all ${{ }} expressions out of run: shell strings into env: blocks in publish/action.yml, get-release-notes/action.yml, and e2e-deploy.yml. Shell scripts now reference plain environment variables.

2. github-env-injection: In get-release-notes/action.yml, added sanitization with `printf '%s' "$notes" | tr -d '\r'` before writing to $GITHUB_OUTPUT to prevent newline injection attacks.

3. unpinned-uses: Pinned all mutable action references to full 40-character commit SHAs: actions/checkout@v4, fjogeleit/http-request-action@v1, actions/setup-node@v4, changesets/action@v1, sonarsource/sonarqube-scan-action@master, sonarsource/sonarqube-quality-gate-action@master. Also pinned the Docker image wpengine/site-deploy:1.0.6 to its sha256 digest while preserving the docker:// scheme.

4. missing-permissions: Added top-level permissions blocks to all 4 workflow files. e2e-deploy.yml, lint-files.yml, and sonar.yml get `contents: read`. release.yml gets `contents: write` and `pull-requests: write` (required for changesets to create PRs and for pushing tags/creating GitHub releases).

### Iteration 2

**Fixes applied:** script-injection, github-env-injection

**Notes:**

1. publish/action.yml (line 18): Fixed script-injection by converting the run command from a double-quoted YAML string to a block scalar, and properly double-quoting $INPUT_VERSION as a separate argument: `"$ACTION_PATH/publish.sh" "$INPUT_VERSION"`. This prevents word-splitting and shell metacharacter injection.
2. get-release-notes/action.yml (line 20): Fixed github-env-injection by changing `tr -d '\r'` to `tr -d '\n\r'` to strip both newlines and carriage returns, and replacing the heredoc output pattern with a safe single-line `echo "RELEASE_NOTES=$safe" >> "$GITHUB_OUTPUT"`. Since newlines are now stripped, the value cannot contain a newline followed by EOF to break out of a heredoc, and the simpler key=value form is safe.

