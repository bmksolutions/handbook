# Changelog System (org-wide pattern)

Every bmk-solutions project that has a user-facing dashboard implements a
changelog. The pattern is:

1. **Source of truth**: `CHANGELOG.md` at repo root, in [Keep a Changelog](https://keepachangelog.com) format.
2. **Authoring**: each PR adds a bullet under `[Unreleased]`. Internal-only changes use the `no-changelog` label
instead.
3. **Promotion**: on push to `master`, a CI workflow promotes `[Unreleased]` → dated section and commits back.
4. **Publishing**: on every deploy, `php artisan changelog:publish` syncs the file to a `changelogs` DB table.
5. **Display**: a dashboard component (typically Livewire for our Laravel projects) shows the entries with per-user
unread tracking.
6. **Staging preview**: on non-production envs, `[Unreleased]` content is rendered as a "Staged for next release"
preview row so the team can verify wording before shipping.

## Required files in each project

- `CHANGELOG.md` (root)
- `app/Console/Commands/PublishChangelogCommand.php`
- `app/Models/Changelog.php` and `app/Models/ChangelogRead.php`
- Migrations for `changelogs` and `changelog_reads` tables
- `.github/workflows/changelog-check.yml` (PR enforcement, see `workflow-templates/`)
- `.github/workflows/release.yml` (promotion job, project-specific)
- A dashboard component for display

## Reference implementation

See `edco/docs/changelog-implementation.md` for a full Laravel 8 + Livewire 2
implementation including schema, model code, artisan command, deployment
integration for ploi.io, and edge cases.

## Pattern rules

- Versions are date-based (`YYYY.MM.DD`), not semver.
- Changelog entries are user-facing: describe what changed for the user, not
what was refactored internally.
- Allowed section headings: `Added`, `Changed`, `Fixed`, `Removed`, `Deprecated`, `Security`.
- Promotion only happens on production. Staging mirrors `[Unreleased]` as a preview row.
- Preview rows never count toward unread badges.

## Why this pattern (and not alternatives)

- **Manual `CHANGELOG.md` curation, not auto-generated from commits**: commit
messages are noisy; user-facing notes need to be intentional.
- **DB sync, not file-served-at-runtime**: enables per-user unread tracking,
cheap to query, file changes only on deploy.
- **Date-based, not semver**: SaaS dashboards don't have meaningful versions.
Date-based "what shipped this day" is what users actually care about.

For deeper rationale see [`decisions/0001-changelog-pattern.md`](./decisions/0001-changelog-pattern.md).
