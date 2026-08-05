# Project Instructions for AI Agents

This file provides instructions and context for AI coding agents working on this project.
The project sections below are kept in sync with `AGENTS.md`; that file additionally carries
Codex-specific setup, which is why `bd doctor`'s divergence check is opted out of here.

<!-- bd-doctor-divergence: ok -->

<!-- BEGIN BEADS INTEGRATION v:1 profile:minimal hash:970c3bf2 -->
## Beads Issue Tracker

This project uses **bd (beads)** for issue tracking. Run `bd prime` to see full workflow context and commands.

### Quick Reference

```bash
bd ready              # Find available work
bd show <id>          # View issue details
bd update <id> --claim  # Claim work
bd close <id>         # Complete work
```

### Rules

- Use `bd` for ALL task tracking — do NOT use TodoWrite, TaskCreate, or markdown TODO lists
- Run `bd prime` for detailed command reference and session close protocol
- Use `bd remember` for persistent knowledge — do NOT use MEMORY.md files

**Architecture in one line:** issues live in a local Dolt DB; sync uses `refs/dolt/data` on your git remote; `.beads/issues.jsonl` is a passive export. See https://github.com/gastownhall/beads/blob/main/docs/SYNC_CONCEPTS.md for details and anti-patterns.

## Agent Context Profiles

The managed Beads block is task-tracking guidance, not permission to override repository, user, or orchestrator instructions.

- **Conservative (default)**: Use `bd` for task tracking. Do not run git commits, git pushes, or Dolt remote sync unless explicitly asked. At handoff, report changed files, validation, and suggested next commands.
- **Minimal**: Keep tool instruction files as pointers to `bd prime`; use the same conservative git policy unless active instructions say otherwise.
- **Team-maintainer**: Only when the repository explicitly opts in, agents may close beads, run quality gates, commit, and push as part of session close. A current "do not commit" or "do not push" instruction still wins.

## Session Completion

This protocol applies when ending a Beads implementation workflow. It is subordinate to explicit user, repository, and orchestrator instructions.

1. **File issues for remaining work** - Create beads for anything that needs follow-up
2. **Run quality gates** (if code changed) - Tests, linters, builds
3. **Update issue status** - Close finished work, update in-progress items
4. **Handle git/sync by active profile**:
   ```bash
   # Conservative/minimal/default: report status and proposed commands; wait for approval.
   git status

   # Team-maintainer opt-in only, unless current instructions forbid it:
   git pull --rebase
   bd dolt push
   git push
   git status
   ```
5. **Hand off** - Summarize changes, validation, issue status, and any blocked sync/commit/push step

**Critical rules:**
- Explicit user or orchestrator instructions override this Beads block.
- Do not commit or push without clear authority from the active profile or the current user request.
- If a required sync or push is blocked, stop and report the exact command and error.
<!-- END BEADS INTEGRATION -->


## Architecture Overview

This repo is **not an application**, it is **deprecated**, and **it no longer contains a
template**. It was the public source for the "Axeptio Consent Mode v2" GTM custom template,
superseded by the "Axeptio CMP" template at
[axeptio/axeptio-gtm-public-template](https://github.com/axeptio/axeptio-gtm-public-template).

`template.tpl`, `metadata.yaml`, `scripts/validate-gallery.py` and the `Validate gallery contract`
workflow have all been deleted. All of them are recoverable from tag `v1.0.1`. What remains is
licensing (`LICENSE`, `CONTRIBUTING.md`), release automation (`.github/workflows/`,
`release-please-config.json`) and agent tooling (`.beads/`).

**Do not recreate any of the deleted files** without a deliberate decision to resubmit the
template to the gallery — the deletions are what forced it out.

Why two deletions were needed: removing `metadata.yaml` (commit `e64c746`, March 2026) is
documented by Google as the way to delist a template, and it **did not work**. Verified 2026-08-05:
the gallery reported `Current Status: Available` at `8b2237f7` (February 2025), with
`The metadata.yaml file was not found` at the top of its sync log. It detects the missing file,
fails the sync, and keeps serving the last commit it read. Deleting `template.tpl` breaks the
required repository structure, which is the remaining documented trigger. See
[docs/release-automation.md](docs/release-automation.md).

## Build & Test

There is **no build, no compile, and no test runner**, and since the gallery validator was removed,
nothing to install either. Validation is by inspection plus:

```bash
python3 -c "import json; json.load(open('release-please-config.json'))"
python3 -c "import json; json.load(open('.release-please-manifest.json'))"
node --check commitlint.config.mjs
```

CI is down to `Lint commits` (every PR) and `Release` (pushes to `master`).

## Conventions & Patterns

- **Conventional Commits are mandatory.** PRs land as **merge commits** (squash and rebase are
  disabled), so *every* commit in the branch reaches `master` and is what release-please parses —
  tidy the history before merging. CI (`Lint commits`) checks every commit and the PR title.
  Types/scopes live in `commitlint.config.mjs`.
- **`master` requires signed commits**, and is the only branch: default and release branch both.
- **Never hand-edit `VERSION`, `CHANGELOG.md` or `.release-please-manifest.json`** — all three are
  generated. See [docs/release-automation.md](docs/release-automation.md). The versioning baseline
  is the `v1.0.0` tag at `4ea6d9d`; the twelve commits below it predate this pipeline and are not
  conventional.
- **`LICENSE` carries a deliberate deprecation notice above the Apache 2.0 text.** The gallery
  removes a template whose licence is not Apache-2.0-only — that is what caused SUP-1008 on the
  sibling repository, within ~24h — and after the `metadata.yaml` and `template.tpl` deletions
  both failed to delist, that notice is what forces removal. The Apache 2.0 text below it is
  **unchanged and still applies in full**; the edit is purely additive. Do not remove the notice
  except as part of a deliberate resubmission, and never touch the licence text itself.
- **No metadata-sync release step.** The sibling repo's `Release` workflow has a tail that
  regenerates `metadata.yaml` and pushes a GPG-signed commit. It does not apply here and must not
  be ported; this repo needs only `BOT_GITHUB_TOKEN`, not `BOT_GPG_PRIVATE_KEY`.
- **This repository is public**, so every GitHub Action must be pinned to a full commit SHA, and
  `axeptio/tech-scripts` reusable workflows are unusable (internal repo, public caller).
- `gh` is the canonical interface for GitHub work.
