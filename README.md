# Laravel Libs Shared

Central repository for Laravel packages.

## Architecture

This repository operates under two main mechanisms:
1. **Reusable Workflows**: Cloud workflows located in `.github/workflows/` that are executed remotely by child repositories via the `uses:` directive.
2. **File Synchronization**: Physical synchronization of community files, tooling configurations, and templates to child repositories using `repo-file-sync-action`.

## Local Development & Testing

Use Docker and `act` (Nektos/act) for local workflow execution.

### Validate Configurations (Lint)
Check syntax for markdown, JSON, and YAML files:
```bash
docker compose run --rm dev sudo act -W .github/workflows/validate-configs.yml -s GITHUB_TOKEN="ghp_your_token_here" > logs/validate-configs.txt 2>&1
```

### Test File Synchronization (Dry Run)
Simulate the file synchronization process (dry-run mode is automatically enforced when executing via `act`):
```bash
docker compose run --rm dev sudo act -j sync -s SYNC_TOKEN="ghp_your_token_here" > logs/sync.txt 2>&1
```
