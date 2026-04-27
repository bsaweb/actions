# bsaweb/actions

**Private** org repository: [reusable GitHub Actions workflows](.github/workflows/) for WordPress (Bedrock) sites — parity with the GitLab components under `bsaweb/ci` (wordpress + lftp).

## Workflows (callable)

| File | Role |
| --- | --- |
| [`wordpress-bedrock-build.yml`](.github/workflows/wordpress-bedrock-build.yml) | `composer install`, optional `npm ci` + `npm run build`, upload `bedrock-build` artifact. |
| [`lftp-deploy.yml`](.github/workflows/lftp-deploy.yml) | Download artifact, `lftp` mirror to FTPS (regex excludes, same file as `mirror_exclude_rx_from` on GitLab). |

**Pin a ref** in the caller (`@v1`, `@<sha>`, or branch name). Prefer **release tags** for production.

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
      ftp_host: ${{ secrets.STAGING_FTP_HOST }}
      ftp_login: ${{ secrets.STAGING_FTP_LOGIN }}
      ftp_password: ${{ secrets.STAGING_FTP_PASSWORD }}
```

`lftp` credentials must match the values you use today in GitLab CI variables. Reusable workflows in **private** other repos are allowed if [this repository allows access to workflows from private callers](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository#allowing-access-to-components-in-a-private-repository).

## Secrets (typical)

| Secret | Use |
| --- | --- |
| `COMPOSER_AUTH` | JSON for private Composer (same as local `auth.json` / GitLab). |
| `STAGING_FTP_HOST`, `STAGING_FTP_LOGIN`, `STAGING_FTP_PASSWORD` | Staging deploy. |
| `PRODUCTION_FTP_*` | Production, etc. |

## Notes (v1)

- Build uploads the **whole** checkout (including `.git`); the mirror step applies the same **regex** exclude file as on GitLab. Consider a slimmer artifact later to save minutes and upload size.
- `lftp` passwords containing **commas** may need an escape strategy or `.netrc` (follow-up if needed).
- Pivaut multi-target (e.g. Canada): add a second `deploy_*` job with `needs: [ build ]` and the correct secrets/paths; reuse the same `lftp-deploy` workflow.

## Development

- Clone: `git@github.com:bsaweb/actions.git`
- For quick YAML checks locally: [actionlint](https://github.com/rhasspy/actionlint) on `.github/workflows/*.yml` if installed.
