---
name: dokuwiki-admin-pager-filter
description: "DokuWiki core has no reusable numbered pager; usermanager pagination/filtering is auth-backend-delegated and can't sort derived columns"
metadata: 
  node_type: memory
  type: reference
---

DokuWiki core ships **no reusable numbered pager** anywhere (grepped the whole tree). The only built-in pagination is in `lib/plugins/usermanager/admin.php`, and it is **prev/next/start/last submit buttons only — no page numbers**. It delegates slicing + filtering to the auth backend: `retrieveUsers($start,$limit,$filter)` + `getUserCount($filter)`. authplain's filter turns each field into a case-insensitive **regex** (`/term/i`) matched on user/name/mail/grps, and it only ever sorts by username.

**Consequence for our admin tables:** any table that sorts by a *derived* column the auth backend can't see (last-seen time, setting value, changed-at) or mixes in data from our own stores must pull all users via `retrieveUsers(0,0)` and **filter + sort + paginate in PHP itself** — the usermanager mechanism cannot be reused.

Applied 2026-06-15 in [[plugin-inventory]] `lastseen` and `usersettings`: each got its own numbered windowed pager (`« 1 … 4 [5] 6 … 20 »`, gap marker = 0 from a `pageWindow()` helper), a per-column text-filter `<tr>` (GET form, `q[<col>]` array params via `http_build_query` in `wl()`, **substring** match via `dokuwiki\Utf8\PhpString::strtolower`+`strpos` — not regex, for safety), and `entries_per_page` config (0 = show all). Logic is duplicated per plugin on purpose (separate repos, only one used at a time). usersettings also keeps its existing Setting drop-down (param `filter`) alongside the text filters.
