---
name: plugin-publishing-workflow
description: How/where to publish these plugins to dokuwiki.org; the field+tag reference and the reusable page-generation prompt
metadata:
  node_type: memory
  type: project
---

Publishing the custom plugins to the official directory (<https://www.dokuwiki.org/plugins>).
Three docs live in `_docs/public/` (the shareable repo):

- **PUBLISHING.md** — how-to + the ready-to-fill DokuWiki page template + the repo conventions.
- **PLUGIN_REPO_REFERENCE.md** — authoritative `type` bitmask (from the pluginrepo plugin's
  `helper/repository.php` `$types`: 1 Syntax,2 Admin,4 Action,8 Render,16 Helper,32 Template,
  64 Remote,128 Auth,256 CLI,512 CSS/JS-only — empty `type:` ⇒ CSS/JS-only); the full **309-tag
  vocabulary** captured **2026-06-17** from the live tag cloud (`.repoCloud .cloud a`, weight
  `cl0`–`cl5`); current `compatible` releases (Librarian 2025-05-14 newest … Greebo 2018-04-22);
  JSON API at `…/lib/plugins/pluginrepo/api.php`.
- **PROMPT_NEW_PLUGIN_PAGE.md** — reusable prompt to generate the pages in a fresh chat.

Conventions decided with the user: `downloadurl`→`…/zipball/main`; `screenshot_img`→raw GitHub
`images/<name>-logo.png` (external URLs are allowed — proxied via `ml()`/`fetch.php`); changelog
via `{{rss>…/commits/main.atom date 8}}`; body screenshots from `images/<name>-screen*.png`;
**usersettings = optional prose mention, never `depends`**. Page id must be exactly `plugin:<name>`.
Identity in the docs is kept concrete, with a find-&-replace table at the top of PUBLISHING.md to
genericize before sharing the repo publicly.

**AI-assisted disclosure (decided 2026-06-17, applies to every original/adopted plugin page):**
each generated page must carry the `ai-assisted` tag, a `:!:` notice right after the intro that
links to `[[#made_with_ai_assistance]]`, and a final `===== Made with AI assistance =====`
section (the `enhanced.jpg` banner + short standard text + a link to `AI-FOR-GOOD.md` for
skeptics + pointer to the shared `dokuwiki-claude` repo). Banner is hot-linked from the shared
dokuwiki-claude repo (`…/main/images/enhanced.jpg`), NOT each plugin's `images/`,
and right-aligns ONLY via the single leading space inside `{{ …}}` (DokuWiki align-by-whitespace,
confirmed `inc/parser/handler.php` Doku_Handler_Parse_Media — don't let it get trimmed). The standard
wording lives in PUBLISHING.md's page template (also threaded through PLUGIN_REPO_REFERENCE.md §5
and PROMPT_NEW_PLUGIN_PAGE.md). Intro length is now free (short or copy the README opening).

Repos: `github.com/<user>/dokuwiki-<name>` (note: `dokuwiki-<name>`, not
`dokuwiki-plugin-<name>`), branch `main`. **5 plugins pending publication** with derived types:
sitebackup=`Admin`; hideip=`Action,Admin`; lastseen/usersettings/annotations=`Action,Admin,Helper`.
Only **annotations** consumes usersettings (registers an `annotations_enabled` toggle); usersettings
itself is the provider. The actual per-plugin pages are to be generated in a separate chat via the
prompt — not yet done. See [[plugin-inventory]], [[usersettings-api]].
