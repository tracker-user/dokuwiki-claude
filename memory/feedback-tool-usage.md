---
name: feedback-tool-usage
description: "Tool-call efficiency rules — PowerShell for docker, Chrome for anti-scraping-blocked sites, absolute paths, Read-before-Edit"
metadata: 
  node_type: memory
  type: feedback
---

Recurring wasted tool calls observed across many past sessions in this project. Avoid them from the first call to save usage.

1. **PowerShell for docker / container paths.** The Bash tool is Git Bash; MSYS rewrites Unix absolute paths (`/storage/...`, `/var/www/html/...`, `/tmp/...`) into `C:/Program Files/Git/...`, so `docker exec ... php -l /storage/...` fails with "Could not open input file". Run these in PowerShell from the start (hit in ~17 past sessions). The operational rule also lives in CLAUDE.md "Shell". In PowerShell use `2>$null`, never `2>/dev/null` ("Missing file specification after redirection operator").
2. **Don't fan out unverified parallel Bash calls** sharing the same command/path form — if one fails, the whole parallel batch is cancelled ("Cancelled: parallel tool call Bash(...)"). One mangled-path call cancelled ~13 siblings in a single session. Verify the form with one call, then parallelize.
3. **External docs/pages: use the Chrome browser, not WebFetch.** Raw requests (WebFetch) to sites like dokuwiki.org are blocked by anti-scraping (HTTP 402 / challenge pages). Chrome is set up and usable — navigate there, the user clears any challenge/CAPTCHA quickly, then extract the text/data (`get_page_text` / `read_page`). For DokuWiki internals still prefer the local `F:\Projects\dokuwiki-stable` source first (token-efficiency); reach for Chrome when you genuinely need a live web page. Local container `dokuwiki.local` is reachable too.
4. **Reads: absolute paths, confirm before guessing.** Repeated "File does not exist. Note: your current working directory is …plugins" came from relative/guessed paths. Read `dokuwiki-stable` files by absolute path; if unsure a source file/subpath exists, Glob first instead of guessing.
5. **Read before Edit/Write** — editing an unread file fails ("File has not been read yet").
6. **Chrome browser is available** (the user set it up properly) for external pages and for frontend testing at `dokuwiki.local`. If a page needs a login or anti-bot challenge, the user can clear it interactively — proceed once it's passed.
7. **Grep tool silently skips vendored plugin dirs — use PowerShell for completeness checks.** The Grep tool (ripgrep) returned only 3 matches for `DOKU_INC` across `storage/lib/plugins/`, missing every actual plugin (annotations, move, include, edittable, note, …); `Get-ChildItem -Recurse -Force | Select-String` found them all. Cause is ripgrep's default ignore/symlink handling in this tree (the third-party plugin dirs are skipped). Lesson: for "does X exist anywhere / how many files" sweeps in `lib/plugins/`, cross-check with PowerShell before concluding "none" — a false-empty Grep led to a wrong conclusion this session. Grep is still fine for targeted searches of files you know it traverses.

**Why:** The user flagged repeated Git-Bash→PowerShell retries spending usage and asked to eliminate these extra calls across the board.
**How to apply:** Reach for PowerShell (not Bash) for anything touching the container or Unix paths; treat external web as unavailable and lean on local source + static analysis; use absolute paths and Read-before-Edit. See [[environment-paths]], [[token-efficiency]], [[feedback-plugin-git-commit]].
