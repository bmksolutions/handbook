# Documentation Conventions

How bmk-solutions organizes project documentation across all repos so that humans *and* Claude can find, trust, and update it.

## Goals

- A new developer (or Claude session) can land in any of our repos and orient themselves in under 5 minutes.
- Decisions and their *reasoning* survive code changes, team changes, and time.
- Docs stay close to the code they describe — no separate wiki to drift out of sync.
- Claude reliably picks up project-specific conventions without us having to repeat them in every prompt.

## The Three-Layer Structure

Every project gets the same three layers. Adding more is fine; skipping any of these is not.

```
repo-root/
├── CLAUDE.md              ← Layer 1: the map
├── README.md              ← human-facing onboarding (existing convention)
├── CHANGELOG.md           ← user-facing release notes (see changelog-system.md)
├── .github/
│   └── workflows/
│       └── sync-handbook.yml   ← weekly auto-sync of docs/handbook/ (see Cross-Project Conventions)
└── docs/
    ├── conventions.md     ← Project-specific overrides ONLY (kept short)
    ├── handbook/          ← Auto-synced mirror of bmk-solutions/handbook (DO NOT hand-edit)
    │   ├── conventions.md
    │   └── ...
    ├── .handbook-version  ← Single line, e.g. "v1.3" — tracks what's synced
    ├── <feature>.md       ← Layer 2: feature/system docs
    ├── <feature>.md
    └── decisions/         ← Layer 3: ADRs
        ├── 0001-<slug>.md
        ├── 0002-<slug>.md
        └── ...
```

### Layer 1 — `CLAUDE.md` (the map)

Auto-loaded into every Claude session. **Treat it as a map, not a manual.** One-line pointers to where things live; Claude follows them.

Target: under 150 lines. If it grows past that, it's becoming a manual — extract content into `docs/`.

Required sections:

- **Stack** — framework + version, language version, frontend tooling, key packages with non-obvious behavior.
- **Conventions** — link to `docs/handbook/conventions.md` (canonical) and `docs/conventions.md` (project overrides).
- **Features** — bulleted index of `docs/<feature>.md` files with one-line descriptions.
- **Decisions** — link to `docs/decisions/` and a one-line note on the ADR format.
- **Deployment** — name the targets and link to any deploy docs. See `docs/handbook/deployment-playbook.md` for org-wide conventions.
- **Local setup** — only if non-obvious (point to README otherwise).

There is no canonical `CLAUDE.md` template in this handbook on purpose: every project has a different stack, deploy target, and feature set. Instead, when bootstrapping a new project, ask Claude to generate the initial `CLAUDE.md` by reading this conventions file and the project's existing structure (`composer.json`, `package.json`, framework config). The output should follow the section list above; review it, trim the noise, and commit.

### Layer 2 — `docs/<feature>.md` (feature plans & systems)

One file per feature, integration, or subsystem. Examples:

- `docs/<feature>-implementation.md` — concrete implementation plans (schema, classes, edge cases).
- `docs/<integration>.md` — third-party integration details (API contracts, auth, error handling).
- `docs/<subsystem>.md` — cross-cutting subsystems (auth-and-permissions, billing, notifications).

