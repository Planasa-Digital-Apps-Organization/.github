# Planasa Digital Apps — Org defaults

Special `.github` repository for the
[`Planasa-Digital-Apps-Organization`](https://github.com/Planasa-Digital-Apps-Organization)
GitHub org. Holds org-wide defaults that are *not* duplicated per repo:

- **Reusable workflows** under `.github/workflows/` — invoked from every
  Planasa repo via `uses: Planasa-Digital-Apps-Organization/.github/.github/workflows/<name>.yml@<ref>`.
- **Label catalog** in `.github/labels.yml` — single source of truth.
  Propagated to every non-archived org repo by `sync-labels.yml`.

## What lives here

| Path | Purpose |
| --- | --- |
| `.github/labels.yml` | Declarative catalog of org-wide labels (name, color, description). |
| `.github/workflows/sync-labels.yml` | Reads `labels.yml`, syncs to every non-archived org repo. Triggers: push to `main` on `labels.yml`, scheduled weekly, manual `workflow_dispatch`. |
| `.github/workflows/sensitive-paths-check.yml` | Reusable workflow (`workflow_call`). Applies `area:ai-behavior` label and posts a checklist comment when a PR touches paths that shape AI behavior or PR safeguards. Invoked from each Planasa repo via a thin caller stub. |

## How a downstream repo opts in

Each Planasa repo includes a thin caller workflow that delegates to the
reusable here. Example: `.github/workflows/sensitive-paths-check.yml` in
the downstream repo:

```yaml
name: Flag sensitive paths
on:
  pull_request:
    paths:
      - '.claude/**'
      - 'CLAUDE.md'
      - 'AGENTS.md'
      - '.github/**'
      - '.gitignore'

permissions:
  contents: read
  pull-requests: write

jobs:
  flag:
    uses: Planasa-Digital-Apps-Organization/.github/.github/workflows/sensitive-paths-check.yml@master
```

## Required org secret

`sync-labels.yml` needs `LABEL_SYNC_TOKEN`: a fine-grained PAT with
`metadata: read` + `issues: write` on **all repositories** of the org.
Configure it once at `Settings → Secrets and variables → Actions` of the
**organization** (not this repo).

## Versioning

Bootstrap: callers reference `@master` to track latest. Once the workflow
contract stabilises we'll cut `v1` tag and switch callers to `@v1` so
breaking changes require explicit opt-in.

## Bootstrap exception

The initial scaffold of this repo was committed directly to `master`
(no PR, no branch protection). The org-level ruleset was temporarily
switched to `Evaluate` on `master` to unblock the first push; standard
enforcement on the other branch patterns (`feature/*`, `hotfix/*`,
`release/*`, `develop`) remained active. Subsequent changes follow the
standard flow (feature branch → PR → CODEOWNERS review).
