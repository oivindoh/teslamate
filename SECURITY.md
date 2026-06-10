# Security Policy

## Reporting a Vulnerability

For reporting a security vulnerability, please contact `security AT teslamate DOT org`.

## Verifying published container images

Every TeslaMate image published to Docker Hub (`teslamate/teslamate`,
`teslamate/grafana`) and GitHub Container Registry
(`ghcr.io/teslamate-org/teslamate`, `ghcr.io/teslamate-org/teslamate/grafana`)
ships with two signed attestations, generated keylessly via Sigstore using the
GitHub Actions OIDC identity of this repository:

- **SLSA build provenance** — proves the image was built by the
  `teslamate-org/teslamate` GitHub Actions workflow, from a specific commit.
- **SPDX SBOM** — software bill of materials for the image contents.

### Verify with `gh`

```sh
gh attestation verify \
  oci://docker.io/teslamate/teslamate:latest \
  --repo teslamate-org/teslamate
```

Use `oci://ghcr.io/teslamate-org/teslamate:latest` for the GHCR image, and the
`grafana` variants similarly. The `gh` CLI resolves the tag to the multi-arch
manifest digest and verifies the provenance attached to it.

### Verify with `cosign`

```sh
cosign verify-attestation \
  --type slsaprovenance \
  --certificate-identity-regexp '^https://github\.com/teslamate-org/teslamate/' \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  docker.io/teslamate/teslamate:latest
```

### Verifying per-platform SBOMs

SBOMs are attached to each per-platform image digest (their natural scope —
filesystem contents differ per architecture). To inspect, resolve the platform
digest first:

```sh
docker buildx imagetools inspect docker.io/teslamate/teslamate:latest \
  --format '{{ range .Manifest.Manifests }}{{ .Platform.Architecture }} {{ .Digest }}{{ "\n" }}{{ end }}'

gh attestation verify \
  oci://docker.io/teslamate/teslamate@sha256:<platform-digest> \
  --repo teslamate-org/teslamate \
  --predicate-type https://spdx.dev/Document
```
