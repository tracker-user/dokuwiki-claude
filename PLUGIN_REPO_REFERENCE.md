# DokuWiki Plugin Repository — Field & Tag Reference

Reusable reference data for filling the `---- plugin ----` data block when you publish a
plugin page at `https://www.dokuwiki.org/plugin:<name>`.

This file exists so you **don't have to re-parse the plugin listing** every time. It was
compiled once from two authoritative sources:

- The **pluginrepo** plugin source (the code that actually powers the listing) —
  `helper/repository.php` and `syntax/entry.php`. This is the source of truth for the
  `type` bitmask and every accepted field.
- The **live tag cloud** at <https://www.dokuwiki.org/plugins> — captured **2026-06-17**,
  309 tags. Re-capture occasionally if you want fresh frequencies (see *Re-capturing* at
  the bottom), but the vocabulary changes slowly.

See [PUBLISHING.md](PUBLISHING.md) for the step-by-step how-to and the ready-to-fill page
template; this file is just the field/value data behind it.

---

## 1. The `---- plugin ----` data block

The page body can be anything, but the listing only reads the fenced data block:

```
---- plugin ----
key : value
...
----
```

Parsing rules (from `helper_plugin_pluginrepo_repository::parseData()`):

- One `key : value` per line. Keys are lower-cased.
- A `#` starts a comment to end of line (escape a literal `#` as `\#`, a backslash as `\\`).
- A leading `  * ` bullet is stripped (so you may bullet the External-requirements list).
- A repeated key is **space-concatenated**, not overwritten.
- Every value is truncated to **255 characters**.

| Field | Required | Format / notes |
|-------|----------|----------------|
| `description` | ✅ | One-line summary, plain text, ≤255 chars. Shown in the listing and search index. |
| `author` | ✅ | Display name. |
| `email` | ➖ | Author email. Rendered spam-obfuscated on the page; its MD5 groups "other plugins by the same author". Leave blank to show name only. |
| `type` | ✅ | Comma-separated component labels — see **§2**. |
| `lastupdate` | ✅ | `YYYY-MM-DD`. Drives the "this plugin is old" notice (≥2 years old **and** not marked compatible with a recent release). |
| `compatible` | ✅ | Comma-separated DokuWiki release names/dates — see **§3**. |
| `depends` | ➖ | Comma-separated extension IDs that **must** be installed — see **§4**. |
| `conflicts` | ➖ | Comma-separated extension IDs that break if co-installed — see **§4**. |
| `similar` | ➖ | Comma-separated related/alternative extension IDs — see **§4**. |
| `tags` | ✅ | Space/comma/semicolon-separated tags from the vocabulary — see **§5**. |
| `downloadurl` | ✅ | ZIP the Extension Manager installs. GitHub: `…/zipball/<branch>`. **Without it the plugin is not installable** and the page shows a warning. |
| `bugtracker` | ➖ | Issue tracker URL. |
| `sourcerepo` | ➖ | Source repository URL. |
| `donationurl` | ➖ | Donation link (renders a "donate" button). |
| `screenshot_img` | ➖ | Media ID **or external URL** of a thumbnail/logo — see **§6**. |

**Maintainer-only fields** (set by repository admins, not authors — leave blank):
`securityissue`, `securitywarning`, `updatemessage`. Listed here only so you recognise them;
don't fill them yourself.

---

## 2. `type` — authoritative bitmask

From `helper_plugin_pluginrepo_repository::$types`. You write the **labels**; the repo ORs
them into a bitmask. Matching is a **case-insensitive substring** match, and you separate
labels with commas.

| Bit | Label | Plugin component (file / class) |
|----:|-------|----------------------------------|
| 1 | `Syntax` | `syntax.php` or `syntax/*.php` |
| 2 | `Admin` | `admin.php` or `admin/*.php` |
| 4 | `Action` | `action.php` or `action/*.php` |
| 8 | `Render` | `renderer.php` (custom output format) |
| 16 | `Helper` | `helper.php` or `helper/*.php` |
| 32 | `Template` | *(templates only — set automatically in the `template:` namespace; never put it on a plugin page)* |
| 64 | `Remote` | `remote.php` (XML-RPC / JSON-RPC) |
| 128 | `Auth` | `auth.php` |
| 256 | `CLI` | `cli.php` |
| 512 | `CSS/JS-only` | *(empty `type:` → this; use **only** for plugins that ship no PHP component classes)* |

**How to fill it:** list every PHP component your plugin defines.
Real example — the `move` plugin: `type : admin, action, helper`.

