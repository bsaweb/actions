# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

Private org repository (`bsaweb/actions`) containing **reusable GitHub Actions** for WordPress/Bedrock projects. It is consumed by site repos via `uses: bsaweb/actions/...@<ref>`.

Three components:

| Path | Kind | Purpose |
|---|---|---|
| `.github/workflows/wordpress-bedrock-build.yml` | Reusable workflow (`workflow_call`) | Composer install + optional npm ci/build + artifact upload |
| `lftp-deploy/action.yml` | Composite action | Download artifact → install lftp → mirror to remote via FTPS |
| `rsync-deploy/action.yml` | Composite action | Download artifact → SSH agent → mirror to remote via rsync over SSH |

## Validation

Lint workflows with [actionlint](https://github.com/rhysd/actionlint):

```bash
actionlint .github/workflows/*.yml
```

There are no automated tests — validate by reading the YAML and tracing inputs/secrets manually.

## Action versioning

Callers pin a ref (`@v0.1.0`, `@<sha>`, or branch). Use release tags for production callers. When making breaking changes, bump the tag.

## Key design rules

- `lftp-deploy` **aborts** if the exclude file (`mirror-exclude-rx-from`) is declared but missing in the artifact — this prevents accidental `--delete` without filtering.
- `lftp-deploy` **aborts** if `mirror-remote-path` is empty or dangerous (`/`, `.`, `..`) — mirroring to the server root with `--delete` can remove system directories regardless of exclude patterns.
- `lftp-deploy` input `mirror-exclude` (default `web/app/uploads`) is passed to `mirror --exclude` in addition to `--exclude-rx-from`; pass an empty string to opt out.
- `rsync-deploy` follows the same abort rules as `lftp-deploy` for `mirror-remote-path` and the exclude file (`rsync-exclude-from`).
- `rsync-deploy` input `rsync-exclude` (default `web/app/uploads`) is passed to `rsync --exclude` in addition to `--exclude-from`; pass an empty string to opt out.
- Site repos should keep `.bsaweb/ci-cd/rsync-exclude.txt` (glob patterns) in sync with `lftp-exclude-regex.txt` (regex) when both deploy methods are used.
- Branch → environment mapping in `lftp-deploy.yml` (caller side): `main` → `production`, anything else → `staging`. `LFTP_*` secrets live on each GitHub Environment.
- Composer auth secrets are all optional; each registry is configured only when its secret is non-empty.
- The artifact excludes source/dev files (`.git`, `node_modules`, lock files, config files) but keeps hidden files (`include-hidden-files: true`).

## Secrets reference

| Secret | Scope | Use |
|---|---|---|
| `COMPOSER_BSAWEB_SATIS_REGISTRY_USER/PASSWORD` | repo/org | satis.bsa-web.fr basic auth |
| `COMPOSER_BSAWEB_SATISPRESS_REGISTRY_USER` | repo/org | composer.bsa-web.fr (SatisPress) |
| `COMPOSER_BSAWEB_GITLAB_REGISTRY_TOKEN` | repo/org | gitlab.bsa-web.fr VCS token |
| `COMPOSER_ECOLES_CREATIVES_GITLAB_REGISTRY_TOKEN` | repo/org | GitLab group Composer registry |
| `COMPOSER_GRAVITY_REGISTRY_USER/PASSWORD` | repo/org | composer.gravity.io (Gravity Forms) |
| `NPM_BSAWEB_REGISTRY_TOKEN` | repo/org | GitLab npm registry for `@bsaweb` packages |
| `LFTP_HOST`, `LFTP_USER`, `LFTP_PASSWORD` | per Environment | FTP/FTPS target (different values per staging/production) |
| `SSH_HOST`, `SSH_USER`, `SSH_PRIVATE_KEY`, `SSH_KNOWN_HOSTS`, `SSH_PORT` | per Environment | SSH/rsync target (optional; use with `rsync-deploy`) |
