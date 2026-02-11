# shared-workflows

Reusable GitHub Actions Workflows für Multi-Arch Docker Builds.

## Workflows

| Workflow | Beschreibung |
|----------|-------------|
| `prepare.yml` | Version-Info ermitteln (Commit Hash, Branch, Timestamp) |
| `build-image.yml` | Einzelnes Docker Image für eine Architektur bauen & pushen |
| `create-manifest.yml` | Multi-Arch Manifest aus einzelnen Arch-Images erstellen |

## Schnellstart


```yaml
jobs:
  prepare:
    uses: mogenius/shared-workflows/.github/workflows/prepare.yml@main

  build:
    needs: [prepare]
    strategy:
      matrix:
        include:
          - arch: amd64
            runner: ubuntu-latest
            needs_qemu: false
          - arch: arm64
            runner: ubuntu-latest
            needs_qemu: true
    uses: mogenius/shared-workflows/.github/workflows/build-image.yml@main
    with:
      image_name: ghcr.io/my-org/my-image
      arch: ${{ matrix.arch }}
      runner: ${{ matrix.runner }}
      needs_qemu: ${{ matrix.needs_qemu }}
      version: ${{ needs.prepare.outputs.version }}

  manifest:
    needs: [prepare, build]
    uses: mogenius/shared-workflows/.github/workflows/create-manifest.yml@main
    with:
      image_name: ghcr.io/my-org/my-image
      version: ${{ needs.prepare.outputs.version }}
      architectures: '["amd64", "arm64"]'
```

## Architektur hinzufügen / entfernen

Einfach die `matrix.include` und `architectures` anpassen:

**Nur amd64:**
```yaml
matrix:
  include:
    - arch: amd64
      runner: ubuntu-latest
      needs_qemu: false
# ...
architectures: '["amd64"]'
```

**amd64 + arm64 + armv7:**
```yaml
matrix:
  include:
    - arch: amd64
      runner: arc-runner-set-amd64
      needs_qemu: false
    - arch: arm64
      runner: arc-runner-set-arm64
      needs_qemu: false
    - arch: armv7
      runner: arc-runner-set-amd64
      needs_qemu: true
# ...
architectures: '["amd64", "arm64", "armv7"]'
```

## Projekt-spezifische Builder Images

Über `build_args` in der Matrix können pro Architektur und Projekt eigene Builder-Images übergeben werden:

```yaml
matrix:
  include:
    - arch: amd64
      runner: arc-runner-set-amd64
      needs_qemu: false
      build_args: |
        GO_BUILDER_IMAGE=ghcr.io/mogenius/go-builder:latest-amd64
        RUST_BUILDER_IMAGE=ghcr.io/mogenius/rust-builder:latest-amd64
    - arch: arm64
      runner: arc-runner-set-arm64
      needs_qemu: false
      build_args: |
        GO_BUILDER_IMAGE=ghcr.io/mogenius/go-builder:latest-arm64
        RUST_BUILDER_IMAGE=ghcr.io/mogenius/rust-builder:latest-arm64
```

## Manifest Tags

Der `create-manifest.yml` Workflow unterstützt flexible Tagging-Strategien:

| Input | Default | Beschreibung |
|-------|---------|-------------|
| `tag_latest` | `false` | `latest` Tag erstellen |
| `tag_sha` | `true` | Tag mit kurzem Commit SHA |
| `tag_branch` | `true` | Tag mit Branch-Name |
| `additional_tags` | `[]` | Zusätzliche Tags als JSON Array |

## Versionierung

Für Produktion empfohlen: Reusable Workflows über Tags referenzieren:

```yaml
uses: mogenius/shared-workflows/.github/workflows/build-image.yml@v1.0.0
```

## Beispiele

Siehe [`examples/`](./examples/) für vollständige Caller-Workflows.