# Supabase Reusable GitHub Actions

This repository hosts shared, reusable GitHub Actions workflows used across Supabase repositories.

## Available Workflows

### 1. Block WIP/Draft Merges

Prevents merging of pull requests that aren't ready.

**Workflow:** `.github/workflows/block-merge.yml`

**Usage:**
```yaml
name: Block WIP Merges
on:
  pull_request:
    types: [opened, synchronize, reopened, labeled, unlabeled]

jobs:
  block-merge:
    uses: supabase/actions/.github/workflows/block-merge.yml@main
    with:
      block_labels: 'do-not-merge,needs-review'  # Optional
      block_keywords: 'wip,do not merge,draft'    # Optional
```

**What it blocks:**
- Draft PRs
- PRs with specified labels (default: `do-not-merge`)
- PRs with keywords in title (default: `wip`, `do not merge`)

---

### 2. Stale Issue Management

Automatically manages stale issues and pull requests.

**Workflow:** `.github/workflows/stale.yml`

**Usage:**
```yaml
name: Stale Issues and PRs
on:
  schedule:
    - cron: '0 0 * * 0'  # Weekly on Sundays
  workflow_dispatch:

permissions:
  issues: write
  pull-requests: write

jobs:
  stale:
    uses: supabase/actions/.github/workflows/stale.yml@main
    with:
      days_before_issue_stale: 180     # Optional, default: 180
      days_before_issue_close: 30      # Optional, default: 30
      days_before_pr_stale: 90         # Optional, default: 90
      days_before_pr_close: 14         # Optional, default: 14
      exempt_issue_labels: 'priority,security,planned'  # Optional
      exempt_pr_labels: 'priority,security'            # Optional
      operations_per_run: 100          # Optional, default: 100
```

**Features:**
- Different timelines for issues vs PRs
- Exempts labeled, assigned, or milestone items
- Removes stale labels when updated
- Rate-limited for large repos

---

### 3. Auto-Label Issues and PRs

Automatically labels issues and PRs based on content.

**Workflow:** `.github/workflows/label-issues.yml`

**Usage:**
```yaml
name: Label Issues and PRs
on:
  issues:
    types: [opened, transferred]
  pull_request:
    types: [opened]

permissions:
  issues: write
  pull-requests: write

jobs:
  label:
    uses: supabase/actions/.github/workflows/label-issues.yml@main
    with:
      label_mappings: |
        {
          "auth": "auth",
          "storage": "storage",
          "realtime": "realtime",
          "functions": "functions",
          "postgrest": "database"
        }
```

**How it works:**
- **For PRs:** Extracts scope from conventional commit titles (e.g., `fix(auth):` → `auth` label)
- **For Issues:** Parses issue template content for module checkboxes or mentions
- **Migration Detection:** Adds `migration` label if `[migration]` is in title

---

### 4. Slack Notifications

Sends rich notifications to Slack.

**Workflow:** `.github/workflows/slack-notify.yml`

**Usage:**
```yaml
name: Notify on Failure
on:
  workflow_run:
    workflows: ["CI"]
    types: [completed]
    branches: [main]

jobs:
  notify:
    if: ${{ github.event.workflow_run.conclusion == 'failure' }}
    uses: supabase/actions/.github/workflows/slack-notify.yml@main
    with:
      title: "CI Failed on Main"
      status: failure
      version: "v1.2.3"  # Optional
      mention_group: "@group-sdk"  # Optional
    secrets:
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

**Supported statuses:**
- `success` - ✅ Green color
- `failure` - ❌ Red color
- `info` - ℹ️ Blue color

**Features:**
- Rich Slack message with repository, workflow, and commit info
- Action buttons linking to workflow run and commit
- Optional version display
- Optional group mentions

---

## Best Practices

### Versioning

Pin workflows to specific versions for stability:

```yaml
# Pin to major version (recommended)
uses: supabase/actions/.github/workflows/block-merge.yml@v1

# Pin to specific commit (most stable)
uses: supabase/actions/.github/workflows/block-merge.yml@a1b2c3d

# Use main branch (latest, less stable)
uses: supabase/actions/.github/workflows/block-merge.yml@main
```

### Permissions

Reusable workflows inherit permissions from the caller. Always specify minimum required permissions:

```yaml
jobs:
  my-job:
    permissions:
      issues: write
      pull-requests: write
    uses: supabase/actions/.github/workflows/stale.yml@main
```

### Secrets

Pass secrets explicitly:

```yaml
jobs:
  notify:
    uses: supabase/actions/.github/workflows/slack-notify.yml@main
    secrets:
      SLACK_WEBHOOK_URL: ${{ secrets.SLACK_WEBHOOK_URL }}
```

---

## Contributing

### Adding New Workflows

1. Create workflow in `.github/workflows/`
2. Use `workflow_call` trigger
3. Document inputs, secrets, and permissions
4. Add usage example to this README
5. Test in a consumer repository

### Testing Changes

Before merging, test changes by:

1. Push to a branch in this repo
2. Update consumer repo to use the branch:
   ```yaml
   uses: supabase/actions/.github/workflows/my-workflow.yml@my-branch
   ```
3. Verify behavior
4. Merge when confirmed working

---

## Support

For issues or feature requests, please [open an issue](https://github.com/supabase/actions/issues/new).

For Supabase-specific support, visit [supabase.com/support](https://supabase.com/support).
