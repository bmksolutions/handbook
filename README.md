# bmk-solutions Engineering Handbook

Canonical conventions, templates, and patterns shared across all bmk-solutions
projects. Mirrored into each project at `docs/handbook/` via an automated weekly
sync (see `conventions.md` § Cross-Project Conventions).

## Files

- `conventions.md` — documentation structure, naming, writing-for-Claude rules.
- `deployment-playbook.md` — environment conventions (staging vs production, deploy targets).
- `changelog-system.md` — org-wide pattern for user-facing changelogs.
- `adr-template.md` — Architecture Decision Record template.
- `pr-template.md` — canonical PR template (copied to `.github/PULL_REQUEST_TEMPLATE.md` per project).
- `decisions/` — handbook-level ADRs (rationale for the patterns above).
- `workflow-templates/` — reusable GitHub Actions workflows.

## Repository visibility

This handbook is a **public** repository, and the sync setup assumes that. The choice is deliberate:

- Contents are conventions, templates, and patterns — nothing customer-specific or sensitive.
- Public means zero authentication overhead. Consumer projects clone and read tags from `https://github.com/bmksolutions/handbook.git` without tokens, secrets, or per-repo configuration.
- Organization-level secrets (the natural "private but no per-repo setup" path) are a paid-tier GitHub feature, so going public sidesteps that cost as well.
- A new bmksolutions project works the moment it copies `workflow-templates/sync-handbook.yml`. No setup steps an engineer can forget.

### If the handbook ever needs to go private

Two paths, in increasing order of polish:

1. **Org-level PAT secret** (requires GitHub Team or higher): create a fine-grained PAT scoped to this repo with `Contents: Read-only`, add it as an organization secret named `HANDBOOK_READ_TOKEN` with "All repositories" access, and patch the two `git ls-remote` / `git clone` steps in `workflow-templates/sync-handbook.yml` to authenticate via `https://x-access-token:${TOKEN}@github.com/bmksolutions/handbook.git`. One-time setup, yearly PAT renewal.
2. **GitHub App on the bmksolutions org** (further future upgrade): replaces the PAT with auto-rotated short-lived tokens. Org secrets become `HANDBOOK_APP_ID` and `HANDBOOK_APP_PRIVATE_KEY`; workflow uses `actions/create-github-app-token` to mint a token before the clone steps. No expiry to manage, not tied to any individual's account.

Don't pre-emptively go private. Until something in the handbook becomes genuinely sensitive, public is cheaper in every dimension that matters.

## Adopting in a new project

For a project that doesn't exist yet, before you write any feature code:

1. Initialize the repo with whatever framework starter applies (Laravel, etc.) and push to GitHub.
2. Add the sync workflow:
   ```bash
   mkdir -p .github/workflows
   curl -fsSL https://raw.githubusercontent.com/bmksolutions/handbook/main/workflow-templates/sync-handbook.yml \
     -o .github/workflows/sync-handbook.yml
   git add .github/workflows/sync-handbook.yml
   git commit -m "chore: add handbook sync workflow"
   git push
   ```
3. Trigger the workflow once via GitHub's UI ("Actions" → "Sync handbook" → "Run workflow"). Within a minute it opens a PR titled "Sync handbook to vX.Y" that creates `docs/handbook/` and `docs/.handbook-version`. Merge it.
4. Pull the merge locally so the handbook is on disk:
   ```bash
   git pull
   ```
5. Have Claude draft a `CLAUDE.md` based on `docs/handbook/conventions.md` and the project's stack files. Review, trim, commit.
6. Create `docs/conventions.md` as a thin overrides file (template in `docs/handbook/conventions.md` § Project file layout).
7. Create `docs/decisions/0001-initial-stack.md` documenting the framework + key library choices (use `docs/handbook/adr-template.md`).
8. Create `CHANGELOG.md` at repo root with an empty `[Unreleased]` section (see `docs/handbook/changelog-system.md`).
9. Add the PR template:
   ```bash
   cp docs/handbook/pr-template.md .github/PULL_REQUEST_TEMPLATE.md
   ```
10. If the project has user-facing changes, add the changelog enforcement workflow:
    ```bash
    cp docs/handbook/workflow-templates/changelog-check.yml .github/workflows/changelog-check.yml
    ```
11. Implement the changelog system itself (per `docs/handbook/changelog-system.md`).
12. Set up the deploy targets per `docs/handbook/deployment-playbook.md`.

## Adopting in an existing project

For a project that already has code, history, and possibly its own deploy setup:

1. **Audit what's already there.** Check for an existing `CLAUDE.md`, `README.md`, `CHANGELOG.md`, `docs/`, and any `.github/workflows/`. Understand what overlaps with the handbook before importing.
2. **Add the sync workflow** on a feature branch:
   ```bash
   git checkout -b chore/adopt-handbook
   mkdir -p .github/workflows
   curl -fsSL https://raw.githubusercontent.com/bmksolutions/handbook/main/workflow-templates/sync-handbook.yml \
     -o .github/workflows/sync-handbook.yml
   git add .github/workflows/sync-handbook.yml
   git commit -m "chore: add handbook sync workflow"
   git push -u origin chore/adopt-handbook
   ```
