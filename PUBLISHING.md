# Publishing a Plugin to dokuwiki.org

A repeatable recipe for publishing a DokuWiki plugin to the official directory at
<https://www.dokuwiki.org/plugins>, with the conventions used across these repos baked in.

For the meaning and allowed **values** of every data-block field (`type`, `tags`,
`compatible`, …) see the companion [PLUGIN_REPO_REFERENCE.md](PLUGIN_REPO_REFERENCE.md).

> ### Before you share this repo — find & replace
> The examples below use concrete values so they're copy-paste-ready. Swap these for your own
> before publishing the repository publicly:
>
> | Token in examples | Meaning | Replace with |
> |-------------------|---------|--------------|
> | `tracker-user` | GitHub user/org (in URLs) | your GitHub username |
> | `TrackerUser` | Author display name | your name/handle |
> | `for-open-source-projects-trust-building@protonmail.com` | Author email | your contact email (or blank) |
> | `dokuwiki-<name>` | Repo naming pattern | your repo naming pattern |
> | `main` | Default branch (in download/RSS URLs) | `master` if that's your default |

---

## How the directory actually works

The page you create at `plugin:<name>` contains a `---- plugin ----` data block. When you
save the page, the **pluginrepo** plugin parses that block and indexes it into the database
that drives the whole directory: the type filter, the tag cloud, full-text search, and the
**Extension Manager** (which reads `downloadurl` to install your plugin). So getting the data
block right is the entire job — the prose around it is for humans.

---

## Prerequisites

- A **dokuwiki.org account** (the wiki is open but editing the directory needs a login).
- Your plugin on **public GitHub**, containing:
  - `plugin.info.txt` with a correct **`base`** (must equal the install folder / plugin name —
    the Extension Manager renames the unzipped folder to it) and **`url`**.
  - `images/<name>-logo.png` — the listing thumbnail/logo.
  - `images/<name>-screen*.png` — one or more screenshots for the page body.
  - A `README.md` — the source material for the body sections.
- Your default **branch name** (these repos use `main`; many others use `master`).

---

## Step by step

1. **Open the page.** Go to `https://www.dokuwiki.org/plugin:<name>` and log in.
   The id must be exactly **`plugin:<name>`** — one namespace level, lower-case, `a–z0–9`,
   **no underscores** (an underscore separates a component name and also zeroes the
   popularity score). A deeper id won't be indexed.
