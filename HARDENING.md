<!-- markdownlint-disable -->

# Hardening Report: wpengine--github-action-wpe-site-deploy/v3.2.9

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **wpengine--github-action-wpe-site-deploy/v3.2.9** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action's Docker image reference uses a mutable tag (`1.0.7`) instead of an immutable SHA digest. This means the image could be replaced with a different (potentially malicious) version without changing the action.yml. The reference `docker://wpengine/site-deploy:1.0.7` should be pinned to a SHA digest, e.g. `docker://wpengine/site-deploy@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:47`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yml from the mutable tag `docker://wpengine/site-deploy:1.0.7` to the immutable digest `docker://wpengine/site-deploy@sha256:dffc860dbbaeac16be910d52c3a5c7890e71f0deceb13b586b9705e02463d114 # 1.0.7`. The original tag is preserved as a comment for readability.