Rules:
- List **all** components, comma-separated, in any case/order.
- An **empty** `type:` is interpreted as **CSS/JS-only (512)**. Only leave it empty if the
  plugin truly has no PHP component (pure `script.js`/`style.css`). Shipping JS/CSS
  *alongside* PHP components does **not** add a type.
- Don't write `Template` — that's namespace-driven for the separate template repo.

**Derived types for these five plugins** (from the component files present — verify before publishing):

| Plugin | Component files | `type` |
|--------|-----------------|--------|
| `sitebackup` | `admin.php` | `Admin` |
| `hideip` | `action.php`, `admin.php` | `Action, Admin` |
| `lastseen` | `action.php`, `admin.php`, `helper.php` | `Action, Admin, Helper` |
| `usersettings` | `action.php`, `admin.php`, `helper.php` | `Action, Admin, Helper` |
| `annotations` | `action.php`, `admin.php`, `helper.php` | `Action, Admin, Helper` |

---

## 3. `compatible` — DokuWiki releases

Comma-separated DokuWiki **release codenames** (most authors) or ISO dates. Modifiers:

- `Name` — compatible with that release.
- `!Name` — explicitly **not** compatible (e.g. `!igor`).
- `Name+` — compatible with that release **and all later ones** (implicit forward-compat).
- `devel` — only compatible with the development build.

Matching is case-insensitive; codename or date both work. Only releases the repo knows are
recognised; the listing's compatibility box shows the **most recent four**.

Current releases (newest first):

| Codename | Date |
|----------|------|
| **Librarian** | 2025-05-14 *(current)* |
| Kaos | 2024-02-06 |
| Jack Jackrum | 2023-04-04 |
| Igor | 2022-07-31 |
| Hogfather | 2020-07-29 |
| Greebo | 2018-04-22 |

**Recommendation:** list only releases you've actually tested. These plugins were built and
tested on **Librarian**, so `compatible : Librarian` is honest and sufficient. Add older
codenames only if verified. (Real examples: `move` → `Greebo, Hogfather, Igor, Jack Jackrum,
Kaos, Librarian`; `gallery` → `!greebo, !hogfather, !igor, Jack Jackrum, Kaos, Librarian`.)

---

## 4. `depends` / `conflicts` / `similar`

All three take **comma-separated extension IDs** — a plugin's base name (`move`, `wrap`), or
`template:<name>` for a template. The repo lower-cases and de-duplicates them.

- **`depends`** — *hard* requirements only. The Extension Manager surfaces these.
  ⚠️ Do **not** list *optional* integrations here (e.g. `usersettings`). If the plugin works
  without it, mention it in prose instead — see [PUBLISHING.md](PUBLISHING.md) §usersettings.
- **`conflicts`** — extensions that break when installed together. **Bidirectional**: if A
  declares a conflict with B, B's page shows it too.
- **`similar`** — alternatives / related extensions. Also **bidirectional**.

---

## 5. `tags` — the controlled vocabulary

Tags are free-form, lower-cased, and split on commas/semicolons/whitespace — but you should
pick from the **established vocabulary** so your plugin is discoverable through the tag cloud
and tag filters. Aim for **3–7 tags**: what it does, the content domain it touches, and any
integration surface.

### Status tags (special — they change how the listing treats the plugin)

| Tag | Effect |
|-----|--------|
| `!obsolete` | **Hides** the plugin from the default listing. Use when truly dead. |
| `!discontinued` | Flags it as no longer maintained. |
| `!broken` | Flags it as currently broken. |
| `!experimental` | Flags it as experimental/unstable. |
| `!bundled` | Marks a plugin bundled with DokuWiki core. |

### Full vocabulary (309 tags, by cloud frequency — tier 5 = most used, 0 = rare)

**Tier 5 (18):** authentication, code, diagram, editing, embed, formatting, images, include, links, listing, media, namespace, navigation, search, tables, users · *(plus status `!discontinued`, `!experimental`)*

**Tier 4 (41):** acl, ajax, annotations, boxes, button, calendar, chart, data, database, date, email, export, feed, file, form, google, groups, hide, highlight, html, javascript, language, list, maintenance, maps, math, menu, meta, oauth, page, poll, redirect, spam, statistics, style, syntax, syntaxhighlight, tags, template, toolbar, typography

**Tier 3 (64):** admin, blog, bookmark, bugtracker, cache, calculation, collapsible, counter, create, css, discussion, download, editor, facebook, formula, gallery, geo, git, header, icons, image, import, index, ip, issue, latex, logging, login, management, markup_language, mediawiki, music, mysql, news, notification, odt, password, pdf, php, popup, quotes, random, references, replace, repository, revisions, rss, section, security, seo, sidebar, slideshow, social, sqlite, sso, struct, svg, task, time, todo, tooltip, video · *(plus status `!broken`, `!bundled`)*

