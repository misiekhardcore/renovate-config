# renovate-config

Shared [Renovate](https://docs.renovatebot.com/) presets used across my projects.

Consuming repos reference these files directly from `main`:

```json5
{
  $schema: "https://docs.renovatebot.com/renovate-schema.json",
  extends: ["github>misiekhardcore/renovate-config//default.json5"],
}
```

Presets are read from `main`, so a change here takes effect for every repo on its next Renovate run. Open a PR rather than pushing to `main` directly.

## Presets

| Preset | What it does | Used by |
| --- | --- | --- |
| [`default.json5`](default.json5) | Base policy for every repo: `config:best-practices`, automerge limited to digest/pin/patch/minor, majors never automerge, weekday schedule, `prConcurrentLimit: 60`, `baseBranchPatterns: ["main"]`, `deps:` commit prefix, 3 day release age, generic JS/TS grouping (GitHub Actions, React, Vite/Vitest, Jest, Playwright, Tailwind, Postgres clients, repo tooling) | every repo below |
| [`presets/nextjs.json5`](presets/nextjs.json5) | Groups `next`, `eslint-config-next`, `@next/*` and `next-sitemap` into one PR so they never drift apart | a-star-pathfinding, color-scale-generator, traveling-salesman-genetic-algorithm, wave-function-collapse, loopytinker-worldmap, konopskidev, konopskiwebdev, terra-madera |
| [`presets/payload.json5`](presets/payload.json5) | Groups `payload` + every `@payloadcms/*` package into one exact-pinned PR so they stay on one version; groups auth and AWS SDK packages | konopskidev, konopskiwebdev, terra-madera |
| [`presets/docker.json5`](presets/docker.json5) | Groups routine Dockerfile/compose image updates, keeps engine majors manual, disables dev-only `:latest` services | konopskidev, konopskiwebdev, terra-madera |

`default.json5` is not pulled in implicitly - always list it first:

```json5
{
  $schema: "https://docs.renovatebot.com/renovate-schema.json",
  extends: [
    "github>misiekhardcore/renovate-config//default.json5",
    "github>misiekhardcore/renovate-config//presets/nextjs.json5",
  ],
}
```

## Usage notes

- **Repos without CI must opt out of automerge.** `:automergeRequireAllStatusChecks` only requires the checks that exist, so a repo with no workflows would merge every non-major update unattended. Set `"automerge": false` in that repo's config until it has CI.
- **Majors are never automerged.** They arrive as regular PRs labelled `major` (and `docker` for image majors). Renovate's dependency dashboard lists them if you want to defer.
- **`config:best-practices` pins devDependency ranges** (it includes `:pinDevDependencies`). A repo with `^`-ranged dev dependencies will get one large pinning PR on the first run.
- **The `.github/renovate.json5` file is the standard location.** `renovate-config-validator` picks it up automatically.

## Validation

[`.github/workflows/validate-renovate-config.yml`](.github/workflows/validate-renovate-config.yml) is a reusable workflow (`renovate-config-validator --strict`). Each repo calls it from a small workflow of its own:

```yaml
name: Validate Renovate config
on:
  pull_request:
    paths: ["**/renovate.json", "**/renovate.json5"]
jobs:
  validate:
    uses: misiekhardcore/renovate-config/.github/workflows/validate-renovate-config.yml@main
```

Locally:

```bash
npx --yes --package renovate -- renovate-config-validator --strict
```
