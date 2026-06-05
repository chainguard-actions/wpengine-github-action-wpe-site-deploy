# Hardening Report: wpengine--github-action-wpe-site-deploy/v3.2.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `ff50f15e4b79bfbf764dafdfd2579175a6ea9771`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **wpengine--github-action-wpe-site-deploy/v3.2.9** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action uses `runs.using: docker` with `image: docker://wpengine/site-deploy:1.0.7`. This references a mutable Docker image tag (`1.0.7`) rather than an immutable SHA digest (e.g., `docker://wpengine/site-deploy@sha256:<64-hex-digest>`). A mutable tag can be silently overwritten with a malicious image, creating a supply-chain attack vector.

Locations:

- `action.yml:48`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable Docker image tag `docker://wpengine/site-deploy:1.0.7` with the immutable digest `docker://wpengine/site-deploy@sha256:dffc860dbbaeac16be910d52c3a5c7890e71f0deceb13b586b9705e02463d114 # 1.0.7` in action.yml line 48. This pins the image to an exact, immutable version and prevents supply-chain attacks via tag mutation.

