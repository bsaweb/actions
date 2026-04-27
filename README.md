# bsaweb/actions

**Private** org repository: [reusable GitHub Actions workflows](.github/workflows/) for WordPress (Bedrock) projects.

## Workflows (callable)

| File | Role |
| --- | --- |
| [`wordpress-bedrock-build.yml`](.github/workflows/wordpress-bedrock-build.yml) | `composer install`, optional `npm ci` + `npm run build`, upload an artifact (default `bedrock-build`). |
| [`lftp-deploy.yml`](.github/workflows/lftp-deploy.yml) | Download artifact, `lftp` mirror to FTPS, regex-based excludes via `mirror_exclude_rx_from`. |

**Pin a ref** in the caller (`@v1`, `@<sha>`, or branch). Prefer **release tags** for production.

## Example (consumer repo)

Add `.github/workflows/deploy.yml` in a Bedrock project:

```yaml
on:
  push:
    branches: [ develop, main ]

# Avoid overlapping deploys.
concurrency:
  group: deploy-${{ github.ref }}-${{ github.workflow }}
  cancel-in-progress: true

jobs:
  build:
    uses: bsaweb/actions/.github/workflows/wordpress-bedrock-build.yml@v1
    secrets:
      COMPOSER_AUTH: ${{ secrets.COMPOSER_AUTH }}

  deploy_staging:
    if: github.ref == 'refs/heads/develop'
    needs: [ build ]
    uses: bsaweb/actions/.github/workflows/lftp-deploy.yml@v1
    with:
      environment: staging
      mirror_remote_path: /httpdocs
    secrets:
      site: ${{ secrets.STAGING_FTP_HOST }}
      user: ${{ secrets.STAGING_FTP_LOGIN }}
      password: ${{ secrets.STAGING_FTP_PASSWORD }}
```

`lftp` credentials must be stored as repository or environment secrets. Reusable workflows in **private** other repos are allowed if [this repository allows access to workflows from private callers](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository#allowing-access-to-components-in-a-private-repository).

Set `mirror_exclude_rx_from` if your regex exclude file is not at the default path in `lftp-deploy`.

## Secrets (typical)

| Secret | Use |
| --- | --- |
| `COMPOSER_AUTH` | JSON for private Composer (same as local `auth.json` / `COMPOSER_AUTH` env). |
| `STAGING_FTP_HOST`, `STAGING_FTP_LOGIN`, `STAGING_FTP_PASSWORD` (or your naming) | Map to `site` / `user` / `password` in the deploy workflow. |
| `PRODUCTION_FTP_*` | Production, etc. |

## Notes (v1)

- The build job uploads the **whole** checkout (including `.git`); the mirror step applies a **regex** exclude list. You can add a slimmer packaging step later to save time and space.
- `lftp` passwords containing **commas** may need a different auth approach (e.g. `.netrc`); follow up if you hit that.
- Several targets (e.g. staging + production, or another region): add a `deploy_*` job per target with `needs: [ build ]` and the right secrets/paths, reusing `lftp-deploy`.

## Development

- Clone: `git@github.com:bsaweb/actions.git`
- Optional: [actionlint](https://github.com/rhasspy/actionlint) on `.github/workflows/*.yml`.
