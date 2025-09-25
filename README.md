# Sync Deploy Branches Action

A GitHub composite action that discovers all `deploy/*` branches and syncs them with the main branch by force-pushing main to each deploy branch.

## Features

- 🔍 **Auto-discovery**: Automatically finds all `deploy/*` branches in the repository
- 🚫 **Exclusion support**: Exclude specific deploy branches from syncing
- 🔄 **Force sync**: Ensures deploy branches are exactly like main
- 📊 **Outputs**: Provides JSON arrays of discovered and synced branches
- 🛡️ **Safe**: Includes proper git configuration and workspace permissions

## Usage

### Basic Usage

```yaml
- name: Sync Deploy Branches
  uses: pioncorp/sync-deploy-branches@main
  with:
    token: ${{ secrets.GITHUB_TOKEN }}
```

### With Custom Exclusions

```yaml
- name: Sync Deploy Branches
  uses: pioncorp/sync-deploy-branches@main
  with:
    exclude-branches: "deploy/old,deploy/legacy,deploy/experimental"
    token: ${{ secrets.GITHUB_TOKEN }}
```

### Complete Workflow Example

```yaml
name: Sync Deploy Branches
on:
  schedule:
    - cron: "0 0 * * 0" # Weekly on Sunday
  workflow_dispatch:

permissions:
  contents: write

jobs:
  sync:
    runs-on: ubuntu-latest
    steps:
      - name: Sync Deploy Branches
        id: sync
        uses: pioncorp/sync-deploy-branches@main
        with:
          token: ${{ secrets.GITHUB_TOKEN }}
          exclude-branches: "deploy/example,deploy/dummy"

      - name: Display Results
        run: |
          echo "Discovered branches: ${{ steps.sync.outputs.branches-discovered }}"
          echo "Synced branches: ${{ steps.sync.outputs.branches-synced }}"
```

## Inputs

| Input              | Description                                               | Required | Default                       |
| ------------------ | --------------------------------------------------------- | -------- | ----------------------------- |
| `exclude-branches` | Comma-separated list of deploy branches to exclude        | No       | `deploy/example,deploy/dummy` |
| `token`            | GitHub token with write permissions for repository access | **Yes**  | None                          |

## Outputs

| Output                | Description                                          |
| --------------------- | ---------------------------------------------------- |
| `branches-discovered` | JSON array of all deploy branches found              |
| `branches-synced`     | JSON array of branches that were successfully synced |

## How It Works

1. **Discovery**: Fetches all remote branches and filters for `deploy/*` pattern
2. **Filtering**: Excludes branches specified in the `exclude-branches` input
3. **Sync**: For each remaining branch, force-pushes `main` to that branch
4. **Reporting**: Outputs the results as JSON arrays for further processing

## Requirements

- **Token with write permissions**: The action requires a GitHub token with `contents: write` permission to perform git push operations
- **Workflow permissions**: Ensure your workflow has the necessary permissions:
  ```yaml
  permissions:
    contents: write
  ```
- Works with both GitHub-hosted and self-hosted runners
- Requires `jq` (automatically installed if not present)

## Notes

- This action performs **force pushes**, which will overwrite the history of deploy branches
- Use with caution in repositories with important deploy branch history
- The action is designed for environments where deploy branches should always mirror main