3. **Open a PR** for the workflow file, get review, merge.
4. **Trigger the workflow** via GitHub's UI to seed `docs/handbook/`. Merge the resulting auto-sync PR.
5. **Reconcile existing docs**:
   - If the project has a `CLAUDE.md`, compare it against the structure in `docs/handbook/conventions.md`. Add any missing required sections; leave the project-specific content alone.
   - If the project has `docs/` already, leave the existing files in place. Add `docs/conventions.md` as a thin overrides file noting where this project deviates from the handbook (e.g. "we don't use Livewire, see `docs/frontend.md`").
   - If existing ADRs use a different format, leave them alone — never renumber. New ADRs follow the handbook template going forward.
6. **Backfill `CHANGELOG.md` if missing.** If you've been releasing without one, start with a `[Unreleased]` block plus a single `[YYYY.MM.DD]` entry summarizing the current production state. Don't try to reconstruct full history — the changelog starts now.
7. **Add the PR template** if not already present:
   ```bash
   cp docs/handbook/pr-template.md .github/PULL_REQUEST_TEMPLATE.md
   ```
   If a template exists, merge the changelog reminder section into it.
8. **Add the changelog enforcement workflow** if relevant. On existing repos, expect failed checks for in-flight PRs that predate the workflow — apply the `no-changelog` label to grandfather them.
9. **Implement the changelog system** if the project doesn't have one yet (see `docs/handbook/changelog-system.md`). This is a separate, larger effort and can land in subsequent PRs.
10. **Audit deploy scripts** against `docs/handbook/deployment-playbook.md`. If `php artisan changelog:publish` isn't in the deploy script yet, add it once the changelog system is implemented (not before — the command won't exist).

Land changes incrementally. Adopting the handbook is not a single-PR migration; it's a sequence of small, reviewable changes. Order of dependency:

```
sync-handbook workflow  →  CLAUDE.md  →  PR template  →  CHANGELOG.md  →  changelog system  →  changelog-check workflow  →  deploy script update
```

## Adding an ADR

ADRs (Architecture Decision Records) document *why* a non-obvious choice was made. They live in two places:

- **Handbook-level**: `decisions/` in this repo — for choices that affect every project (e.g. `0001-changelog-pattern.md`).
- **Project-level**: `docs/decisions/` inside each consumer project — for choices specific to that project (framework version, integrations, etc.).

The folder structure both Claude and humans rely on:

```
decisions/
├── 0001-<kebab-slug>.md
├── 0002-<kebab-slug>.md
└── ...
```

Numbering is sequential, never reused, never renumbered. If a decision is reversed, write a new ADR that supersedes the old one and link both ways — the old ADR stays as historical context. Claude scans `decisions/` and `docs/decisions/` automatically when it needs to understand *why* something is the way it is, so the consistent folder name and `NNNN-slug.md` filename pattern matter.

### Steps

1. Find the next number: `ls decisions/ | tail -1` (or `ls docs/decisions/` for a project ADR).
2. Create `decisions/NNNN-<short-slug>.md` (or `docs/decisions/...` for a project).
3. Copy the body from `adr-template.md` (or `docs/handbook/adr-template.md` inside a project).
4. Fill in **Context**, **Decision**, **Consequences**, **Alternatives considered**.
5. Set **Status** to `Accepted` if the decision is final, or `Proposed` if still under discussion.
6. Commit. Handbook ADRs go through the standard handbook release flow (see below). Project ADRs merge with the PR that implements the decision — no special process.

### When to write one

- A choice that's hard to reverse (database engine, framework version, auth provider).
- A choice that surprises someone reading the code ("why is this in the DB instead of a file?").
- A choice that ruled out a tempting alternative, so future contributors don't burn time re-evaluating.

See `conventions.md` § Layer 3 for the full convention.

## Claude collaboration rules

Rules that apply to Claude (and other AI assistants) working in any bmk-solutions repo. These are enforced by review, not by tooling — be explicit if Claude breaks them.

- **Never co-author commits.** Do not add `Co-Authored-By: Claude ...` (or any AI assistant attribution) to commit messages. Commits are authored by the human who reviewed and shipped the change; AI involvement is a *how*, not a *who*. The commit history is a human record.

## Releasing the handbook

When you change content in this repo:

1. Open a PR. The handbook's own CI will check that `CHANGELOG.md` was updated (or that the PR has a `no-changelog` label) and that `workflow-templates/*.yml` is valid YAML.
2. Get review.
3. Merge to `main`.
4. Tag a new version:
   ```bash
   git tag v1.X
   git push --tags
   ```
5. Within ~7 days every consumer project gets an auto-sync PR. To propagate immediately, manually trigger `workflow_dispatch` on each consumer's "Sync handbook" workflow.

## Versioning

Tags are `MAJOR.MINOR`. Bump MAJOR for breaking convention changes (renaming
required files, changing required workflow paths, removing patterns). Bump
MINOR for additions and clarifications. Skip tagging for typo fixes — wait
until the next real change so consumer projects don't get noise.
