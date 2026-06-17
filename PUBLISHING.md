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
  `{{https://raw.githubusercontent.com/tracker-user/dokuwiki-<name>/main/images/<name>-screen.png|Caption}}`
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
- **AI-assisted disclosure (every original/adopted plugin).** Carry the `ai-assisted` tag, a
  `:!:` notice right after the intro that links down to a final **"Made with AI assistance"**
  section, and that section itself — the `enhanced.jpg` banner, the short standard text, a link to
  [AI-FOR-GOOD.md](AI-FOR-GOOD.md) for skeptics, and a pointer to the
  [dokuwiki-claude](https://github.com/tracker-user/dokuwiki-claude) repo. The banner shows the
  `enhanced-thumb.jpg` thumbnail wrapped in a `[[…|{{ …}}]]` link to the full-size `enhanced.jpg`
  (don't resize with `?width` — dokuwiki.org renders the scaled copy poorly; ship a real
  thumbnail instead). Both are hot-linked from that shared repo
  (`…/dokuwiki-claude/main/images/`), **not** each plugin's own `images/`, and the banner
  right-aligns purely via the **single leading space** inside the inner `{{ …}}` — DokuWiki's
  align-by-whitespace rule (`{{ img}}` = right, `{{img }}` = left, `{{ img }}` = center; an editor
  that trims it silently kills the float). Tweak the wording per plugin, but keep the disclosure.

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
tags       : ai-assisted, <plus ~3–6 more from the vocabulary — see reference §5>

downloadurl: https://github.com/tracker-user/dokuwiki-<name>/zipball/main
bugtracker : https://github.com/tracker-user/dokuwiki-<name>/issues
sourcerepo : https://github.com/tracker-user/dokuwiki-<name>/
donationurl:

screenshot_img : https://raw.githubusercontent.com/tracker-user/dokuwiki-<name>/main/images/<name>-logo.png
----

===== Description =====

<Intro: what the plugin does and who it's for. Keep it short, or let it run as long as you like —
adapt it from, or simply copy, the opening of the README.>

:!: This is an **AI-assisted plugin**. That doesn't mean it's bad — quite the opposite.
[[#made_with_ai_assistance|Read the details]].

===== Installation =====

Search for **<name>** in the [[plugin:extension|Extension Manager]] and install it, or see the
[[:Plugin Installation Instructions]] for manual installation.

<Only if it needs something extra — otherwise delete this note:>
:!: **Optional:** if the [[plugin:usersettings]] plugin is installed, users can toggle
<feature> from their personal settings page. It is **not** required.

===== Usage =====

<What the user sees / does. Adapt from README. Embed a screenshot:>

{{https://raw.githubusercontent.com/tracker-user/dokuwiki-<name>/main/images/<name>-screen.png|<Name> in action}}

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

===== Made with AI assistance =====

[[https://raw.githubusercontent.com/tracker-user/dokuwiki-claude/main/images/enhanced.jpg|{{ https://raw.githubusercontent.com/tracker-user/dokuwiki-claude/main/images/enhanced-thumb.jpg|AI-assisted DokuWiki development}}]]

This plugin was either fully AI-generated or heavily AI-assisted — and I think that's a good
thing. The idea is my own, I use the plugin myself, and I test it by hand; the actual code is
written and changed by an LLM from my prompts, guidance, and review afterwards.

Working this way I can hold a high bar — no XSS, current coding standards, proper escaping and
CSRF handling — and keep the plugin maintained. I think that compares well with much of the
existing plugin repository, where XSS holes are common: I found and reported [[https://raw.githubusercontent.com/tracker-user/dokuwiki-claude/main/images/xss-reports.png|close to twenty]]
myself, and only from quick broad checks rather than thorough reviews, so there are surely many
more — and many plugins are simply abandoned besides.

To be upfront: I can't claim to follow every advanced technique the model may put in the code. I
do read and understand most of it, and nothing ships without my review — but I won't pretend
otherwise.

If your own principles, or your company's policy, don't allow using anything touched by AI, feel
free to skip this plugin — no hard feelings.

If you're more broadly skeptical of AI-assisted work — the energy and water cost, the copyright
questions, the "it's just slop" worry — I tried to answer those honestly and with sources in
[[https://github.com/tracker-user/dokuwiki-claude/blob/main/AI-FOR-GOOD.md|AI Can Be Used for Good]].
It's worth a read even if you end up disagreeing.

Curious how this is done? The setup, the development guide, and the AI memory behind these plugins
are public in my [[https://github.com/tracker-user/dokuwiki-claude|dokuwiki-claude]] repository —
a starting point if you'd like to build a plugin the same way.
```

---

## After publishing

- The plugin becomes installable via the Extension Manager (search its name).
- **Updating:** edit the page and bump `lastupdate`. The `zipball/<branch>` download always
  tracks your branch HEAD, so you don't re-upload anything for code changes.
- **Translations (optional):** create `<lang>:plugin:<name>` (e.g. `de:plugin:<name>`); the
  English `plugin:<name>` page is what feeds the directory index.
