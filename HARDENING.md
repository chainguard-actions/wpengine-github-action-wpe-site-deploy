<!-- markdownlint-disable -->

# Hardening Report: wpengine--github-action-wpe-site-deploy/v3.2.5

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **wpengine--github-action-wpe-site-deploy/v3.2.5** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action uses a Docker image referenced by a mutable tag rather than an immutable SHA digest: `image: docker://wpengine/site-deploy:1.0.3`. If the image at this tag is replaced or compromised upstream, the action will silently execute the new image without any indication of change. It should be pinned to a full SHA256 digest, e.g. `image: docker://wpengine/site-deploy@sha256:<64-hex-char-digest> # 1.0.3`.

Locations:

- `action.yml:49`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image `wpengine/site-deploy:1.0.3` to its immutable SHA256 digest `sha256:513f6cafe3119ddcb096340715c8098f26f6a598af702989b3875b2fd554bae0` in action.yml. The original tag is preserved as a comment for readability.

