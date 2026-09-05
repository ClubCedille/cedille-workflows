# CEDILLE Github Workflows Repository

This repository is a collection of Github workflows that can be used to automate
various tasks in a CEDILLE project. The workflows are written in YAML and are
stored in the `.github/workflows` directory. The workflows are triggered by
various Github events, such as pushes to the repository, pull requests, etc.

## kube-score gate (`kube-score-validator.yml`)

`kube-score-validator.yml` builds every changed kustomization root (`kubectl
kustomize --enable-helm`) and runs `kube-score` on the result. Any
`[CRITICAL]` finding fails the job unless it's explicitly skipped with a
`/kube-score skip <object> <check>` (or `/kube-score probe-skip ...` for
probe-related findings) PR comment.

**Blocking checks** (all repos, all resources): missing liveness/readiness/
startup probes, privileged containers, and any check not explicitly
downgraded below.

**Downgraded to `::warning::` (non-blocking) on resources built from a
kustomization tree that uses `helmCharts:`** — consuming repos don't control
upstream chart templates, so these checks are informational there instead of
gating:

- `container-resources`
- `container-ephemeral-storage-request-and-limit`
- `container-security-context-user-group-id`
- `pod-networkpolicy`
- `container-image-tag`
- `container-image-pull-policy`

This is a backstop, not a replacement for fixing findings at the source:
consuming repos are still expected to inject a `kube-score/ignore: <checks>`
annotation via a kustomize `patches:` block on resources they know are noisy
(see `k8s-shared/README.md#politique-kube-score` in `ClubCedille/k8s-shared`
for the pattern and worked examples). The gate-level downgrade above exists
for findings that show up before someone gets around to adding that patch —
e.g. right after a chart version bump.
