# Migrate flashcandy.us from cPanel to the VPS

## Problem

`flashcandy.us` is currently hosted on the shared cPanel account (`pcbigkhq` on
`server167.web-hosting.com`), same account that used to host `eggcalculator.com`.
DNS for `flashcandy.us` still resolves to the cPanel host (`162.0.212.2`), not the
VPS (`167.99.14.211`).

The site is fully static: `index.html`, `contact.html`, `faq.html`, `products.html`,
`big-nugget-v1/v2.html`, `little-nugget-v1/v2.html`, `sitemap.xml`, an `img/` folder
of product photos, and a `src/` folder that holds both the compiled stylesheet
(`theme.scss.css`) and downloadable PCB design files (Gerbers, BOMs, schematics,
assembly drawings) referenced from the product pages. No PHP/WordPress, no
database, `cgi-bin/` is empty.

## Approach

Follow the exact pattern already used for `eggcalculator.com` in `mr-cal/vps-infra`:
this repo holds the static site content and deploys itself via a
`repository_dispatch` to `vps-infra`, which mounts the repo as a git submodule and
serves it with the shared Caddy instance. No new server infrastructure is needed —
only additions to the existing VPS stack.

Repo layout decision: mirror the live docroot under `public/` (not `src/`, since the
site already has its own `src/` subfolder for CSS + Gerbers — reusing `src/` as the
repo's docroot wrapper would create a confusing `src/src/...` path).

## Todos

1. **Pull site content from cPanel** — use the cPanel UAPI (creds in `.env.llm`) to
   download every file/folder under `/home/pcbigkhq/flashcandy.us/` (top-level HTML
   files, `sitemap.xml`, `img/`, `src/`) into `flashcandy/public/` in this repo,
   preserving relative paths so existing `href`/`src` references keep working.

2. **Add repo scaffolding** (mirroring `egg-calculator`):
   - `.pre-commit-config.yaml` with the `gitleaks` hook.
   - `README.md` explaining the site and the deploy flow (push to `main` →
     dispatch to `vps-infra` → Caddy serves `public/`).
   - `AGENTS.md`, adapted (briefly) from `~/dev/craft/craft-dashboard/agents.md`:
     no build/lint/test commands (static HTML, nothing to run), just: where the
     site content lives (`public/`), how deploys work, and where `.env.llm` /
     `.env` fit in.
   - `.github/workflows/deploy.yml` — on push to `main`, dispatch a
     `flashcandy-updated` `repository_dispatch` event to `mr-cal/vps-infra`
     (same shape as `egg-calculator`'s).
   - `.gitignore` already excludes `.env*` — no change needed.

3. **Commit and push** the above to `mr-cal/flashcandy` `main`.

4. **Wire up `mr-cal/vps-infra`**:
   - Add `static-sites/flashcandy` submodule (`.gitmodules`) pointing at
     `mr-cal/flashcandy` `main`.
   - Add a `flashcandy.us` + `www.flashcandy.us` block to `caddy/Caddyfile`
     (root `/srv/flashcandy/public`, `file_server`, www→apex redirect).
   - Add `flashcandy-updated` to the `repository_dispatch` `types` list in
     `.github/workflows/deploy.yml`.
   - Add a curl health check for `flashcandy.us` in the deploy script's
     verification section, alongside the existing `eggcalculator.com` check.
   - Update `AGENTS.md`'s "Sites served" list.

5. **Add the `VPSINFRA_PAT` secret** to the `mr-cal/flashcandy` repo (Settings →
   Secrets and variables → Actions), a fine-grained PAT scoped to
   `mr-cal/vps-infra` with **Contents: Read and write** — same as `egg-calculator`.

6. **Push `vps-infra` changes** and confirm the deploy workflow succeeds: submodule
   pulled, Caddy restarted, `flashcandy.us` reachable over HTTP on the VPS via
   `curl --resolve` (TLS cert can't issue yet until DNS is cut over).

7. **Cut over DNS** — update the `flashcandy.us` A (and AAAA, if desired) record
   from `162.0.212.2` to `167.99.14.211` / the VPS's IPv6 address. **Blocked:
   registrar credentials for `flashcandy.us` (dns1/dns2.registrar-servers.com) are
   not in `.env.llm` — need those before this step, or you flip the record
   manually.**

8. **Verify post-cutover**: confirm Let's Encrypt issues a cert for
   `flashcandy.us`/`www.flashcandy.us`, all pages load, images render, and the
   Gerber/BOM/schematic downloads under `src/` still work.

9. **Decommission the cPanel copy** (optional, after the cutover has been stable
   for a while) — remove or archive `/home/pcbigkhq/flashcandy.us` via the cPanel
   API.

## Notes

- No database, no build step, no PHP — this is a straight file copy + DNS switch,
  simpler than the `pcbisolation` (WordPress→Hugo) migration.
- Reuse the exact `egg-calculator` deploy pattern rather than inventing something
  new, since `vps-infra` already supports multiple static-site submodules this way.
- Stop and ask before doing anything that needs credentials beyond what's in
  `.env.llm` (this already came up for the domain registrar in step 7).
