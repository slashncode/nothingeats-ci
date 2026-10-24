# nothingeats-ci

Reusable GitHub Actions workflows and the shared Renovate preset for every [NothingEats](https://github.com/slashncode/nothingeats) repo.

## Workflows

| Workflow | What it does | Used by |
|---|---|---|
| [`kotlin.yml`](.github/workflows/kotlin.yml) | Build, ktlint and tests (Testcontainers); `verify-image` also builds the image tarball. On `main`: Gradle dependency graph submission. | Kotlin platform, Kotlin services, service template |
| [`kotlin-image.yml`](.github/workflows/kotlin-image.yml) | Multi-arch Jib image pushed to GHCR, signed with cosign (keyless), SPDX SBOM attestation. Tags `main-<sha>-<unix time>` for Flux image policies. Separate workflow so callers without images never grant `packages`/`id-token`. | Kotlin services |
| [`codeql.yml`](.github/workflows/codeql.yml) | CodeQL; `language: java-kotlin` (default, traced build) or `javascript-typescript` (no build) | every code repo |
| [`pnpm.yml`](.github/workflows/pnpm.yml) | `pnpm install --frozen-lockfile`, then the repo's `lint`, `typecheck`, `test` and `build` scripts; `browser-tests` installs Chromium for Vitest browser mode; `checkout-ui` builds nothingeats-ui next to the repo first | nothingeats-ui, web apps |
| [`web-image.yml`](.github/workflows/web-image.yml) | Builds the SPA, packs it with the repo's nginx Dockerfile into a multi-arch image on GHCR, cosign-signed with SBOM attestation, tag `main-<sha>-<unix time>` | web apps |
| [`pr-hygiene.yml`](.github/workflows/pr-hygiene.yml) | Conventional-commit PR titles, dependency review (fails on high severity) | every repo |

All third-party actions are pinned to commit SHAs; Renovate keeps the pins current.

Kotlin repos are built against the platform as a **composite build**: `kotlin.yml` checks out `nothingeats-kotlin-platform` next to the repo, the same layout as the local workspace. Inputs: `checkout-platform` (false for the platform itself), `platform-ref` (default `main`), `verify-image`.

### Example caller (a service)

```yaml
name: ci
on:
  push:
    branches: [main]
  pull_request:

permissions: {}

jobs:
  kotlin:
    uses: slashncode/nothingeats-ci/.github/workflows/kotlin.yml@main
    permissions:
      contents: write   # dependency graph submission
  image:
    if: github.event_name == 'push' && github.ref == 'refs/heads/main'
    needs: kotlin
    uses: slashncode/nothingeats-ci/.github/workflows/kotlin-image.yml@main
    permissions:
      contents: read
      packages: write   # GHCR
      id-token: write   # cosign keyless signing
  codeql:
    uses: slashncode/nothingeats-ci/.github/workflows/codeql.yml@main
    permissions:
      contents: read
      security-events: write
  pr:
    if: github.event_name == 'pull_request'
    uses: slashncode/nothingeats-ci/.github/workflows/pr-hygiene.yml@main
    permissions:
      contents: read
      pull-requests: write
```

## Renovate

Each repo has a `renovate.json`:

```json
{ "extends": ["github>slashncode/nothingeats-ci//renovate/default"] }
```

The preset pins action digests, uses semantic commits, groups Kotlin/KSP, Ktor and Exposed updates, and automerges GitHub Actions, patch and digest updates once CI is green. Renovate runs as the [Renovate GitHub App](https://github.com/apps/renovate).

## License

MIT
