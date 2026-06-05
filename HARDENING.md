# Hardening Report: wpengine--github-action-wpe-site-deploy/v3.2.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **wpengine--github-action-wpe-site-deploy/v3.2.9** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action uses a Docker image reference with a mutable version tag instead of an immutable SHA digest. `image: docker://wpengine/site-deploy:1.0.7` should be pinned to a SHA256 digest (e.g., `docker://wpengine/site-deploy@sha256:<64-hex-char-digest>`) to prevent supply-chain attacks where the tag could be silently overwritten with malicious content.

Locations:

- `action.yml:46`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image `wpengine/site-deploy:1.0.7` to its immutable SHA256 digest `sha256:dffc860dbbaeac16be910d52c3a5c7890e71f0deceb13b586b9705e02463d114` in action.yml line 46. The original tag is preserved as a comment for readability: `docker://wpengine/site-deploy@sha256:dffc860dbbaeac16be910d52c3a5c7890e71f0deceb13b586b9705e02463d114 # 1.0.7`.