Recommended sections (tailor to the feature, don't force every section):

1. **Goals** — what this feature is *for*.
2. **Non-goals** — what it intentionally is *not*. This is the section that prevents scope creep in 6 months.
3. **Architecture** — components, data flow, key models. Diagrams welcome (Mermaid renders in GitHub).
4. **Implementation details** — schema, key classes, edge cases.
5. **Deployment / operations** — environment-specific behavior, monitoring, rollback.
6. **Future extensions** — things deliberately deferred. Stops "we should add X" from getting lost or rebuilt from scratch later.

Length: as long as it needs to be. A 400-line feature doc is fine if every section earns its place. If a doc becomes 1000+ lines it's probably two features.

### Layer 3 — `docs/decisions/` (Architecture Decision Records)

Short, numbered, immutable-ish records of *why* something was chosen. Industry standard format ([adr.github.io](https://adr.github.io)).

**Filename**: `NNNN-kebab-case-slug.md` — never renumber. If a decision is reversed, write a new ADR that supersedes the old one and link both ways. The old one stays as historical context.

The canonical template is in `adr-template.md`.

**When to write an ADR**:

- A choice that's hard to reverse (database engine, framework version, auth provider).
- A choice that surprises someone reading the code (e.g. "why store changelogs in DB instead of reading the file?").
- A choice that ruled out a tempting alternative (so future contributors don't burn time re-evaluating).

**When NOT to write an ADR**:

- Choices fully captured by the code itself (naming conventions, file layout — those go in conventions if they need recording).
- Reversible day-to-day decisions (which utility class to use, how to format a query).

## Naming & Style

- **Filenames**: kebab-case `.md`. No spaces, no uppercase except the conventional `CLAUDE.md`, `README.md`, `CHANGELOG.md`.
- **Headings**: sentence case, not title case. `## Edge cases and decisions`, not `## Edge Cases And Decisions`.
- **Code blocks**: always tag the language (` ```php `, ` ```bash `, ` ```yaml `) so Claude and syntax highlighters parse them correctly.
- **Links**: prefer relative links to other docs (`docs/<feature>.md`) over absolute URLs. Survives repo renames and forks.
- **Dates**: always absolute (`2026-04-28`), never relative ("last week"). Relative dates rot.

## Writing for Claude (and humans)

The same things make docs Claude-readable and human-readable:

1. **State the goal first.** Claude (and tired humans) skim. The first paragraph should tell the reader why this doc exists.
2. **Use concrete examples over abstract descriptions.** A code block beats two paragraphs of prose.
3. **Name files and symbols precisely.** Write `App\Models\Changelog::scopeVisible()`, not "the visible scope". Claude can grep for the former.
4. **Link forward and backward.** When a doc references another, hyperlink it. Claude follows links; isolated files get missed.
5. **Date your assumptions.** If a doc says "Postgres 14 is required", note when that was true. Future Claude will check whether it still holds.
6. **Keep one concept per doc.** Don't merge "changelog" and "release process" into one file just because they're related. Cross-link instead.

## Cross-Project Conventions

This handbook is the canonical source for cross-cutting conventions. Each consumer project mirrors its content into `docs/handbook/` via an automated weekly GitHub Action.

### Architecture

- **Canonical source**: `github.com/bmk-solutions/handbook` (this repo).
- **Mirrored into each project at**: `docs/handbook/` (auto-synced — projects must not hand-edit).
- **Project-specific overrides**: `docs/conventions.md` (a thin file in each project, kept short).
- **Sync state**: `docs/.handbook-version` (single line — the handbook git tag the project is synced to).
- **Versioning**: handbook releases are git tags (`v1.0`, `v1.1`, ...). Cosmetic edits to `main` don't trigger downstream syncs — only tagged releases do.
- **Sync trigger**: weekly cron in each project's GitHub Action; manually triggerable via `workflow_dispatch`.

### Project file layout

In each consumer project:

```
docs/
├── conventions.md          ← Project-specific overrides ONLY
├── handbook/               ← Auto-synced mirror of this repo (DO NOT hand-edit)
│   ├── conventions.md
│   ├── deployment-playbook.md
│   ├── changelog-system.md
│   ├── adr-template.md
│   └── pr-template.md
├── .handbook-version       ← Single line, e.g. "v1.3"
└── <feature>.md            ← Project-specific feature docs
```

The project's `docs/conventions.md` is a thin overrides file:

```markdown
# Project-specific convention overrides

> Canonical handbook is mirrored at `docs/handbook/conventions.md`.
> Synced version: see `docs/.handbook-version`.

## Overrides
<!-- Anything that diverges from the canonical handbook for this project.
     Keep this short — if an override would benefit other projects too,
     propose the change upstream in the handbook repo. -->

- (none yet)
```

### The sync workflow (required in every project)

The canonical workflow YAML lives in this handbook at `workflow-templates/sync-handbook.yml`. Each consumer project copies it verbatim into `.github/workflows/sync-handbook.yml`. No per-project customization needed — the workflow reads the consuming repo's own `docs/.handbook-version` and the handbook's latest tag.

What the workflow does:

1. Runs every Monday at 09:00 UTC, plus on manual `workflow_dispatch` trigger.
2. Reads the latest git tag from this handbook repo.
3. Reads `docs/.handbook-version` from the consumer project.
4. If they differ:
   - Wipes and re-clones `docs/handbook/` at the latest tag.
   - Updates `docs/.handbook-version`.
   - Opens (or force-pushes to) a single PR titled `Sync handbook to vX.Y`.
5. If they match, exits quietly. No PR, no noise.

The PR is the human review gate. Always review before merging — the handbook may have changed in ways that don't fit the project.

### Why git tags, not commit SHAs

If syncs triggered on every commit to `main` in the handbook, every typo fix would fan out into N PRs across every project. Tags let the handbook maintainer decide when changes are "ready for everyone". Cosmetic edits stay on `main` without triggering anything; intentional releases get tagged and propagated.

### Why a PR (not auto-merge)

- Convention changes deserve a glance. The PR is cheap to skim.
- A force-pushed sync branch means PRs never pile up — there's always at most one open sync PR per project, always reflecting the latest handbook tag.
- If a handbook change conflicts with a project-specific need, the PR is the place to reconcile (either decline the bump and pin to an older tag, or update the project's overrides to match the new direction).

### The `no-changelog` label

The sync PR is internal docs maintenance, not a user-facing change. The workflow labels it `no-changelog` so it bypasses the changelog enforcement check (see `changelog-system.md`).

## When Conventions Change

### Changing the canonical handbook

1. Open a PR against `bmk-solutions/handbook`.
2. Get review from at least one other engineer (conventions affect every project — second pair of eyes is cheap).
3. Merge to `main`.
4. Bump this handbook's `CHANGELOG.md` and tag a new version: `git tag v1.X && git push --tags`.
5. Within ~7 days every consumer project will get an auto-sync PR. Manually trigger `workflow_dispatch` on a specific project if you want it instantly.

### Changing a project-specific override

1. Edit the project's `docs/conventions.md` (the overrides file).
2. Open a normal PR with title `docs(conventions): <what changed>`.
3. If the change feels like it should be the org-wide default, after merging open a follow-up PR upstreaming it to this handbook.

### Reverting a handbook change for one project

If a handbook v1.X change doesn't suit a particular project:

1. Decline the auto-sync PR.
2. Either pin `docs/.handbook-version` to the previous tag manually and document why in the project's `docs/conventions.md` overrides, or upstream a fix to the handbook so the next version works for everyone.
3. The auto-sync action will keep proposing newer tags until you either accept or change the pinned version. The recurring PR is the nag mechanism.

## Anti-Patterns to Avoid

| Anti-pattern | Why it hurts |
|--------------|--------------|
| Docs in a separate wiki (Confluence, Notion) for code-related concerns | Drifts immediately; Claude can't read it; PR reviewers don't see it. |
| One giant `ARCHITECTURE.md` | Nobody updates it; nobody trusts it. Split into per-feature docs. |
| ADRs that document *what* the code does instead of *why* | The code already shows what. Docs exist for the why. |
| Duplicate `CHANGELOG.md` and `docs/changelog.md` | Pick one role per filename. `CHANGELOG.md` is the user-facing release notes. The implementation plan lives in `docs/<feature>-implementation.md`. |
| Renaming an ADR after the fact | Breaks links and historical context. Write a superseding ADR instead. |
| Embedding screenshots without alt text | Claude can't see images; alt text describes the content for both Claude and screen readers. |
| Markdown without language-tagged code fences | ` ``` ` instead of ` ```php ` loses syntax highlighting and Claude's language detection. |
| Hand-editing files inside `docs/handbook/` | The next sync overwrites your changes. Edit upstream in this handbook repo, or add an override to the project's `docs/conventions.md`. |
| Letting `docs/.handbook-version` drift from the actual content of `docs/handbook/` | Breaks the auto-sync detection. Always let the workflow update both together; never edit `.handbook-version` by hand unless intentionally pinning. |

## Bootstrapping a New Project

When starting a new project, before writing any feature code:

1. Create `CLAUDE.md` by asking Claude to draft one from this conventions file plus the project's stack files (`composer.json`, `package.json`, framework config). Review and trim. It should reference `docs/handbook/conventions.md` for org-wide rules and `docs/conventions.md` for project overrides.
2. Create `docs/conventions.md` as a thin overrides file (template above in § Cross-Project Conventions).
3. Add `.github/workflows/sync-handbook.yml` (copy from `workflow-templates/sync-handbook.yml` in this handbook).
4. Trigger the workflow once via `workflow_dispatch` to seed `docs/handbook/` and `docs/.handbook-version`. Merge the resulting PR.
5. Create `docs/decisions/0001-initial-stack.md` documenting the framework + key library choices (use `adr-template.md`).
6. Create `CHANGELOG.md` with an empty `[Unreleased]` section (see `changelog-system.md` for the org-wide pattern).
7. Add `.github/PULL_REQUEST_TEMPLATE.md` (copy from `pr-template.md` in this handbook).
8. If the project has user-facing changes, add the changelog enforcement workflow (`.github/workflows/changelog-check.yml`, copy from `workflow-templates/changelog-check.yml`).

This takes 30 minutes and saves weeks of "where is the doc for X?" later. Most of the time is waiting for the first handbook sync PR to land.

## See Also

- `changelog-system.md` — org-wide pattern for user-facing changelogs.
- `deployment-playbook.md` — environment conventions and deploy targets.
- `adr-template.md` — Architecture Decision Record template.
- `pr-template.md` — canonical PR template.
- `workflow-templates/sync-handbook.yml` — the workflow projects copy to enable auto-sync.
- `workflow-templates/changelog-check.yml` — PR enforcement for `CHANGELOG.md` updates.
- [Keep a Changelog](https://keepachangelog.com/) — format for `CHANGELOG.md`.
- [adr.github.io](https://adr.github.io) — Architecture Decision Records reference.
