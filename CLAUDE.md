# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this repo is

The canonical source for bmk-solutions engineering conventions, templates, and reusable GitHub Actions. It is **docs + workflow YAML only** — there is no application code, no build, no test runner.

Every consumer project mirrors the contents of this repo into its own `docs/handbook/` via the workflow in `workflow-templates/sync-handbook.yml`. That mirror is read-only on the consumer side, so **changes here propagate to every project**. Treat edits accordingly.

## Propagation model (the thing that's easy to miss)

- Consumer projects sync on **git tags** (`v1.0`, `v1.1`, …), not commits to `main`. Pushing to `main` alone changes nothing downstream.
- Tag format is `MAJOR.MINOR`. Bump MAJOR for breaking changes (renaming required files, changing required workflow paths, removing patterns); bump MINOR for additions and clarifications. Skip tagging typo-only fixes.
- Release flow: PR → review → merge `main` → `git tag vX.Y && git push --tags`. Within ~7 days each consumer gets an auto-sync PR.
- Because the next sync overwrites `docs/handbook/` in every project, **never assume a consumer project hand-edited the mirror** — they can't.

## Repo-specific commands

There is no build or test step. CI (`.github/workflows/ci.yml`) enforces two things on PRs:

- `CHANGELOG.md` was updated under `[Unreleased]`, OR the PR has the `no-changelog` label.
- `workflow-templates/*.yml` and `.github/workflows/*.yml` are valid YAML (yamllint, relaxed config).

To reproduce the lint locally:

```bash
pip install yamllint
yamllint -d "{extends: relaxed, rules: {line-length: disable, truthy: disable}}" workflow-templates/ .github/workflows/
```

## Layout

- `conventions.md` — three-layer doc structure (`CLAUDE.md` map, `docs/<feature>.md`, `docs/decisions/`), naming rules, anti-patterns, and the cross-project sync architecture. Read this first when editing other handbook files.
- `changelog-system.md` — org-wide changelog pattern (Keep a Changelog format, `YYYY.MM.DD` versions, DB sync via `php artisan changelog:publish`).
- `deployment-playbook.md` — environments (local/staging/production), ploi.io deploy targets, branch model (`staging` → `master`).
- `adr-template.md` / `pr-template.md` — templates copied verbatim into consumer projects.
- `decisions/NNNN-<slug>.md` — handbook-level ADRs. Never renumber; supersede with a new ADR instead.
- `workflow-templates/` — reusable workflow YAML that consumer projects copy verbatim into `.github/workflows/`. Changes here are breaking if they rename the file or change required inputs.
- `README.md` — adoption guide for new and existing projects (consumer-facing).

## Working in this repo

- **Never add `Co-Authored-By: Claude` (or any AI attribution) to commit messages.** This applies in every bmk-solutions repo and is called out in `README.md` § Claude collaboration rules. The commit author is the human who reviewed the change.
- When a content edit could affect every consumer project, prefer minor, additive changes; avoid renames of files referenced by the README's adoption steps (`conventions.md`, `pr-template.md`, `adr-template.md`, `workflow-templates/sync-handbook.yml`, `workflow-templates/changelog-check.yml`) — those are paths consumers `curl` or `cp` from.
- Dates in docs are absolute (`2026-04-28`), never relative.
- Headings are sentence case. Code fences are language-tagged.
- Editing `workflow-templates/sync-handbook.yml` deserves extra care — it runs in every consumer repo on a weekly cron.
