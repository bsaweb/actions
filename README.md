# bsaweb/actions

**Private** org repository: [reusable GitHub Actions workflows](.github/workflows/) for WordPress (Bedrock) projects.

## Workflows (callable)

| File | Role |
| --- | --- |
| [`wordpress-bedrock-build.yml`](.github/workflows/wordpress-bedrock-build.yml) | `composer install`, optional `npm ci` + `npm run build`, upload an artifact (default `bedrock-build`). |
| [`lftp-deploy.yml`](.github/workflows/lftp-deploy.yml) | `lftp` mirror. The job uses `github.ref`: branch **main** → **production**, any other ref → **staging** (name your [Environments](https://docs.github.com/en/actions/deployment/targeting-different-environments/using-environments-for-deployment) accordingly). `LFTP_*` secrets live on each Environment. |

**Pin a ref** in the caller (`@v0.1.0`, `@<sha>`, or branch). Prefer **release tags** for production.

## Example (consumer repo)

`mirror_remote_path` and optional exclude path are the usual `with` values:

```yaml
  deploy:
    needs: [ build ]
    uses: bsaweb/actions/.github/workflows/lftp-deploy.yml@v0.1.0
    with:
      mirror_remote_path: /httpdocs
```

In **Settings → Environments**, add **staging** and **production** with `LFTP_HOST`, `LFTP_USER`, `LFTP_PASSWORD` (same names, different values per environment).

`lftp` credentials must be stored as environment secrets. Reusable workflows in **private** other repos are allowed if [this repository allows access to workflows from private callers](https://docs.github.com/en/repositories/managing-your-repositorys-settings-and-features/enabling-features-for-your-repository/managing-github-actions-settings-for-a-repository#allowing-access-to-components-in-a-private-repository).

Set `mirror_exclude_rx_from` if your regex exclude file is not at the default path in `lftp-deploy`.

## Secrets (typical)

| Secret | Use |
| --- | --- |
| `COMPOSER_AUTH` | JSON for private Composer (repo or org secret, often the same in all envs). |
| `LFTP_HOST`, `LFTP_USER`, `LFTP_PASSWORD` | FTP/FTPS target; set per **Environment** (same names, different values per `staging` / `production` / …). |

## Notes (v1)

- The build job uploads the **whole** checkout (including `.git`); the mirror step applies a **regex** exclude list. You can add a slimmer packaging step later to save time and space.
- `lftp` passwords containing **commas** may need a different auth approach (e.g. `.netrc`); follow up if you hit that.
- Several targets (e.g. staging + production, or another region): add a `deploy_*` job per target with `needs: [ build ]` and the right secrets/paths, reusing `lftp-deploy`.

## Development

- Clone: `git@github.com:bsaweb/actions.git`
- Optional: [actionlint](https://github.com/rhasspy/actionlint) on `.github/workflows/*.yml`.
