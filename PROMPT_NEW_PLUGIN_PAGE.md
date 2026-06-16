<!--
Reusable prompt for generating dokuwiki.org plugin pages in a fresh Claude Code session.
Open a new session in F:\Projects\dokuwiki-docker\storage\lib\plugins and paste everything
below the line. Edit the plugin list / identity tokens for other batches.
-->

# Prompt: generate dokuwiki.org plugin pages

Paste the block below into a new chat (run from the `lib/plugins` directory).

---

Generate **dokuwiki.org directory pages** for these plugins, one DokuWiki-syntax page each,
ready to paste at `https://www.dokuwiki.org/plugin:<name>`:

**sitebackup, hideip, lastseen, usersettings, annotations**

Read these two references first — they define every field value and the conventions; follow
them exactly:

- `_docs/public/PLUGIN_REPO_REFERENCE.md` — `type` bitmask, tag vocabulary, `compatible`
  releases, field semantics, the API.
- `_docs/public/PUBLISHING.md` — the conventions and the page template (start from its
  "Page template" block).

## Fixed values (already concrete — keep as-is)

- Author: `TrackerUser` · Email: `for-open-source-projects-trust-building@protonmail.com`
- Repo: `https://github.com/tracker-user/dokuwiki-<name>` · Branch: `main`
- `downloadurl` → `…/dokuwiki-<name>/zipball/main`
- `bugtracker` → `…/dokuwiki-<name>/issues` · `sourcerepo` → `…/dokuwiki-<name>/`
- `screenshot_img` → `https://raw.githubusercontent.com/tracker-user/dokuwiki-<name>/main/images/<name>-logo.png`
- `compatible : Librarian` (these were tested on Librarian only)
- `lastupdate` → read each `plugin.info.txt` `date` field
- `depends`/`conflicts` → blank unless the code/README proves otherwise.
  `similar` → optionally fill by checking the directory (`api.php?q=…` or
  `api.php?tag[]=…`); else blank.

## Per plugin

For **each** plugin, before writing the page:

1. Read its `README.md` and `plugin.info.txt`.
2. Confirm `type` by listing its component files (`syntax*/action*/admin*/helper*/…`).
3. Pick 3–7 `tags` from the vocabulary in the reference (don't invent new tags).
4. `ls images/` to get the **exact** screenshot filenames (some are `-screen.png`,
   some are `-screen1.png`, `-screen2.png`, …). Embed them in the **Usage** section with
   `{{<raw-url>?600|caption}}`.
5. Build the body from the README: Installation → Usage (with screenshots) → Configuration
   (only if `conf/metadata.php` exists) → Syntax (**only** if it has a Syntax component) →
   Development → Changelog (RSS, `main` branch) → "Issues, questions & support" (GitHub issues
   + optional forum).

Pre-derived facts to **verify, not assume**:

| Plugin | `type` | usersettings | screenshots |
|--------|--------|--------------|-------------|
| sitebackup | `Admin` | none | `sitebackup-screen.png` |
| hideip | `Action, Admin` | none | `hideip-screen.png` |
| lastseen | `Action, Admin, Helper` | none | `lastseen-screen.png` |
| usersettings | `Action, Admin, Helper` | **provider** | `usersettings-screen1..3.png` |
| annotations | `Action, Admin, Helper` | **consumer** | `annotations-screen1..5.png` |

usersettings handling:
- **annotations** registers a per-user `annotations_enabled` toggle via usersettings →
  add the **optional-integration** note from PUBLISHING.md (mention `[[plugin:usersettings]]`,
  do **not** put it in `depends`).
- **usersettings** itself is the provider → in its body, mention that other plugins can add
  per-user toggles through it (optional for them), but it stands alone.

## Output

Save each page to `_docs/pages/plugin-<name>.txt` (raw DokuWiki markup, LF endings) and print
the path. Do **not** edit any plugin code — this task only produces the wiki pages.
