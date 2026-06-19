# Consuming the template sync workflow

Derived repos stay in sync with the [`claude-sanbox`](https://github.com/Planasa-Digital-Apps-Organization/claude-sanbox)
template by calling the reusable [`sync-template.yml`](../../.github/workflows/sync-template.yml)
workflow that lives in this org repo. Each repo pins the template version it
tracks and opts in with a thin caller stub. This page is the copy-paste source.

## Two files per derived repo

### 1. `.claude/template-version.yml`

Pins the template release this repo tracks and lists the sync-safe paths the
workflow copies (no delete — repo-local additions under these paths survive).

```yaml
# Template release this repo tracks. Bump to adopt a newer template version;
# the sync workflow then opens a PR with the diff.
version: v0.1.0

# Sync-safe paths: the template owns these; the sync workflow overwrites them
# from the template tarball. Anything else in the repo is per-repo and untouched.
sync-paths:
  - .claude/skills
  - .claude/rules
  - .claude/agents
  - .claude/hooks
  - AGENTS.md
```

### 2. `.github/workflows/sync-claude-template.yml`

The caller stub. Triggers when the pinned version changes, and on demand.

```yaml
name: sync-claude-template

on:
  # Re-sync whenever the pinned template version is bumped.
  push:
    branches:
      - master
    paths:
      - .claude/template-version.yml
  # And allow a manual run from the Actions tab.
  workflow_dispatch:

jobs:
  sync:
    uses: Planasa-Digital-Apps-Organization/.github/.github/workflows/sync-template.yml@master
    permissions:
      contents: write
      pull-requests: write
    # `inherit` forwards org/repo secrets (incl. the template read token) to the
    # reusable workflow. The template repo is private, so a token with read
    # access to claude-sanbox is required — see the note below.
    secrets: inherit
```

## Pinning the reusable workflow ref

The stub above uses `@master`. For reproducible bootstraps you can instead pin a
tag of this org repo (e.g. `@v0.1.0`) so a derived repo's sync behaviour only
changes when you deliberately bump the ref.

## Required token

The caller's default `GITHUB_TOKEN` is **not** enough, for two reasons:

1. `claude-sanbox` is **private**, and a repo's `GITHUB_TOKEN` cannot read
   another private repo's tarball.
2. The enterprise **disables** "Allow GitHub Actions to create and approve pull
   requests" (it is forced off org-wide and cannot be toggled per-repo). A PR
   opened with `GITHUB_TOKEN` is therefore rejected. A PAT bypasses this — the
   same reason `RELEASE_PLEASE_TOKEN` and `LABEL_SYNC_TOKEN` exist.

So `TEMPLATE_TOKEN` must be a **PAT (or GitHub App token)** that does *both* the
tarball read and the PR creation. Least-privilege scope:

| Repo | Permission |
| --- | --- |
| `claude-sanbox` (template) | Contents: **Read** |
| each consumer repo (`fichajes-app`, …) | Contents: **Read & write**, Pull requests: **Read & write** |

Expose it as the org secret `TEMPLATE_TOKEN` (uppercase, no hyphen — GitHub
secret names allow only `[A-Za-z0-9_]`; matches the `<CAPABILITY>_TOKEN`
convention of ADR 0004) and make it **visible to the consumer repos** (not to
`claude-sanbox`, which never runs the sync). It is forwarded via
`secrets: inherit`.

> Strict-minimum alternative: two tokens — one read-only on `claude-sanbox` for
> the tarball, one write-only on consumer repos for the PR — at the cost of a
> second secret and a workflow input. The single-PAT setup above mirrors the
> existing org-PAT pattern and is the recommended default.

## What a run produces

A run opens a `feature/ID-1237-sync-template-<version>` PR labelled
`area:ai-behavior` and `automation`. (The branch sits in the `feature/<JIRA>`
namespace because the enterprise naming ruleset rejects `chore/*` branches at
creation; if `chore/**` is later excluded from that ruleset, this can revert to
a `chore/sync-template-<version>` name.) Review before merging — `.claude/**`
and `AGENTS.md` are
CODEOWNERS-gated. If nothing changed, no PR is opened.
