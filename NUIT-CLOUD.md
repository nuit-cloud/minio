# The Nuit Cloud fork of minio

The Nuit Cloud platform runs the MinIO server. MinIO no longer publishes images anyone can pull (Docker Hub went first; quay.io answers 401 since 2026)
and github.com/minio/minio is archived, so the platform builds its own image from this fork and pins it by digest:
`ghcr.io/nuit-cloud/minio`.

## What is here

`main` (the default branch) is upstream's last release, `RELEASE.2025-10-15T17-29-55Z` (commit `9e49d5e7a648`), unchanged, plus:

- `Dockerfile.nuit-cloud`: compiles this repository's own source, cross-compiled on the builder's platform (no emulation), stamped the
  way upstream's release build stamps it.
- `.github/workflows/nuit-cloud-image.yml`: the only workflow. Upstream's workflows were removed, so nothing else spends runner time.
- this file.

Work is trunk-based: `main` is the only branch, and every change is a commit on it, with no feature branches. Nothing builds from any
other branch.

## Releasing

Build locally first (`docker buildx build -f Dockerfile.nuit-cloud --platform linux/amd64,linux/arm64 -o type=local,dest=out .`), then push a
tag `nuit-cloud/<release>`, for example `nuit-cloud/RELEASE.2025-10-15T17-29-55Z`. The workflow builds amd64 and arm64, pushes
`ghcr.io/nuit-cloud/minio:<release>` with an SBOM and provenance, signs it by digest with cosign and writes the digest to the run's
summary. The platform (github.com/nuit-cloud/nuit-cloud) pins that digest; moving to a new image is changing the digest there (deploy/CLAUDE.md).

## Security patches

Upstream will not fix anything again: security patches land here, on `main`, as ordinary commits (a Go dependency bump, a backported
fix). Release a patched build under a new tag, `nuit-cloud/<upstream release>-nc<n>` (the build stamps `--version` from the part up to
`Z`, and the image's revision label names the commit built). The longer-term plan is to move off MinIO altogether (an issue in
nuit-cloud/nuit-cloud).