**Tier 2 (86):** 2fa, 404, ai, alert, analytics, auth, barcode, bibtex, blacklist, books, bootstrap, box, breadcrumb, captcha, cas, changelog, charts, chat, clipboard, cms, color, command, comment, config, configuration, copy, csv, definitions, delete, display, doodle, encryption, extension, filter, game, github, graph, groupmail, headings, if, jquery, json, keyboard, lightbox, macro, markdown, metadata, moderation, move, network, numbering, orphan, paste, plain, plugins, preview, printing, projects, publications, rename, science, semantic, server, share, shortcut, snippets, spatial, sql, status, sync, tabs, text, tickets, timeline, title, toc, tracking, translation, twitter, two-factor, update, upload, variables, vote, wysiwyg, xml

**Tier 1 (44):** ad, api, archive, bugzilla, caption, chemistry, comic, contact, convert, devel, disable-actions, documentation, docx, farm, flowchart, hide-menus, html5, iframe, interwiki, java, ldap, like, log, mantis, mathml, medical, pages, performance, piwik, qrcode, quiz, rack, rating, rendering, shibboleth, signature, sort, source, statichtml, subscription, uml, userpage, wikipedia, xmlrpc

**Tier 0 (56):** abstract, address, advertising, anchor, backup, biology, bpmn, cage, carousel, cloud, collaboration, columns, denied, deprecated, disqus, dropdown, ebook, events, flash, fold, fonts, footer, footnotes, gantt, glossary, helper, history, icon, marking, mediamanager, minecraft, missing, mobile, note, orgchart, otp, overlay, permissions, plot, prismjs, privacy, progressbar, proof, regexp, saml, schedule, sequence, sitemap, soap, spoiler, storage, test, tex, thumbnail, toggle, url

### Suggested tags for these five plugins (all drawn from the vocabulary above — finalise per plugin)

| Plugin | Candidate tags |
|--------|----------------|
| `sitebackup` | `backup`, `export`, `archive`, `admin`, `maintenance` |
| `hideip` | `ip`, `privacy`, `security`, `hide`, `admin` |
| `lastseen` | `users`, `statistics`, `tracking`, `logging`, `admin` |
| `usersettings` | `users`, `configuration`, `admin`, `toggle` |
| `annotations` | `annotations`, `comment`, `discussion`, `collaboration`, `note` |

> There is **no** `usersettings` tag — usersettings integration is mentioned in prose, not tagged or depended-on.

---

## 6. `screenshot_img`

Accepts a DokuWiki **media ID** (uploaded to dokuwiki.org) **or an external URL**. It's run
through DokuWiki's `ml()` and resized (≈220 px on the plugin page, 80–120 px in the table),
and external URLs are proxied/cached through `fetch.php` — so hot-linking a raw image works.
(Real example: the `gallery` plugin uses `https://i.imgur.com/rj4v2c3.png`.)

**Convention for these repos:** point it at the plugin's logo in its GitHub `images/` folder:

```
screenshot_img : https://raw.githubusercontent.com/tracker-user/dokuwiki-<name>/main/images/<name>-logo.png
```

(Every plugin here has `images/<name>-logo.png` plus one or more `images/<name>-screen*.png`
used for in-body screenshots — see [PUBLISHING.md](PUBLISHING.md).)

---

## 7. Query the repo via API instead of scraping

For programmatic lookups (find existing tags, types, or similar plugins) use the JSON API
rather than parsing HTML:

```
https://www.dokuwiki.org/lib/plugins/pluginrepo/api.php
```

Parameters (all AND-combined): `ext[]=<name>` · `q=<fulltext>` · `type=<bitmask sum>` ·
`tag[]=<tag>` · `mail[]=<md5-of-email>` · `order=lastupdate|popularity|plugin` ·
`limit=<n>` · `fmt=json|yaml|xml|php|debug`.

Examples: `api.php?ext[]=move` · `api.php?tag[]=backup&fmt=json` · `api.php?type=5` (Syntax+Action).

### Re-capturing the tag cloud

If you want fresh tag frequencies, open <https://www.dokuwiki.org/plugins> and run in the
console:

```js
[...document.querySelectorAll('.repoCloud .cloud a')]
  .map(a => a.textContent.trim()).sort().join(', ')
```

(The cloud's CSS class `cl0`–`cl5` is the frequency tier used to bucket the lists above.)
