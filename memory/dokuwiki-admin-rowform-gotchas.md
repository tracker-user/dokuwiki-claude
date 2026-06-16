---
name: dokuwiki-admin-rowform-gotchas
description: "Per-row POST buttons in a DokuWiki admin table — avoid nested forms and the `page` param collision"
metadata: 
  node_type: memory
  type: project
---

DokuWiki admin pages (`admin_plugin_*::html()`) that put a per-row state-changing button inside the GET filter form hit two traps:

- **Nested `<form>` is illegal.** The sortable/filterable table sits inside one GET `<form>`. Don't open a POST `<form>` per row inside it. Render the POST form(s) as siblings *after* the GET form and point each button at one via the HTML5 `form="<id>"` attribute (works on FF78). One shared POST form can serve every row — the clicked button supplies the row key via `name=… value=…`.
- **Don't name a button/field `page`.** `do=admin&page=<plugin>` is how the admin dispatcher routes; a button `name="page"` overrides the hidden routing field and the POST misroutes. Use a distinct name (e.g. `clearpage`) and read it with `$INPUT->post->str('clearpage')`.

Confirm dialogs: inline `onclick="return confirm(…)"` is fine (admin `html()` output isn't sanitized); escape the message with `json_encode(... JSON_HEX_APOS|JSON_HEX_QUOT)` then `hsc()` so it's safe at both the JS-string and attribute layers.

Enumerating per-page meta files (e.g. annotations' `.annotations`): `search($data, $conf['metadir'], $cb)` + map each file back with `pathID(substr($file, 0, -strlen('.ext')))`, validate `=== cleanID()`. First implemented in annotations `admin.php` / `helper::getAnnotatedPages()`. Related: [[dokuwiki-admin-pager-filter]].
