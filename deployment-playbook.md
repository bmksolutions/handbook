# Deployment Playbook

## Environments

Every project has at minimum:
- **Local** — developer machine. `APP_ENV=local`.
- **Staging** — internal preview. `APP_ENV=staging`. Deploys from the `staging` branch.
- **Production** — user-facing. `APP_ENV=production`. Deploys from the `master` branch.

## Deploy targets

Primary: ploi.io for both staging and production. Each environment is a separate
ploi site with its own `.env`. Use ploi's deploy script feature; do not deploy by
SSHing to the box.

Fallback: manual `git pull` deploys are supported for emergency hotfixes only.
Document the exact commands in each project's `docs/deployment.md` if used.

## Required deploy script

Every ploi site's deploy script runs at minimum:

```bash
cd $SITE_DIRECTORY
git pull origin $BRANCH
composer install --no-dev --no-interaction --optimize-autoloader
php artisan migrate --force
php artisan changelog:publish     # see changelog-system.md
php artisan config:cache
php artisan route:cache
php artisan view:cache
```

## Branch model

- Feature work happens on short-lived branches off `staging`.
- PRs target `staging`, get reviewed, merge in.
- Staging is deployed automatically on push.
- Releases happen by merging `staging` → `master` (PR with the bundled changes).
- Production deploys automatically on push to `master`.

## Release cadence

No mandated cadence. Release when ready. The merge from `staging` → `master` is
the release event — make it deliberate (review the diff, the changelog entries,
and any ADRs that landed since the last release).

## Rollback

Revert the merge commit on `master` and let the deploy script run normally.
Database migrations are forward-only; if a rollback requires a schema change,
write a forward migration to undo.
