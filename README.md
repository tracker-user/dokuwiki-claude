# DokuWiki Plugin Development with Claude Code

> The `CLAUDE.md` guide and the AI **memory** I used to review, modernize, and build
> DokuWiki plugins with [Claude Code](https://claude.com/claude-code).

This repository is not a plugin. It's the **context** that turns a general-purpose AI
coding assistant into a competent DokuWiki plugin developer — a condensed knowledge base
of the DokuWiki plugin API, coding conventions, a security-focused review checklist, and a
growing pile of hard-won, non-obvious lessons. Drop it into your own project and your AI
assistant starts each session already knowing how DokuWiki plugins actually work.

---

## Why this exists

This was my **first time using AI coding tools like Claude Code** to review and build
software. I'm not new to development, though — I work at a mid level with PHP and JavaScript
and have a broad background across other languages and tech (I used to be a frontend developer
and team lead, even if I don't write code full-time anymore). I mention this because I think
it's *why* the collaboration worked well: I could follow what the assistant was doing, steer
it, and catch when something was off. AI tooling is a force multiplier on top of your own
understanding, not a substitute for it.

With this guide and memory in place, the assistant helped me:

- **build several new plugins** from scratch,
- **modernize over a dozen existing plugins** for current DokuWiki and PHP 8.3, and
- **find and responsibly report numerous XSS vulnerabilities** — the fixes in my own plugins
  shipped before anything here was published; third-party findings were reported privately to
  their authors.

The single biggest lever was giving the AI good, durable context. A capable model with a
blank slate keeps re-learning the same DokuWiki quirks every session and re-making the same
mistakes. The same model with a tight `CLAUDE.md` and an accumulated memory of "here's the
thing that bit us last time" is a different experience entirely.

I'm sharing it so others can skip the cold-start and use it as a **basis for their own
DokuWiki development**. If you've been on the fence about using AI tools for real work, I'd
encourage you to try them — and to bring your own judgment along for the ride.

---

## What's inside

```
.
├── CLAUDE.md                      # Project instructions, auto-loaded by Claude Code each session
├── PROSEMIRROR_PLUGIN_SUPPORT.md  # Deep-dive: adding WYSIWYG (ProseMirror) support to a syntax plugin
├── AI-FOR-GOOD.md                 # Essay: the fact-based case for responsible AI-assisted work
├── PUBLISHING.md                  # How to publish a plugin to dokuwiki.org + page template
├── PLUGIN_REPO_REFERENCE.md       # Directory field/type/tag reference (the data behind it)
├── PROMPT_NEW_PLUGIN_PAGE.md      # Reusable prompt to generate plugin directory pages
└── memory/                        # Persistent AI memory — one lesson per file
    ├── MEMORY.md                  # The index (one line per memory, loaded every session)
    ├── user-profile.md            # Who the assistant is working with
    ├── environment-paths.md       # Local + Docker paths, container name, URLs
    ├── plugin-inventory.md        # Which plugins are done / WIP / skipped
    ├── *-xss-*.md / csrf-*.md     # Security patterns learned from real reviews
    ├── dokuwiki-*.md              # DokuWiki API gotchas verified against source
    └── feedback-*.md / *.md       # Tooling and workflow lessons
```

**`CLAUDE.md`** is the heart of it. Claude Code automatically reads a `CLAUDE.md` from your
project (and `~/.claude/CLAUDE.md` globally) into context at the start of every session.
This one packs the plugin layout, class/file naming rules, PHP/JS/CSS conventions, a
condensed DokuWiki API reference, and a step-by-step review process + checklist — all kept
deliberately lean so it doesn't burn the context budget.

**`memory/`** is the assistant's persistent notebook between sessions. `MEMORY.md` is a
lightweight index that's loaded each session; each linked file holds a **single fact or
lesson** (a security pattern, an API quirk, a tooling tip) with a short note on *why* it
matters and *how to apply* it. They cross-link with `[[wiki-style]]` references. Most of
these were written by the assistant itself as we worked — they're the "don't make that
mistake again" layer.

**`PROSEMIRROR_PLUGIN_SUPPORT.md`** is a standalone technical guide produced during the
work, documenting how to make a syntax plugin render in the ProseMirror WYSIWYG editor.

**`PUBLISHING.md`**, **`PLUGIN_REPO_REFERENCE.md`** and **`PROMPT_NEW_PLUGIN_PAGE.md`** cover
publishing a finished plugin to the official directory at <https://www.dokuwiki.org/plugins>:
the step-by-step guide and page template, the reusable field/`type`/tag reference behind it
(extracted once from the repository plugin and the live tag cloud), and a ready-to-paste
prompt that generates the directory pages. The published plugin pages each carry a short
**"Made with AI assistance"** disclosure that links back to the `dokuwiki-claude` repo.

**`AI-FOR-GOOD.md`** is a standalone essay — a fact-based, sourced reply to the common criticisms
of AI-assisted development (environmental cost, copyright, and the "it's just slop" objection).
Those published disclosure sections link to it for readers who are skeptical of the approach.

---

## This is shaped to *my* environment — you will need to adapt it

The guide and memory describe a very specific setup. Treat the specifics as examples, not
gospel, and change them to match yours:

| Thing | My value (change it) |
|------|----------------------|
| DokuWiki version | "Librarian" 2025-05-14b |
| Host OS | Windows 10, paths like `F:\Projects\dokuwiki-docker\...` and `F:\Projects\dokuwiki-stable` |
| Runtime | Docker, container `dokuwiki-docker-dokuwiki-1`, PHP 8.3 / Debian 13 |
| Wiki URL / login | `http://dokuwiki.local/`, `admin` / `admin` (local dev only) |
| Browser floor | **Firefox 78 ESR** — this heavily constrains the allowed JS/CSS |
| Translations | I add **English + German, Russian, Japanese** to everything |
| Attribution | `tracker-user` / `TrackerUser` is my handle, used in `plugin.info.txt` conventions |

A few of these are worth calling out because they ripple through everything:

- **Firefox 78 ESR floor.** Much of the JS/CSS guidance (the "FORBIDDEN" lists) exists only
  because I support a very old browser. If your floor is modern, you can delete those
  constraints and let the assistant use current syntax.
- **Four-language translations.** I chose to ship `de` / `ru` / `ja` alongside English.
  That's a personal requirement, not a DokuWiki one — drop or change the language list.
- **Hardcoded absolute paths.** They're all over `CLAUDE.md` and `environment-paths.md`.
  Search-and-replace them for your machine, or the shell/lint commands won't run.
- **Docker + PowerShell quirks.** Several tooling notes (run `docker exec` from PowerShell,
  not Git Bash; `Select-String` instead of the ripgrep-based search; cache-busting) are
  Windows/Docker-specific. Harmless to keep, but only relevant on a similar setup.

### Adapt-before-you-use checklist

1. Replace every `F:\Projects\...` path with yours.
2. Set your DokuWiki version, container name, wiki URL, and login.
3. Set (or remove) the **browser floor** and its JS/CSS rules.
4. Set (or remove) the **translation languages**.
5. Replace the `tracker-user` / `TrackerUser` attribution convention with your own.
6. Start your **own** `memory/` — keep the DokuWiki/API/security lessons (they're general),
   and let the assistant write new memories as *you* work. The `plugin-inventory.md` and
   `*-audit-clean.md` files describe **my** plugins and audit state — replace them.

---

## How to use it

1. Copy `CLAUDE.md` to the root of the project you open Claude Code in (for plugin work,
   that's typically your `lib/plugins/` directory), or to `~/.claude/CLAUDE.md` to make it
   global. Edit it for your environment (see above).
2. If your tool supports a file-based memory, seed it from `memory/` — or just keep these
   as reference docs and let your assistant build memory from scratch.
3. Open Claude Code and ask it to review or build a plugin. The conventions and checklist in
   `CLAUDE.md` give it a repeatable process; the memory keeps it from relearning the same
   quirks each time.

You don't need Claude Code specifically — any AI coding assistant that can load project
context will benefit from `CLAUDE.md`. The memory format is just Markdown.

---

## A note on the security content & responsible disclosure

Part of this work was security review, so several memory files document **real XSS / CSRF
patterns** — the bug shapes, how to find them, and how to fix them. These are general lessons
worth learning from. The findings in my **own** plugins were fixed and shipped before anything
here was published. For third-party plugins I've deliberately **kept the techniques but removed
the plugin names and proof-of-concept payloads**: those issues were reported privately to the
authors, and the goal here is to help people write *safer* plugins — not to hand out exploits.

---

## License

Released under the **GNU General Public License v2.0** (see [`LICENSE`](LICENSE)) — the same
license I use for the plugins themselves. The code snippets in `CLAUDE.md` and
`PROSEMIRROR_PLUGIN_SUPPORT.md` are short, illustrative, and meant to be reused freely; a link
back is appreciated but not required.

---

*Built while learning AI-assisted development on a real project. If it helps you take your
first step, that's the whole point.*
