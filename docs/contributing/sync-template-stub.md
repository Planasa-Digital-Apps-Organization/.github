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

`claude-sanbox` is **private**, so the caller's default `GITHUB_TOKEN` cannot
read its tarball. Provide a **read-only** token on `claude-sanbox` — a
fine-grained PAT or GitHub App token scoped to **only `claude-sanbox`** with
**Contents: Read** (nothing else) — exposed as the secret `TEMPLATE_TOKEN`
(uppercase, no hyphen — GitHub secret names allow only `[A-Za-z0-9_]`; matches
the `<CAPABILITY>_TOKEN` convention of ADR 0004). It is forwarded via
`secrets: inherit` and used **only** to download the template tarball.

The sync **PR is opened with the caller's auto-minted `GITHUB_TOKEN`**, not
`TEMPLATE_TOKEN`, so `TEMPLATE_TOKEN` needs no write access anywhere. For this
to work the caller repo (or org) must have **Settings → Actions → General →
"Allow GitHub Actions to create and approve pull requests"** enabled.

If `claude-sanbox` is later made public, the workflow falls back to the caller's
`GITHUB_TOKEN` for the tarball too and no extra secret is needed.

## What a run produces

A run opens a `feature/ID-1237-sync-template-<version>` PR labelled
`area:ai-behavior` and `automation`. (The branch sits in the `feature/<JIRA>`
namespace because the enterprise naming ruleset rejects `chore/*` branches at
creation; if `chore/**` is later excluded from that ruleset, this can revert to
a `chore/sync-template-<version>` name.) Review before merging — `.claude/**`
and `AGENTS.md` are
CODEOWNERS-gated. If nothing changed, no PR is opened.
