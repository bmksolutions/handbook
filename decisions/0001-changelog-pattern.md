# 0001. User-facing changelog pattern

- **Status**: Accepted
- **Date**: 2026-04-28
- **Deciders**: alexander

## Context

We want every bmk-solutions project that has a user-facing dashboard to expose
release notes inside the product, so users can see what's new and we can track
which entries each user has read. Several mechanical choices needed making:

1. **Where do entries come from?** Generated from git/commit messages, written
   by hand, or sourced from a separate tracking system?
2. **How are entries served?** Static file read at request time, or synced into
   the database?
3. **How are versions identified?** Semantic versioning, date-based, or
   release-tag-based?
4. **How does staging fit in?** Should staging show entries that haven't been
   released to production yet?

These choices were specifically shaped by the way we deploy: ploi.io for
staging and production, GitHub Actions for CI, occasional manual `git pull`
deploys as a fallback. Branches: feature → `staging` → `master`.

## Decision

We will adopt the following pattern across all projects:

1. **Source of truth**: a curated `CHANGELOG.md` at repo root in
   [Keep a Changelog](https://keepachangelog.com) format. Authors add bullets
   under `[Unreleased]` during their PR. Internal-only changes use a
   `no-changelog` PR label.
2. **Promotion**: a GitHub Actions workflow on push to `master` promotes
   `[Unreleased]` to a dated version section and commits the file back.
3. **Publishing**: `php artisan changelog:publish` runs on every deploy and
   syncs the file into a `changelogs` database table. Per-user read state lives
   in a `changelog_reads` table.
4. **Display**: a Livewire component (or equivalent for non-Laravel projects)
   shows the entries in a slide-over panel with an unread badge in the nav.
5. **Versioning**: date-based (`YYYY.MM.DD`), not semver.
6. **Staging preview**: on non-production environments, the `[Unreleased]`
   block is also synced as a preview row (`is_preview = true`) labeled "Staged
   for next release", visible to the team but invisible to real production
   users. Preview rows never count toward unread badges.

The full implementation pattern lives in `changelog-system.md` in this
handbook. A reference implementation for Laravel 8 + Livewire 2 lives in the
`edco` project at `docs/changelog-implementation.md`.

## Consequences

**Becomes easier:**

- Users see release notes inside the product without us shipping a separate
  marketing surface.
- Authoring is a one-bullet tax per PR — low friction once it's a habit.
- Per-user unread tracking is trivial because data is in the DB.
- Staging acts as a verification environment for both the *content* and the
  *rendering* of upcoming entries before they hit production.
- The `changelog:publish` command is idempotent, which means manual `git pull`
  fallback deploys work without ceremony.

**Becomes harder:**

- Authors must consciously distinguish user-facing changes from internal ones.
  We accept this as a desirable forcing function — it makes us think about
  users, not just code.
- Promotion requires a CI step that commits back to the repo. Production
  servers don't need write access to git, but CI does (via the default
  `GITHUB_TOKEN`).
- Schema and DB sync are more moving parts than a static `/changelog.json`
  file. Justified by per-user read state — see alternatives below.

**New constraints:**

- Every project with a dashboard now has a `CHANGELOG.md`, two migrations, a
  model, an artisan command, and a Livewire component as required artifacts.
  Listed in `changelog-system.md` § Required files.
- The `no-changelog` PR label must exist in every repo (or the enforcement
  workflow needs adjusting).
- Branch model assumed: `staging` and `master` exist, prod deploys from
  `master`. Projects that diverge need to adapt the promotion workflow.

## Alternatives considered

- **Auto-generate from Conventional Commits** (e.g. `release-please`,
  `semantic-release`): rejected because commit messages are noisy and shaped
  by code review, not user empathy. User-facing notes need to be written
  *for users*, which is a different mental mode than writing a commit message.

- **Static `changelog.json` served at runtime, no DB**: rejected because we
  want per-user unread tracking. A static file would force us to track read
  state in `localStorage` (loses across devices) or in a separate user table
  anyway, at which point the savings disappear.

- **Curated entries via a separate dashboard tool / Notion / external admin
  UI**: rejected because it creates a parallel surface that drifts from
  releases. Coupling the entry to the PR that ships the change is the only
  reliable way to keep them in sync.

- **Semver versioning**: rejected because our products are SaaS dashboards
  without meaningful version semantics for users. "What shipped on 2026-04-28"
  is the question users actually ask.

- **No staging preview row**: simpler, but loses the ability to verify wording
  and rendering before production. Adopted preview rows because the cost
  (one boolean column + an environment branch in the artisan command) is
  small and the validation benefit is real.

- **Push-based propagation of changelog updates** instead of deploy-time DB
  sync (e.g. webhook-driven): rejected because the deploy is the natural
  consistency boundary. Shipping changelog content separately from the code it
  describes risks showing users notes for behavior that isn't live yet.
