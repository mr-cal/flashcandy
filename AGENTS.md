# agents.md

## Before completing any task

This is a static HTML site — there's no build, lint, or test step. Just make
sure any HTML/CSS/asset changes render correctly (open the file locally or
check relative links/paths still resolve), then commit and push.

## Site content

All production content lives in `public/`, mirroring the docroot served by
Caddy: top-level HTML pages, `img/` (product photos), and `src/` (compiled
stylesheet plus downloadable Gerbers/BOMs/schematics). Keep relative paths
intact — pages reference assets like `src/theme.scss.css` directly.

## Production

The VPS for this project is managed by the `mr-cal/vps-infra` repo on GitHub.
Pushing to `main` triggers `.github/workflows/deploy.yml`, which dispatches a
`flashcandy-updated` `repository_dispatch` event to
[mr-cal/vps-infra](https://github.com/mr-cal/vps-infra). The vps-infra deploy
workflow pulls this repo as a git submodule and Caddy serves `public/`. Verify
the vps-infra deploy job succeeded after pushing commits.

A `VPSINFRA_PAT` secret must be set on this repo (Settings → Secrets and
variables → Actions) with a fine-grained PAT scoped to `mr-cal/vps-infra` with
**Contents: Read and write**.

## Key config files

- `.env.llm` — information for connecting to the cPanel host (during
  migration) and the VPS for debugging and pushing changes to git repos. Not
  committed.
