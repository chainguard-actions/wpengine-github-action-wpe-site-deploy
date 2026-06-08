# Hardening Report: wpengine--github-action-wpe-site-deploy/v3.2.8

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **wpengine--github-action-wpe-site-deploy/v3.2.8** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action uses a Docker image referenced by a mutable version tag rather than an immutable SHA digest. `image: docker://wpengine/site-deploy:1.0.6` uses the tag `1.0.6`, which can be changed at any time by the image publisher, exposing the action to supply-chain attacks. It should be pinned to a full SHA256 digest, e.g. `image: docker://wpengine/site-deploy@sha256:<64-hex-char-digest> # 1.0.6`.

Locations:

- `action.yml:46`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yml from the mutable tag `wpengine/site-deploy:1.0.6` to the immutable digest `wpengine/site-deploy@sha256:404cfc445e1d614e1b98e785081a1c74c412a24fd9aeac0d942b7c83e876fb55 # 1.0.6`. The tag is preserved as a comment for readability.

