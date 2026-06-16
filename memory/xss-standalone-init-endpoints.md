---
name: xss-standalone-init-endpoints
description: Plugin standalone PHP scripts that bootstrap inc/init.php with no auth = reflected XSS / ACL-bypass / LFI class
metadata:
  node_type: memory
  type: reference
---

A whole vuln class found in third-party DokuWiki plugins: a plugin ships a **standalone
`.php` script** (under `exe/`, `scripts/`, or the plugin root) that is reachable by direct
URL and does `require_once DOKU_INC.'inc/init.php'` itself, then reads request input and
`echo`s — **bypassing** the normal `action.php` / `AJAX_CALL_UNKNOWN` path, so there is **no
ACL check, no admin check, no `checkSecurityToken()`**. Result: unauthenticated reflected
XSS, ACL bypass / info disclosure, and path traversal.

Shapes seen in the wild (plugin names and payloads omitted — responsible disclosure):
- A standalone script echoed a `$_GET` index raw inside a `die('… '.$index.' …')` message →
  unauthenticated reflected XSS; the same script also `rawWiki()`'d a `$_GET['id']` with no
  ACL check → it could read blocks from any page.
- A data/stats endpoint had no auth or CSRF and echoed several `$_POST` fields raw → unauth
  reflected XSS via a cross-site auto-submit POST. (Transport can hide it: if the normal table
  path `rawurlencode`s output and the JS double-`decodeURIComponent`s + `innerHTML`s it, naive
  direct-nav XSS is defeated — but an un-encoded prologue echoed elsewhere in the same script
  bypasses that.) A `meta_path`-style POST param also enabled path traversal into
  `unserialize(io_readFile(...))`.
- A help/include script concatenated `$INPUT->get->str('syntax')` into a `help/<syntax>.txt`
  path with no sanitization → traversal / arbitrary `.txt` read + ACL bypass (the content was
  `p_render`'d so not XSS, but it disclosed protected pages).

Same pattern but **safe** when there's no reflected input: a static icon/data popup (its `ns`
is `cleanID`'d), a GIF beacon (no echo), an XML feed generator (the library XML-escapes).

Find them fast (Grep is .gitignore-blind on plugin dirs — use PowerShell, see
[[grep-gitignore-blind]]):
`Get-ChildItem <dir> -Recurse -Include *.php | Select-String 'inc/init\.php'` then read any
hit that isn't a normal action/syntax/admin file and check for echo of request input.
NOTE: such scripts compute `DOKU_INC` via `__DIR__.'/../../../'` assuming an install at
`lib/plugins/<name>/`; they break (wrong depth) when nested deeper, so they can't be
curl-tested from a staging subdir without moving them first.
See [[syntax-attr-injection-xss]], [[xss-comment-breakout-and-validator-gaps]].
