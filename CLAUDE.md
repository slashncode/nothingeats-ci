# CLAUDE.md

Reusable GitHub Actions workflows and the Renovate preset for all NothingEats repos. The cross-repo plan and session rules live in `../CLAUDE.md` (workspace root), if present — read it first.

## Commands

- `mise exec actionlint@1.7.12 -- actionlint` — lint all workflows (CI runs the same version in Docker, with shellcheck)
- `mise exec -- npx --yes --package renovate@44.181.3 -- renovate-config-validator --strict renovate/default.json` — validate the preset (needs Node 24 from the workspace `mise.toml`)

## Rules

- Pin every third-party action to a full commit SHA with the version as a comment (`@<sha> # v7.0.1`).
- Reusable workflows start with `permissions: {}` and grant per job only what that job needs; callers must grant at least the same.
- Never interpolate `${{ }}` expressions directly into `run:` scripts; pass them through `env:`.
- `actions/checkout` always with `persist-credentials: false`.
- Changes here affect every repo on their next run: keep inputs backwards compatible or update all callers in the same session.
