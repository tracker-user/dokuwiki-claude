---
name: js-cachebuster
description: "After editing plugin JS/CSS/lang, browsers keep the old bundle until a main config file's mtime changes"
metadata: 
  node_type: memory
  type: reference
---

DokuWiki's front-end JS/CSS bundle URL carries `&tseed=md5($updateVersion + mtimes of the main config files + template style.ini)` (built in `inc/template.php` `tpl_metaheaders`). **Plugin file mtimes are NOT part of `tseed`.** So after editing a plugin's `script.js` / `style.css` / `lang/*`, the server regenerates its own cache (keyed on the source files), but returning browsers keep the OLD bundle because the `tseed` URL is unchanged.

Symptom of a stale bundle: new JS behaviour or strings missing in the browser (e.g. `LANG.plugins.<name>` absent, or old buggy code still running) while a no-store fetch of `lib/plugins/<name>/script.js` shows the new code. Confirmed live 2026-06-02 during the annotations review — the highlight fix only appeared after a cache bust.

To force clients to fetch the new bundle, bump a main config file's mtime (saving any setting in the Configuration Manager does the same):
`docker exec dokuwiki-docker-dokuwiki-1 touch /var/www/html/conf/local.php`

When verifying a JS change live before busting, you can fetch the raw plugin file with `cache:'no-store'` and `eval()` it in the page console to run the new IIFE. See [[environment-paths]].
