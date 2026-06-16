---
name: js-lang-per-user
description: "Why plugin JS strings ignore the usersettings per-user language, and how usersettings fixes it"
metadata: 
  node_type: memory
  type: reference
---

DokuWiki's JS language bundle (`LANG`, `LANG.plugins.<name>`) is built by `lib/exe/js.php` using `$conf['lang']` (`js_pluginstrings()` reads `lang/<conf-lang>/lang.php`). js.php is a **separate request** that never fires `ACTION_ACT_PREPROCESS`, so the usersettings `applyUserLang` handler (hooked there) does NOT run for it — JS strings always shipped in the SITE-default language even when the page UI followed the user's chosen language.

**CRITICAL gotcha (verified 2026-06-03):** js.php defines `NOSESSION` (js.php:17) — no session is started, so `$INPUT->server->str('REMOTE_USER')` is **empty** even with a valid login cookie. Any JS_SCRIPT_LIST handler that resolves the user from REMOTE_USER is a silent no-op. The earlier fix did exactly this and never worked: config page (a real session request) showed Russian, JS bundle stayed English. You cannot identify the logged-in user inside js.php.

**Working fix (usersettings/action.php) — carry the language on the js.php URL, two hooks:**
1. **`TPL_METAHEADER_OUTPUT`** (BEFORE, normal authenticated page request): resolve the user's stored lang, find the `$event->data['script']` entry whose `src` contains `lib/exe/js.php`, and append `&uslang=<code>`. Bonus: the differing URL makes the browser cache the bundle per-language, so a stale English bundle isn't reused after a switch. Caveat: `applyUserLang` (ACTION_ACT_PREPROCESS) has already overwritten `$conf['lang']` by then, so capture the pre-override site default to compare against (don't compare the stored pref to the now-mutated `$conf['lang']`).
2. **`JS_SCRIPT_LIST`** (BEFORE, the js.php request): read `$INPUT->str('uslang')` (survives NOSESSION), validate to a real `inc/lang/<code>` dir (param is now user-controllable → regex `[a-z0-9-]` + `is_dir`), set `$conf['lang']` + `init_lang()`, AND rewrite the `inc/lang/<lang>/jquery.ui.datepicker.js` entry — js.php keys its output cache on `md5(serialize($files))` and that datepicker path is the only language-dependent member; without rewriting it two languages collide on one cached server bundle. js.php skips non-existent files, so the rewrite is safe even when a language ships no datepicker translation.

After deploying, touch `conf/local.php` to bust the cached bundle (see [[js-cachebuster]]). Verify with `Invoke-WebRequest .../lib/exe/js.php?t=<tpl>&uslang=ru` (no cookie needed — lang rides the URL).

This affects ANY plugin exposing `$lang['js']` strings; the usersettings fix covers them all centrally. See [[usersettings-api]].
