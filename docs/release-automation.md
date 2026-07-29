# Release Automation

Releases are driven by [Conventional Commits](https://www.conventionalcommits.org/) and
[release-please](https://github.com/googleapis/release-please). Every merge to `master`
maintains a release PR; merging that PR bumps `VERSION`, updates `CHANGELOG.md`, tags the
commit and publishes a GitHub Release.

This repository is **deprecated** — see [README.md](../README.md). The automation exists so
that the changes that do still land (a security fix, a documentation correction, a final
deprecation notice) are versioned and auditable rather than pushed straight to `master`.

## Branch flow

```
feature branch ──PR──> master ──> release PR ──> tag + GitHub Release
                       (default)
```

`master` is both the default branch and the release branch. There is no `develop`.

Pull requests are merged with a **merge commit** (squash and rebase merges are disabled on this
repository), so every commit in the branch lands on `master` — and every one of them is parsed by
release-please to work out the next version. Merge commits themselves are ignored. Tidy the branch
history before merging; `Lint commits` will reject a non-conventional commit anywhere in it.

## Versioning baseline

The repository had no tags or releases until `v1.0.0`, cut at commit `4ea6d9d` — the state of the
template as it was published to the gallery before being delisted. release-please resolves the last
release from the newest `v*` tag, so it never looks at the twelve legacy commits below that point
(none of which are conventional; they were written through the GitHub web UI).

## Workflows

- **`.github/workflows/commitlint.yml`** (`Lint commits`) — runs on every PR with two jobs:

  | Job | What it checks |
  | --- | --- |
  | `Validate commit messages` | every commit in the PR, against `commitlint.config.mjs` — these are the ones release-please reads |
  | `Validate PR title` | the PR title is a valid Conventional Commit — hygiene today, and the safety net if squash-merging is ever re-enabled |

  This is what makes automated versioning possible: `fix:` → patch, `feat:` → minor,
  `feat!:` / `BREAKING CHANGE:` → major.

- **`.github/workflows/release.yml`** (`Release`) — fires on push to `master`. release-please
  scans the commits since the last release, works out the next version, and opens (or updates)
  a release PR that bumps `VERSION`, updates `CHANGELOG.md` and bumps
  `.release-please-manifest.json`. Merging that PR tags the commit and publishes a GitHub
  Release. That is the whole workflow — a single step.

- **`.github/workflows/validate-gallery.yml`** (`Validate gallery contract`) — runs
  `scripts/validate-gallery.py` on every PR and on pushes to `master`. See below.

## Gallery status: delisted

The template was removed from the
[Community Template Gallery](https://developers.google.com/tag-platform/tag-manager/templates/gallery)
by deleting `metadata.yaml` (commit `e64c746`), which is the sanctioned way to delist. That file
was the gallery's published version history — a `versions:` list of commit SHAs — so there is
nothing left to sync on release.

This is the one place this repository deliberately differs from its sibling
`axeptio/axeptio-gtm-public-template`, whose `Release` workflow has a tail that regenerates
`metadata.yaml` and pushes a GPG-signed sync commit. **Do not port that step here.** As a
consequence this repository needs only `BOT_GITHUB_TOKEN`, not `BOT_GPG_PRIVATE_KEY`.

`scripts/validate-gallery.py` still runs, in "delisted mode": the absence of `metadata.yaml` and
the missing `categories` in `___INFO___` are reported as warnings, while the LICENSE rules and the
`___INFO___` structural rules still fail the build. If `metadata.yaml` is ever restored the script
re-arms the full contract automatically, with no edit needed.

## Authentication

The workflow authenticates as **`axeptio-bot`**, not the default `GITHUB_TOKEN`. The
organisation forbids `GITHUB_TOKEN` from creating or approving pull requests, so release-please
cannot open its release PR without a real bot account.

| Secret | Used for | Source |
| ------------------ | ---------------------------- | --------- |
| `BOT_GITHUB_TOKEN` | release PR, publishing the release | Org-level |

`master` enforces signed commits. release-please creates its release-PR commits through the
GitHub API, which signs them automatically, so no GPG key is needed in this repository.

## Why not the canonical Axeptio release automation?

Axeptio's canonical release automation (ENG-11756) is a pair of thin caller workflows —
`create-release-pr.yml` and `auto-release.yml` — that call reusable workflows hosted in
`axeptio/tech-scripts`.

**They cannot be used here.** This repository is **public** and `axeptio/tech-scripts` is
**internal**. GitHub only allows a public caller repository to use reusable workflows from
**public** repositories, so both callers fail at access time with `workflow was not found`
before running a single job. See
[Access to reusable workflows](https://docs.github.com/en/actions/reference/workflows-and-actions/reusable-workflows).

`release-please-action` is a public action, so it has no such restriction. The sibling public
repositories `axeptio/axeptio-gtm-public-template` and `axeptio/axeptio-sgtm-public-template` use
the same approach, and this repo's setup is deliberately kept aligned with them.
