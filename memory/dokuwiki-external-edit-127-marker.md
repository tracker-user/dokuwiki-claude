---
name: dokuwiki-external-edit-127-marker
description: "DokuWiki hardcodes 127.0.0.1 as its external-edit IP marker; REMOTE_ADDR-based anonymizers can't catch it"
metadata: 
  node_type: memory
  type: reference
---

DokuWiki **hardcodes `127.0.0.1`** as the IP for "external edit" entries — synthesized in `inc/ChangeLog/ChangeLog.php` `getCurrentRevisionInfo()` (~lines 632, 671) whenever a page file's on-disk mtime no longer matches its changelog (file created/edited directly on disk, not via the wiki). Common in bind-mounted/Docker setups and for shipped pages with no changelog (e.g. `wiki:syntax`).

It re-appears on two paths, both re-create it forever:
- **Every view** → `pageinfo()` self-heal (`inc/common.php` ~271-287) persists it into page metadata `last_change.ip` in BOTH `current` and `persistent` branches (because `p_set_metadata` defaults `$persistent=true`) → shows as "2 IP slots in 1 .meta file".
- **Next save** of such a page → `detectExternalEdit()` (`inc/File/PageFile.php` ~257) writes the line to per-page + master changelog.

Key facts: it's a **literal, not from `$_SERVER`**, so a REMOTE_ADDR-overwrite anonymizer (like [[plugin-inventory]]'s hideip action component) cannot intercept it. There is **no DokuWiki event** wrapping `addLogEntry()` or `p_save_metadata()` to hook. It's loopback — not a real visitor IP, leaks nothing. Real edits' IPs come from `clientIP(true)` → `Ip::clientIps()` → reads `$INPUT->server` which binds `&$_SERVER` by reference, so the hook DOES anonymize those to `0.0.0.0`.

Resolution chosen for hideip (2026-06-15): treat `127.0.0.1` as a no-action value alongside `0.0.0.0`/blank in the admin preview/scrub — never counted, never rewritten — rather than fight the treadmill. See its README "Why 127.0.0.1 shows up".
