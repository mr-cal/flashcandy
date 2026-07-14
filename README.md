<h1 align="center">
 flashcandy
</h1>

<h4 align="center">Portable power supplies for prototyping and projects.</h4>

## Overview

Source code for https://flashcandy.us

Static site: `public/` holds the docroot as served in production (HTML pages,
`img/` product photos, and `src/` which contains the compiled stylesheet plus
downloadable PCB design files — Gerbers, BOMs, schematics, assembly drawings).

## Deployment

Pushing to `main` triggers `.github/workflows/deploy.yml`, which dispatches a
`repository_dispatch` event to [mr-cal/vps-infra](https://github.com/mr-cal/vps-infra).
The vps-infra deploy workflow SSHs to the VPS, pulls the latest commit from this
repo via git submodule, and Caddy serves the `public/` directory.

Add a `VPSINFRA_PAT` secret to this repo (Settings → Secrets and variables → Actions)
with a fine-grained PAT scoped to `mr-cal/vps-infra` with **Contents: Read and write**.

## Contributing

Feel free to submit an issue or PR!
