# n8n major-version mirror

Mirror official [n8n](https://github.com/n8n-io/n8n) Docker images to your own **GitHub Container Registry (GHCR)** under a **moving major tag** (`:2`), so you can run n8n from GHCR and let **Watchtower** pick up **new patch and minor releases** without changing image tags.

## Why

- You run n8n (and task runners) in Docker and want **automatic updates** for new **non-major** releases (same major line).
- Watchtower updates when the **tag you use** points at a **new image digest**. A stable tag like `:2` that you refresh from upstream does that; you avoid pinning to a full semver string while still **not** following a hypothetical next major (e.g. `3`) until you change tags or workflow rules.

## How it works

1. **GitHub Actions** ([`.github/workflows/n8n-minor-version-mirror.yml`](.github/workflows/n8n-minor-version-mirror.yml)) runs **daily** (UTC midnight) or on **manual dispatch**.
2. It pulls `n8nio/n8n:latest` and `n8nio/runners:latest` from Docker Hub.
3. **n8n**: reads `org.opencontainers.image.version` on the image; **only if** it matches `2.*`, it tags and pushes `ghcr.io/<repository-owner>/n8n:2`.
4. **Runners**: tags and pushes `ghcr.io/<repository-owner>/runners:2` (aligned with upstream `runners:latest`).

So `n8n:2` tracks upstream **while the published `latest` app image stays on major 2**. Semver boundaries for the app are enforced by that check, not by Watchtower itself.

## Local stack

[`docker-compose.yml`](docker-compose.yml) is an example that uses the mirrored images. Replace `your-username` with your GitHub user or org (same as `repository_owner` on GHCR), set Postgres and n8n **environment** variables, and ensure the **external network** exists if you keep that layout.

For Watchtower to **recreate** containers when GHCR serves a new digest for `:2`, configure Watchtower accordingly (the example labels use `monitor-only`; remove or change that if you want automatic upgrades instead of monitor-only).

## Requirements

- Repository **Actions** enabled and permission to **push packages** to GHCR for this repo (the workflow uses `GITHUB_TOKEN` with `packages: write`).
