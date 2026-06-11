<!-- markdownlint-disable -->

# Hardening Report: wpengine--github-action-wpe-site-deploy/v3.2.7

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **wpengine--github-action-wpe-site-deploy/v3.2.7** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action's Docker image reference uses a mutable tag (`1.0.5`) instead of a SHA digest. This means the image can be silently replaced with a different (potentially malicious) version without any change to the action.yml. The reference `docker://wpengine/site-deploy:1.0.5` should be pinned to a specific SHA digest, e.g. `docker://wpengine/site-deploy@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:46`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned the Docker image reference in action.yml from the mutable tag `wpengine/site-deploy:1.0.5` to the immutable digest `wpengine/site-deploy@sha256:9ca1f817b908acc0e8508ba26e3f8fdc927cb162d4b3c3f8a61882b150e0dd68`. The original tag is preserved as a comment for readability.

