<!-- markdownlint-disable -->

# Hardening Report: wpengine--github-action-wpe-site-deploy/v3.2.6

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `1`

Action **wpengine--github-action-wpe-site-deploy/v3.2.6** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

The action's Docker image reference uses a mutable version tag (`1.0.4`) instead of an immutable SHA digest. This means the image pulled at runtime could change without notice, enabling supply-chain attacks. The reference `docker://wpengine/site-deploy:1.0.4` should be replaced with a pinned digest such as `docker://wpengine/site-deploy@sha256:<64-hex-char-digest>`.

Locations:

- `action.yml:48`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Replaced the mutable Docker image tag `docker://wpengine/site-deploy:1.0.4` with the immutable SHA256 digest `docker://wpengine/site-deploy@sha256:d976bb339d9443a7016ad26ae9d553535969d50cdd23918ec1d49ba1ba061b85 # 1.0.4` in action.yml at line 48. The original tag is preserved as a comment for readability.

