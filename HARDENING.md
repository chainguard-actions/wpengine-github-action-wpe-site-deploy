# Hardening Report: wpengine--github-action-wpe-site-deploy/v3.2.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **wpengine--github-action-wpe-site-deploy/v3.2.7** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action uses a Docker image referenced by a mutable tag (`docker://wpengine/site-deploy:1.0.5`) instead of an immutable SHA digest. If the tag is overwritten (intentionally or via a supply-chain attack), the action will silently execute different code. The image reference should be pinned to a full SHA256 digest, e.g. `docker://wpengine/site-deploy@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yml from the mutable tag `docker://wpengine/site-deploy:1.0.5` to the immutable digest `docker://wpengine/site-deploy@sha256:9ca1f817b908acc0e8508ba26e3f8fdc927cb162d4b3c3f8a61882b150e0dd68 # 1.0.5`. The original tag is preserved as a comment for readability.

