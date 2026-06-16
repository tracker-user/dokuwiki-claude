---
name: jsinfo-embed-xss
description: "Embedding user data into the inline JSINFO <script> needs JSON_HEX_TAG or it's stored XSS"
metadata: 
  node_type: memory
  type: project
---

When a plugin appends user-controlled data into the page's inline `var JSINFO = {...}`
script (the `TPL_METAHEADER_OUTPUT` technique: find the inline `<script>` in
`$event->data['script']` and append `JSINFO.foo={...};`), the `json_encode` flags matter
for security, not just size.

- `json_encode` **default** escapes `/` → `\/`, so `</script>` becomes `<\/script>` —
  accidentally safe against the closing-tag breakout.
- Adding `JSON_UNESCAPED_SLASHES` **removes** that protection: a user body containing
  `</script><img src=x onerror=...>` closes the script element → **stored XSS** in every
  viewer's browser (admins included). Found exactly this in the **annotations** plugin
  (2026-06, body field embedded inline).
- Fix: add **`JSON_HEX_TAG`** (escapes `<`→`<`, `>`→`>`). That neutralises
  `</script>`, `<!--`, `<script` regardless of slash escaping. Inside a `<script>`
  raw-text element HEX_TAG is sufficient; `&`/quotes don't need escaping there.

Verify quickly: `json_encode(["b"=>"</script>"], JSON_HEX_TAG)` → `"<\/script>"`.

The AJAX `application/json` response path is safe (parsed as JSON, not HTML) — only the
inline-`<script>` embed path is vulnerable.
