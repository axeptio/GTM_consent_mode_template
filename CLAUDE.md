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

This repo is **not an application**, and it is **deprecated**. It is the public source for the
"Axeptio Consent Mode v2" GTM custom template, superseded by the "Axeptio CMP" template at
[axeptio/axeptio-gtm-public-template](https://github.com/axeptio/axeptio-gtm-public-template).
One file is the product:

- **`template.tpl`** — the GTM custom template: `___INFO___`, `___TEMPLATE_PARAMETERS___`,
  `___SANDBOXED_JS_FOR_WEB_TEMPLATE___`, `___WEB_PERMISSIONS___` and `___TESTS___` blocks in
  Google's own format. It is UTF-8 **with a BOM** — read it as `utf-8-sig` or the first marker is
  corrupted. Its `___TERMS_OF_SERVICE___` header is Google's mandatory gallery boilerplate —
  **never edit it**.

There is deliberately **no `metadata.yaml`**. Deleting it (commit `e64c746`) is the sanctioned way
to remove a template from the
[Community Template Gallery](https://developers.google.com/tag-platform/tag-manager/templates/gallery),
and this template is intentionally delisted. Do not recreate the file except as part of a
deliberate resubmission decision.

Everything else is licensing (`LICENSE`, `CONTRIBUTING.md`), release automation
(`.github/workflows/`, `scripts/`, `release-please-config.json`) or agent tooling (`.beads/`).

## Build & Test

There is **no build, no compile, and no test runner** — nothing to install beyond PyYAML.
Validation is by inspection plus these checks:

```bash
python3 scripts/validate-gallery.py                                  # expect OK + 2 warnings
python3 -c "import json; json.load(open('release-please-config.json'))"
python3 -c "import json; json.load(open('.release-please-manifest.json'))"
node --check commitlint.config.mjs
```

`validate-gallery.py` runs in **delisted mode** here: the missing `metadata.yaml` and the missing
`categories` in `___INFO___` are warnings, while the LICENSE rules and the `___INFO___` structural
rules still fail the build. Restoring `metadata.yaml` re-arms the full contract automatically.
CI runs it on every PR **and** on pushes to `master`.

To exercise the template itself, import `template.tpl` into a GTM container and use the
**Tests** tab (the `___TESTS___` block — currently empty, `scenarios: []`).

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
- **Do not change `LICENSE`.** The gallery requires it to contain **only** Apache 2.0 and removes
  a template whose licence does not match — replacing it with Axeptio's proprietary terms is what
  caused SUP-1008 on the sibling repository. The constraint is kept here so the repo stays
  resubmittable.
- **No metadata-sync release step.** The sibling repo's `Release` workflow has a tail that
  regenerates `metadata.yaml` and pushes a GPG-signed commit. It does not apply here and must not
  be ported; this repo needs only `BOT_GITHUB_TOKEN`, not `BOT_GPG_PRIVATE_KEY`.
- **This repository is public**, so every GitHub Action must be pinned to a full commit SHA, and
  `axeptio/tech-scripts` reusable workflows are unusable (internal repo, public caller).
- `gh` is the canonical interface for GitHub work.
