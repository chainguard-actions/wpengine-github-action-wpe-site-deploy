<!-- markdownlint-disable -->

# Hardening Report: wpengine--github-action-wpe-site-deploy/v3.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **wpengine--github-action-wpe-site-deploy/v3.2.5** was hardened automatically. 4 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action.yml uses a mutable Docker image tag `docker://wpengine/site-deploy:1.0.3` instead of a SHA digest. This is vulnerable to supply-chain attacks if the image is updated or replaced. Additionally, multiple workflow files reference actions with mutable version tags instead of pinned 40-character commit SHAs: `actions/checkout@v3`, `fjogeleit/http-request-action@v1`, `voxmedia/github-action-slack-notify-build@v1`, `actions/setup-node@v2`, `changesets/action@v1`.

Locations:

- `action.yml:47`
- `.github/workflows/e2e-deploy.yml:16`
- `.github/workflows/e2e-deploy.yml:29`
- `.github/workflows/e2e-deploy.yml:36`
- `.github/workflows/lint-files.yml:13`
- `.github/workflows/release.yml:18`
- `.github/workflows/release.yml:23`
- `.github/workflows/release.yml:30`
- `.github/workflows/release.yml:43`
- `.github/workflows/release.yml:60`

### permissions (severity: medium)

None of the workflow files define a top-level `permissions:` key, and none of the individual jobs define job-level `permissions:` keys. This means workflows run with the default (broad) repository permissions, violating the principle of least privilege.

Locations:

- `.github/workflows/e2e-deploy.yml:1`
- `.github/workflows/lint-files.yml:1`
- `.github/workflows/release.yml:1`

### script-injection (severity: high)

Sub-rule (a): GitHub Actions expressions are interpolated directly into `run:` shell command strings, allowing script injection. (1) `.github/actions/publish/action.yml`: the run block is `"${{ github.action_path }}/publish.sh ${{ inputs.version }}"` — both `github.action_path` and `inputs.version` are injected directly into the shell command without quoting or env-var indirection. (2) `.github/actions/get-release-notes/action.yml`: the run block interpolates `${{ github.action_path }}`, `${{ inputs.version }}`, and `${{ inputs.changelog }}` directly into a shell command. (3) `.github/workflows/e2e-deploy.yml`: the run block `[ ${{ fromJson(steps.fetchResult.outputs.response).status }} = "success" ] || exit 1` interpolates a steps output expression directly into a shell test command.

Locations:

- `.github/actions/publish/action.yml:14`
- `.github/actions/get-release-notes/action.yml:18`
- `.github/workflows/e2e-deploy.yml:33`

### github-env-injection (severity: high)

In `.github/actions/get-release-notes/action.yml`, the variable `$notes` is populated by running `node ... ${{ inputs.version }} ${{ inputs.changelog }}` (attacker-controlled inputs) and then written to `$GITHUB_OUTPUT` via `echo "RELEASE_NOTES=$notes" >> $GITHUB_OUTPUT` without the required sanitization step (`printf '%s' ... | tr -d '\n\r'`). A malicious value containing newlines could inject arbitrary key-value pairs into the GitHub output environment.

Locations:

- `.github/actions/get-release-notes/action.yml:22`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses, permissions, script-injection, github-env-injection

**Notes:**

Fixed all four findings:

1. unpinned-uses: Pinned Docker image in action.yml to SHA digest. Pinned all mutable action tags in workflow files: actions/checkout@v3, fjogeleit/http-request-action@v1, voxmedia/github-action-slack-notify-build@v1, actions/setup-node@v2, changesets/action@v1 — all replaced with full 40-char commit SHAs with tag comments.

2. permissions: Added top-level `permissions: {}` to all three workflow files (e2e-deploy.yml, lint-files.yml, release.yml). Added minimal job-level permissions: `contents: read` for checkout-only jobs, `contents: write` + `pull-requests: write` for the versioning job (needs to create PRs via changesets), `contents: write` for the release job (needs to push tags/releases).

3. script-injection: (a) publish/action.yml: moved github.action_path and inputs.version into env vars ACTION_PATH/INPUT_VERSION, used as shell vars in run. (b) get-release-notes/action.yml: moved github.action_path, inputs.version, inputs.changelog into env vars, used as properly quoted shell vars. (c) e2e-deploy.yml validate step: moved fromJson expression into env var RESPONSE_STATUS, used as "$RESPONSE_STATUS" in shell test.

4. github-env-injection: In get-release-notes/action.yml, replaced the old percent-encoding approach with `safe=$(printf '%s' "$notes" | tr -d '\n\r')` before writing to $GITHUB_OUTPUT, preventing newline injection attacks.

