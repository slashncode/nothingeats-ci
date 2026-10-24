# nothingeats-ci

Reusable GitHub Actions workflows and the shared Renovate preset for every [NothingEats](https://github.com/slashncode/nothingeats) repo.

## Workflows

| Workflow | What it does | Used by |
|---|---|---|
| [`kotlin.yml`](.github/workflows/kotlin.yml) | Build, ktlint and tests (Testcontainers). On `main`: Gradle dependency graph submission and, with `push-image`, a multi-arch Jib image pushed to GHCR, signed with cosign (keyless) and attested with an SPDX SBOM. Image tags are `main-<sha>-<unix time>` for Flux image policies. | Kotlin platform, Kotlin services, service template |
| [`codeql.yml`](.github/workflows/codeql.yml) | CodeQL for Java/Kotlin with a traced build | Kotlin repos |
| [`pr-hygiene.yml`](.github/workflows/pr-hygiene.yml) | Conventional-commit PR titles, dependency review (fails on high severity) | every repo |

All third-party actions are pinned to commit SHAs; Renovate keeps the pins current.

Kotlin repos are built against the platform as a **composite build**: `kotlin.yml` checks out `nothingeats-kotlin-platform` next to the repo, the same layout as the local workspace. Inputs: `checkout-platform` (false for the platform itself), `platform-ref` (default `main`), `verify-image`, `push-image`.

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
      packages: write   # GHCR
      id-token: write   # cosign keyless signing
    with:
      push-image: true
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
