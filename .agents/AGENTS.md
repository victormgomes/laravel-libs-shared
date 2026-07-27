# AI Guidelines for `laravel-libs-shared`

You are an AI coding assistant helping to develop and maintain the
`victormgomes/laravel-libs-shared` infrastructure repository. This is **not** a
Laravel package — it has no `composer.json`, no `src/`, and no `tests/`. It is
the central source of truth for shared CI/CD workflows, configs, community health
files, GitHub templates, and AI guidelines used by all downstream repositories in
the `victormgomes` ecosystem.

## 1. Repository Identity

- **Purpose:** Infrastructure hub for the `victormgomes` Laravel ecosystem.
- **No application code:** This repo provides reusable workflows, file
  synchronization, shared configs, and governance rules — not a PHP package.
- **Canonical source:** Downstream repos (like `laravel-query-engine`) consume
  files from this repository via sync or repo-qualified `uses:` paths.

## 2. The Two Workflow Layers

This repository has two distinct directories for GitHub Actions workflows:

### `workflows/` (sync source)

- Files here are **templates** that get physically copied to downstream repos'
  `.github/workflows/` via `BetaHuhn/repo-file-sync-action`.
- These files are NOT executed directly in this repository.
- They contain the CI pipeline orchestration (`ci.yml`), docs deployment
  (`docs.yml`), auto-approve (`auto-approve.yml`), and branch cleanup
  (`branch-cleanup.yml`).

### `.github/workflows/` (reusable + local infrastructure)

- **Reusable workflows** (`on: workflow_call`): Called by downstream repos via
  repo-qualified paths like `victormgomes/laravel-libs-shared/.github/workflows/<name>.yml@main`.
- **Local infrastructure:** `sync.yml`, `sync-settings.yml` — run only in this
  repository.

**Never confuse these two directories.** Files in `workflows/` are synced
downstream. Files in `.github/workflows/` are either reusable or local-only.

## 3. The Sync Mechanism

`.github/sync.yml` defines what gets synced where using
`BetaHuhn/repo-file-sync-action`.

### Adding a new synced file

1. Create the source file in the appropriate directory:
   - `configs/` → repo root (for config files)
   - `community/` → `.github/` (for community health files)
   - `github-templates/` → `.github/` (for GitHub templates)
   - `agents/` → `.agents/` (for AI guidelines)
   - `workflows/` → `.github/workflows/` (for CI workflows)
2. Add a source-to-destination mapping in `.github/sync.yml`:
   ```yaml
   - source: <source-directory>/<filename>
     dest: <destination-path>
   ```
3. If the file should be protected in downstream repos, add it to
   `github-templates/CODEOWNERS`.

### Sync PR behavior

- Sync PRs use the `chore/sync-master-files` branch prefix.
- The `enforce-governance` job in `workflows/ci.yml` exempts sync PRs.
- Sync runs automatically on push to `main`.

## 4. CODEOWNERS as Governance

`github-templates/CODEOWNERS` defines which files are "locked" (owned by
`@victormgomes`). This file is enforced in three places:

1. **GitHub branch protection:** Required review from `@victormgomes`.
2. **CI `enforce-governance` job:** Blocks PRs that modify locked files (unless
   the PR is from a sync branch).
3. **CaptainHook pre-commit hook:** Prevents local commits that modify locked files.

All downstream repos receive this same CODEOWNERS file via sync. Locked files
**must** only be modified in this repository.

## 5. Adding New Reusable Workflows

1. Create the workflow in `.github/workflows/` with `on: workflow_call` trigger.
2. Follow the established naming pattern:
   - `php-code-quality.yml` — Composer validate, audit, PHPStan, PHP Insights
   - `php-style-checker.yml` — Laravel Pint check mode
   - `php-style-fixer.yml` — Laravel Pint fix mode + auto-commit
   - `laravel-package-tests.yml` — Matrix testing across PHP/Laravel/OS
   - `infra-lint.yml` — Markdownlint, JSON lint, Hadolint, actionlint
   - `semantic-release.yml` — Semantic versioning release
   - `vitepress-docs.yml` — VitePress build + GitHub Pages deploy
3. To integrate into downstream CI, add a corresponding job in `workflows/ci.yml`
   that calls it via repo-qualified path:
   ```yaml
   your-job:
       uses: victormgomes/laravel-libs-shared/.github/workflows/your-workflow.yml@main
   ```

## 6. Adding New Shared Configs

1. Place the config file in the appropriate source directory:
   - `configs/` — files that land at the downstream repo root
   - `community/` — files that land in `.github/`
   - `github-templates/` — files that land in `.github/`
   - `agents/` — files that land in `.agents/`
2. Add a source-to-destination mapping in `.github/sync.yml`.
3. If the file should be protected, add it to
   `github-templates/CODEOWNERS`.

## 7. The `agents/` → `.agents/` Convention

- **Sync source:** `agents/` (no dot prefix) — contains generic guidelines
  (commit conventions, code quality, Docker workflow) suitable for any Laravel
  package. These sync to `.agents/` in downstream repos.
- **Local guidelines:** `.agents/` (dot prefix) — this file you are reading now.
  Contains infrastructure-specific guidelines for `laravel-libs-shared` itself.
- Do **not** add package-specific guidelines to `agents/` — those belong in each
  downstream repo's own `.agents/AGENTS.md`.

## 8. Docker-Only Development

- Never run PHP, Composer, or npm commands directly on the host.
- Always use: `docker compose run --rm dev <command>`
- Pipe long outputs to `logs/` to avoid context flooding.
- Use `act` for local workflow testing via Docker.

## 9. Commit Conventions

- Follow Conventional Commits strictly: `feat`, `fix`, `docs`, `style`,
  `refactor`, `perf`, `test`, `build`, `ci`, `chore`, `revert`.
- Breaking changes use `!` suffix (e.g., `feat!:`, `refactor!:`).
- Most changes to this repo will use `ci:`, `chore:`, or `docs:` prefixes.
- All commit messages must be in English.

## 10. What NOT to Do

- Do not add `composer.json`, `src/`, or `tests/` to this repository — it is
  not a Laravel package.
- Do not modify files in `workflows/` without understanding they will be synced
  verbatim to all downstream repos.
- Do not add a downstream repo to sync without updating `.github/sync.yml`.
- Do not forget to update `github-templates/CODEOWNERS` when adding new
  protected files.
- Do not confuse this repository with `actions-workflows` — `laravel-libs-shared`
  is the canonical source.
- Do not create a `.agents/` directory in this repository.