2. **Create it** ("Create this page") and paste the [template](#page-template) below.
3. **Fill the data block** using [PLUGIN_REPO_REFERENCE.md](PLUGIN_REPO_REFERENCE.md) for
   `type`, `tags`, `compatible`, `depends`/`conflicts`/`similar`.
4. **Write the body** from your `README.md`. Keep the sections that apply; delete the rest
   (e.g. drop **Syntax** for a non-syntax plugin).
5. **Embed screenshots and the changelog RSS** (see conventions below).
6. **Preview, then Save.** Confirm the data box at the top shows the right type, tags,
   compatibility row, and a working **Download** link.

---

## Conventions baked into the template

- **`downloadurl` → branch zipball.** GitHub zips any branch on demand:
  `https://github.com/tracker-user/dokuwiki-<name>/zipball/main`
  (use `master` if that's your default branch). The Extension Manager unpacks it into
  `lib/plugins/<base>`.
- **`screenshot_img` → raw GitHub logo.**
  `https://raw.githubusercontent.com/tracker-user/dokuwiki-<name>/main/images/<name>-logo.png`
  (external URLs are allowed and auto-resized — see reference §6).
- **Changelog → GitHub commits Atom feed.** dokuwiki.org renders external RSS (cached, ~hourly):
  `{{rss>https://github.com/tracker-user/dokuwiki-<name>/commits/main.atom date 8}}`
  (`date` shows commit dates; `8` = number of entries).
- **Body screenshots → raw GitHub `-screen` images.** Embed with a width and caption:
  `{{https://raw.githubusercontent.com/tracker-user/dokuwiki-<name>/main/images/<name>-screen.png?680|Caption}}`
  *(Fallback if dokuwiki.org ever blocks external media: upload the images to the page's media
  namespace and reference them as `:plugin:<name>:<file>` instead.)*
- **usersettings = optional integration, not a dependency.** If the plugin registers a
  per-user toggle via the [usersettings](https://www.dokuwiki.org/plugin:usersettings) plugin,
  say so in the body as **optional** — e.g. *"If the [[plugin:usersettings]] plugin is
  installed, users can enable/disable this from their personal settings page."* **Never** put
  `usersettings` in `depends:` — the plugin must work without it. *(Among these five, only
  `annotations` does this — it registers an `annotations_enabled` toggle.)*
- **Support → GitHub issues (questions welcome too).** Direct both bug reports and questions
  to the issue tracker. Optionally also start a thread on the
  [DokuWiki forum](https://forum.dokuwiki.org/) and link it under a Support/Discussion heading.

---

## Page template

Copy verbatim, then fill the `<…>` placeholders. Only `<name>` and the data values change per
plugin; the identity/URL parts are already concrete.

```dokuwiki
====== <Name> Plugin ======

---- plugin ----
description: <one-line description, from plugin.info.txt "desc">
author     : TrackerUser
email      : for-open-source-projects-trust-building@protonmail.com
type       : <Admin / Action / Helper / ... — see reference §2>
lastupdate : <YYYY-MM-DD>
compatible : Librarian
depends    :
conflicts  :
similar    :
tags       : <3–7 tags from the vocabulary — see reference §5>

downloadurl: https://github.com/tracker-user/dokuwiki-<name>/zipball/main
bugtracker : https://github.com/tracker-user/dokuwiki-<name>/issues
sourcerepo : https://github.com/tracker-user/dokuwiki-<name>/
donationurl:

screenshot_img : https://raw.githubusercontent.com/tracker-user/dokuwiki-<name>/main/images/<name>-logo.png
----

<One- or two-sentence intro: what the plugin does and who it's for. Adapt from README.>

===== Installation =====

Search for **<name>** in the [[plugin:extension|Extension Manager]] and install it, or see
[[:Plugins]] for manual installation. The plugin works out of the box.

<Only if it needs something extra — otherwise delete this note:>
:!: **Optional:** if the [[plugin:usersettings]] plugin is installed, users can toggle
<feature> from their personal settings page. It is **not** required.

===== Usage =====

<What the user sees / does. Adapt from README. Embed a screenshot:>

{{https://raw.githubusercontent.com/tracker-user/dokuwiki-<name>/main/images/<name>-screen.png?680|<Name> in action}}

===== Configuration =====

<Settings exposed in Configuration Manager (from conf/metadata.php), if any. Else delete.>

===== Syntax =====

<ONLY for plugins with a Syntax component. Show the markup and the rendered result. Else delete.>

===== Development =====

Source code: https://github.com/tracker-user/dokuwiki-<name>

==== Changelog ====

{{rss>https://github.com/tracker-user/dokuwiki-<name>/commits/main.atom date 8}}

==== Issues, questions & support ====

Please report bugs — and ask questions — in the
[[https://github.com/tracker-user/dokuwiki-<name>/issues|GitHub issue tracker]].
You can also start a thread on the [[https://forum.dokuwiki.org/|DokuWiki forum]].
```

---

## After publishing

- The plugin becomes installable via the Extension Manager (search its name).
- **Updating:** edit the page and bump `lastupdate`. The `zipball/<branch>` download always
  tracks your branch HEAD, so you don't re-upload anything for code changes.
- **Translations (optional):** create `<lang>:plugin:<name>` (e.g. `de:plugin:<name>`); the
  English `plugin:<name>` page is what feeds the directory index.
