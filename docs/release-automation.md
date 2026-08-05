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

The `Validate gallery contract` workflow ran here too, until the template was removed — see below.

## Gallery status: delisted the hard way

`metadata.yaml` was deleted in `e64c746` (March 2026) on the understanding, taken from Google's own
[documentation](https://developers.google.com/tag-platform/tag-manager/templates/gallery), that
this removes a template from the gallery.

**It does not.** Verified on 2026-08-05: the gallery's status page reported
`Current Status: Available` with `Current Version: 8b2237f7` (February 2025), and a sync log whose
newest entry read `The metadata.yaml file was not found`, followed by
`master — GitHub returned an error`. Google detects the missing file, fails the sync, and then goes
on serving the last commit it read successfully. The template stayed installable, advertising a
documentation URL that 404s — which is what issue #1 reported, and what nobody acted on for two and
a half years.

So deleting `metadata.yaml` freezes a template; it does not delist it. `template.tpl` was deleted
as well, breaking the repository structure the gallery requires, which the same documentation says
causes removal. Along with it went `scripts/validate-gallery.py` and
`.github/workflows/validate-gallery.yml` — with no template in the repository they had nothing left
to validate. Both are recoverable from tag `v1.0.1`.

**That did not delist it either.** The status page was unchanged after the deletion landed.

What finally works is the licence. The gallery requires a listed template's `LICENSE` to contain
*only* the Apache 2.0 text, and removes any template whose licence does not match — observed
directly in SUP-1008, where the sibling repository was delisted within roughly 24 hours of its
licence being replaced. A deprecation notice was therefore prepended to `LICENSE`. The change is
**purely additive**: the Apache 2.0 text is untouched and still applies in full, so no rights are
withdrawn from anyone, while the file no longer satisfies the gallery's "only Apache 2.0" rule.

The ranking that emerges, for anyone who needs to do this again:

| Lever | Effect |
| --- | --- |
| delete `metadata.yaml` | freezes the template at the last good commit; **does not delist** |
| delete `template.tpl` | breaks the required structure; **did not delist** here either |
| make `LICENSE` not-only-Apache-2.0 | **delists**, ~24h (SUP-1008) |

A side effect worth knowing: because the gallery froze at `8b2237f`, **no release cut here ever
reached gallery users.** `v1.0.0`, `v1.0.1` and `v2.0.0` exist in GitHub and in the changelog, and
were never served.

This is also why the sibling `axeptio/axeptio-gtm-public-template` has a `Release` tail that
regenerates `metadata.yaml` and pushes a GPG-signed sync commit, and this repository does not.
**Do not port that step here.** This repository needs only `BOT_GITHUB_TOKEN`, never
`BOT_GPG_PRIVATE_KEY`.

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
