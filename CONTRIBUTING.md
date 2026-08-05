# How to Contribute

> **This template is deprecated and no longer maintained.** New implementations
> should use the **"Axeptio CMP"** template
> ([axeptio/axeptio-gtm-public-template](https://github.com/axeptio/axeptio-gtm-public-template)),
> available in the Google Tag Manager template gallery. Contributions here are
> limited to security fixes and documentation; new features belong in the
> replacement template.

For the changes that do still land, there are a few small guidelines to follow.

## Contributions and licensing

This template is distributed under the [Apache License 2.0](./LICENSE). By
submitting a contribution you agree that it is provided under, and may be
redistributed as part of this project under, that licence.

The `LICENSE` file must contain **only** Apache 2.0. The
[Community Template Gallery](https://developers.google.com/tag-platform/tag-manager/templates/gallery)
removes a template whose licence does not match — that is what happened in SUP-1008 — and while
this template is currently delisted (see below), the constraint is kept so the repository stays
resubmittable. **Do not change `LICENSE`.**

## Code reviews

All submissions, including submissions by project members, require review. We
use GitHub pull requests for this purpose. Consult
[GitHub Help](https://help.github.com/articles/about-pull-requests/) for more
information on using pull requests.

## Gallery status

`metadata.yaml` was deleted in `e64c746` (March 2026) to remove this template from the Community
Template Gallery. **That did not delist it.** The gallery detected the missing file, logged
`The metadata.yaml file was not found`, failed the sync, and kept serving the template at the last
commit it had read successfully (`8b2237f`, February 2025) — still installable, with a dead
documentation link.

`template.tpl` was therefore deleted too, breaking the repository structure the gallery requires.
The template file remains in the history and at tag `v1.0.1`.

With no template left to check, `scripts/validate-gallery.py` and its `Validate gallery contract`
workflow were removed in the same change. If the template is ever restored, recover both from
tag `v1.0.1` — the script's `LISTED` flag already handles the listed and delisted cases, and
restoring `metadata.yaml` re-arms the full contract automatically.

**`LICENSE` must still contain only Apache 2.0.** Nothing enforces that any more, so it now rests
on review.

## Commit & pull request conventions

This project uses [Conventional Commits](https://www.conventionalcommits.org/)
to drive automated [Semantic Versioning](https://semver.org/) and changelog
generation via [release-please](https://github.com/googleapis/release-please).

A commit / PR title must follow:

```
<type>(<optional scope>): <description>
```

**Allowed types**

| Type       | Effect on version | Use for                                    |
| ---------- | ----------------- | ------------------------------------------ |
| `feat`     | minor bump        | a new feature                              |
| `fix`      | patch bump        | a bug fix                                  |
| `docs`     | none              | documentation only                         |
| `refactor` | none              | code change that isn't a fix or feature    |
| `perf`     | none              | performance improvement                    |
| `test`     | none              | tests                                      |
| `ci`       | none              | CI / GitHub Actions changes                |
| `build`    | none              | build system or dependencies               |
| `chore`    | none              | maintenance / tooling                      |
| `revert`   | none              | reverting a previous commit                |

A breaking change is signalled by a `!` after the type (e.g. `feat!: ...`) or a
`BREAKING CHANGE:` footer, and triggers a major bump.

**Suggested scopes:** `template`, `docs`, `ci`.

Examples:

```
fix(template): correct the cookie expiry check
docs: clarify the migration steps to the Axeptio CMP template
ci: pin the release-please action to a full SHA
```

**Important notes**

- Pull requests are merged with a **merge commit** (squash and rebase are
  disabled), so **every individual commit** lands on `master` and is parsed by
  release-please. The `Lint commits` CI check lints them all — a stray
  `wip: fixup` in your branch will fail the check, so tidy the history before
  requesting review. The PR title is linted too, so it stays a valid
  Conventional Commit.
- `master` requires **signed commits**. Configure commit signing before you push.
- Releases, `CHANGELOG.md`, `VERSION`, git tags and GitHub Releases are **all
  generated automatically**. Do not edit versions or the changelog by hand. See
  [docs/release-automation.md](./docs/release-automation.md).

## Community Guidelines

Please be respectful and constructive in issues and pull requests. For questions
about the template or Axeptio, see the [Axeptio documentation](https://www.axept.io/)
or contact [support@axeptio.eu](mailto:support@axeptio.eu).
