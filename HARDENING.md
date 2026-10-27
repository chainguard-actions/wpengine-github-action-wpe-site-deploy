<!-- markdownlint-disable -->

# Hardening Report: wpengine--github-action-wpe-site-deploy/v3.2.10

> This file was generated automatically by the hardening agent.

**Policy SHA:** `d636be7e43ef829af6e853da6b3c7566db9f72fe`

**Test Policy SHA:** `843adf9e4b8f85d0c08b27b9d0b09dd094b54702`

**Harden Agent Version:** `2`

Action **wpengine--github-action-wpe-site-deploy/v3.2.10** was hardened automatically. 1 finding(s) were identified and resolved across 1 iteration(s).

## Findings Fixed

### unpinned-uses (severity: high)

This Docker action's runs.image points to a mutable tag, not a SHA digest: `image: docker://wpengine/site-deploy:1.0.9`. Pin it to a digest instead (e.g. `docker://wpengine/site-deploy@sha256:<digest> # 1.0.9`).

Locations:

- `action.yml:1208`

## Iteration Notes

### Iteration 1

**Fixes applied:** unpinned-uses

**Notes:**

Pinned runs.image to docker://wpengine/site-deploy:1.0.9@sha256:e44e7b1ef9edb18150022b4269ce462cc794a33b8186c334627e923ab16cbaad, keeping the docker:// scheme and the tag inline.

