# GTM_consent_mode_template aka Axeptio Consent Mode v2

⚠️ **This Google Tag Manager template is deprecated, and `template.tpl` has been removed from this repository.**

This repository is no longer maintained. New implementations should use the **"Axeptio CMP"** tag template, available directly in the Google Tag Manager template gallery or on GitHub at [https://github.com/axeptio/axeptio-gtm-public-template](https://github.com/axeptio/axeptio-gtm-public-template).

If you are currently using this template, please migrate your configuration to the **"Axeptio CMP"** template to benefit from the latest features and updates.

## Where the template went

`template.tpl` was deleted deliberately, to force this template out of the
[Community Template Gallery](https://developers.google.com/tag-platform/tag-manager/templates/gallery).

The last version that still contained it is tagged
[`v1.0.1`](https://github.com/axeptio/GTM_consent_mode_template/tree/v1.0.1), and the file's full
history is preserved in this repository. Nothing is lost — it is simply no longer served.

## Why it had to be deleted

`metadata.yaml` was removed in March 2026, which Google
[documents](https://developers.google.com/tag-platform/tag-manager/templates/gallery) as a way to
remove a template from the gallery. **It did not work.** The gallery detected the missing file,
recorded `The metadata.yaml file was not found` in its sync log, failed the sync — and then kept
the template listed and installable at the last commit it had successfully read
(`8b2237f`, February 2025).

Deleting `metadata.yaml` stops the gallery from *updating* a template; it does not *delist* it.
Removing `template.tpl` breaks the required repository structure, which the same documentation says
causes removal — and that did not delist it either.

What works is the licence. The gallery requires a listed template's `LICENSE` to contain **only**
the Apache 2.0 text, and removes any template whose licence does not match; we have seen that take
effect within about 24 hours. `LICENSE` therefore now carries a deprecation notice above the
licence text.

**This does not affect your rights to this code.** The change is purely additive — the Apache
License 2.0 is unchanged and still applies in full, to this repository and to every copy of it.

## Documentation

The documentation link this template published to the gallery
(`support.axeptio.eu/hc/en-gb/articles/9263604989457`) is dead. Current Axeptio documentation is at
[support.axeptio.eu](https://support.axeptio.eu/en/) — see the
"Consent Modes : guides d'implémentation" collection.
