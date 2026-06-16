# DokuWiki Plugin Review Guide

## Environment

- **DokuWiki:** Librarian 2025-05-14b, Debian 13, PHP 8.3
- **Docker:** `http://dokuwiki.local/` — container `dokuwiki-docker-dokuwiki-1`
- **Plugins:** this directory (`storage/lib/plugins/`)
- **DokuWiki source (read-only reference):** `F:\Projects\dokuwiki-stable` — read individual files by absolute path when needed, do not index the full tree
- **PHP lint** (run in PowerShell): `docker exec dokuwiki-docker-dokuwiki-1 php -l /storage/lib/plugins/<name>/<file>.php`
- **Shell:** run `docker exec` and any command containing container/Unix paths (`/storage/...`, `/var/www/html/...`, `/tmp/...`) in **PowerShell**, never the Bash tool — Bash is Git Bash, and MSYS rewrites `/storage/...` → `C:/Program Files/Git/storage/...`, so the command fails. Use `2>$null` (not `2>/dev/null`). Container DokuWiki root is `/var/www/html`; `/storage` holds only the bind-mounted host dirs (`/storage/inc`, `/dokuwiki/` don't exist).
- **Superuser:** `@admin` group. Auth: authplain, ACL enabled.
- **Browser floor:** Firefox 78 ESR

## Workflow

- **Review-first:** When asked to review or plan, produce analysis and an implementation plan only. Do not edit files or generate code unless explicitly told to proceed with execution.
- **Verify against source:** Don't assume DokuWiki APIs. Check actual source at `F:\Projects\dokuwiki-stable\` for hooks, classes, functions.
- **Frontend testing:** Test at `http://dokuwiki.local/` (login: `admin` / `admin`) when behavior verification is needed.

## Git & Identity

- Commits use global defaults
- Line endings: LF (`\n`) in all files.

## Plugin Layout

```
<name>/
├── plugin.info.txt           # manifest (required)
├── syntax.php                # wiki syntax (optional)
├── action.php                # event hooks, AJAX, UI injection
├── admin.php                 # admin panel
├── helper.php                # pure logic/API reused by other plugins
├── renderer.php              # custom/export output format
├── remote.php                # XML-RPC/JSON-RPC API methods
├── auth.php                  # authentication backend
├── cli.php                   # `bin/plugin.php <name>` command
├── <type>/<comp>.php         # multi-component: many classes of one type
├── script.js / scripts/*.js  # auto-bundled globally (use sparingly!)
├── style.css / all.css / print.css / *.less  # auto-bundled CSS (screen/all/print/feed)
├── <name>.svg / admin.svg    # menu icon: SVG, <2 KB, single path
├── conf/default.php          # config defaults ($conf[...])
├── conf/metadata.php         # config metadata ($meta[...]; no require_once!)
├── lang/<iso>/lang.php       # UI strings ($lang[...]; no wiki syntax)
├── lang/<iso>/settings.php   # config-manager labels
├── lang/<iso>/<file>.txt     # localized wiki-markup text (locale_xhtml)
└── deleted.files             # optional: files to delete on update
```

**Class & file naming:** single component → `<type>.php` defines `<type>_plugin_<name>` (e.g. `admin.php` → `admin_plugin_acl`). Several components of one type → `<type>/<comp>.php` → `<type>_plugin_<name>_<comp>` (e.g. `syntax/code.php` → `syntax_plugin_code_code`; its syntax mode is then `plugin_<name>_<comp>`). Plugin name = `a-z0-9` only, **no underscore** (separates name from component; an underscore also zeroes the popularity rating).

Base classes (all in `\dokuwiki\Extension\`, legacy `DokuWiki_*_Plugin` aliases still work): `SyntaxPlugin`, `ActionPlugin`, `AdminPlugin`, `Plugin` (base — helpers extend this directly; no `HelperPlugin` exists), `RemotePlugin`, `AuthPlugin`, `CLIPlugin`, `EventHandler`. Renderers extend `Doku_Renderer` (`inc/parser/renderer.php`).

Common events: `TPL_METAHEADER_OUTPUT`, `ACTION_ACT_PREPROCESS`, `AJAX_CALL_UNKNOWN`, `HTML_EDITFORM_OUTPUT`, `MENU_ITEMS_ASSEMBLY`, `TPL_ACT_UNKNOWN`, `DOKUWIKI_STARTED`.

## PHP Conventions

### Mandatory

- **DOKU_INC guard:** `if (!defined('DOKU_INC')) die();` on every non-namespaced PHP file except `conf/` and `lang/` (pure data files — DokuWiki core omits the guard on these)
- **Explicit visibility:** all methods/properties: `public`, `protected`, or `private`. DokuWiki style prefers `protected` over `private` so subclasses/other plugins can override
- **No PHP closing tag:** omit `?>` in all files (prevents stray output). Official code style is **PSR-12**
- **Type hints:** respect the PHP floor; be conservative on public/overridable signatures (a changed hint breaks inheriting plugins) — lean on docblocks
- **Docblocks:** every public/protected method with `@param`, `@return`, purpose
- **Input:** `$INPUT->post->str()`, `->int()`, `->arr()`, `$INPUT->get->str()`. Never `$_REQUEST`/`$_POST`/`$_GET`
- **ID sanitization:** `cleanID($raw)` before passing page IDs to logic
- **File writes:** `io_saveFile()` is already atomic (locks internally). For read-modify-write, hold your own `io_lock()`/`io_unlock()` around the full sequence
- **JSON storage:** `JSON_PRETTY_PRINT | JSON_UNESCAPED_UNICODE`

### Style

- `[]` not `array()`. `$val ?? 'fallback'`. No `@` suppression.
- `in_array($val, $arr, true)` — always strict.
- `str_starts_with()` or `strncmp()` over `substr()` for prefixes.
- No leftover `var_dump`, `print_r`, `error_log`.

### Permissions Pattern

Helpers are pure logic — take facts as parameters, never read globals directly. Action layer gathers facts (`$INPUT->server->str('REMOTE_USER')`, `auth_isadmin()`) and calls helper.

## JS Conventions (Firefox 78 ESR)

**FORBIDDEN:** `#privateField`, `??=`/`||=`/`&&=`, `structuredClone()`, `.at()`, `Object.hasOwn()`, native `<dialog>`/`showModal()`, `.findLast()`/`.findLastIndex()`

**FORBIDDEN CSS:** `:has()`, complex `:not()`, `aspect-ratio`, `@container`, CSS nesting

**ALLOWED:** `async`/`await`, `fetch()`, ES6 classes, `?.`, `??`, `Map`/`Set`, `IntersectionObserver`, template literals, destructuring, spread, arrow functions, `Promise.allSettled()`

**Style:** Vanilla JS, no framework. AJAX via `fetch()` with `JSINFO.sectok`. Gate plugin JS on conditions via `JSINFO`. jQuery available as `jQuery` (not `$`).

## CSS Conventions

- DokuWiki token variables: `__background__`, `__text__`, `__border__`, `__link__`, etc.
- Prefix classes/IDs with plugin name: `.annotations-panel`, `#usersettings-form`.
- Hard-code colors only for semantic UI (highlights, status indicators).

## DokuWiki API (dev-manual reference)

Condensed from <https://www.dokuwiki.org/development>. Verify specifics against `dokuwiki-stable` source. Code autoloads via PSR-4; no `require_once`.

### Plugin types & required methods

All types inherit the common methods below; types are not exclusive (one folder may hold several). `isSingleton()` → `false` forces a fresh instance per load (renderers default to false).

| Type | File / class | Must implement |
|------|--------------|----------------|
| syntax | `syntax_plugin_<name>` | `getType`, `getSort`, `connectTo`, `handle`, `render` |
| action | `action_plugin_<name>` | `register(EventHandler $c)` + handler methods |
| admin | `admin_plugin_<name>` | `handle`, `html` (opt: `forAdminOnly`→false lets managers in, `getMenuText/Sort/Icon`, `getTOC`) |
| helper | `helper_plugin_<name>` | `getMethods` + the public methods it lists |
| renderer | `renderer_plugin_<name>` | `getFormat` (+ `canRender` for an xhtml replacement) |
| remote | `remote_plugin_<name>` | public methods (auto-exposed as `plugin.<name>.<method>`) |
| auth | `auth_plugin_<name>` | ctor sets `$this->cando[...]` + `$this->success`; `checkPass` **or** `trustExternal`; `getUserData` |
| cli | `cli_plugin_<name>` | `setup(Options)`, `main(Options)` — runs via `bin/plugin.php <name>` |

**Common methods:** `getConf($k)`, `getLang($k)`, `locale_xhtml($file)`, `getInfo()`, `getPluginName/Type/Component()`, `loadHelper($name, $msg=true)`. Load others via `plugin_load($type,$name)` guarded by `plugin_isdisabled($name)`; check result `instanceof PluginInterface`. Avoid `render()` (parsing wiki text) — high overhead.

### Syntax plugins

- **getType** (mode type): `container`, `baseonly`, `formatting`, `substition` (sic — keep the typo), `protected`, `disabled`, `paragraphs`. `getAllowedTypes()` lists types allowed to nest inside.
- **getPType:** `normal` (inline, default) / `block` / `stack`. **getSort:** lexer test order; lowest wins on overlap (lets you override core syntax).
- **Patterns** (in `connectTo`/`postConnect`): `addSpecialPattern` (one-shot), `addEntryPattern`+`addExitPattern` (paired), `addPattern` (internal). No delimiters; `(?:…)` for alternation; no back-refs; inline flags only `(?i)`; non-greedy `*?`; anchor tags with `\b`; prefix tag names with `<name>_`.
- **handle($match,$state,$pos,Doku_Handler $h):** do heavy work here — its return is **cached**; pass data to render only via that array. `$state` ∈ `DOKU_LEXER_ENTER|MATCHED|EXIT|SPECIAL|UNMATCHED`.
- **render($format,Doku_Renderer $r,$data):** gate on `$format=='xhtml'`; append to `$r->doc`; escape via `hsc()`/`$r->_xmlEntities()`. Handle `$format=='metadata'` and register links with `$r->internallink()` so backlinks work.

### Events

- `$controller->register_hook($name, 'BEFORE'|'AFTER', $this, 'method', $param=null, $seq=0)`. Handler `method(Event $e, $param)` must be **public**.
- **Seq ranges:** −3999…−1 progressively earlier, `0` default, 1…3999 progressively later; avoid `±PHP_INT_MAX`.
- `$e->data` read/write; `$e->preventDefault()` (BEFORE only) skips the default action; `$e->stopPropagation()` skips later handlers (not the default). Fire your own: `Event::createAndTrigger($name,$data,$action=null,$canPrevent=true)`.
- `use dokuwiki\Extension\{ActionPlugin,EventHandler,Event};`. Load external libs in the ctor or on demand, **never at file top**.

### Config metadata (`conf/metadata.php`)

`$meta['k'] = ['<class>', '_param'=>…];` Classes: `string`, `numeric`/`numericopt` (`_min`/`_max`), `onoff` (defaults must be int `0`/`1`, not bool), `multichoice` (`_choices`), `multicheckbox` (`_choices`,`_other`), `email`, `password` (`_code`), `dirchoice` (`_dir`), `array`, `regex` (`_delimiter`,`_pregflags`), `fieldset` (group, key prefixed `_`), `authtype`. `_pattern` validates most. Labels → `lang/<iso>/settings.php`.

### Security (see also XSS in checklist)

- **Escape:** `hsc()` for HTML, `rawurlencode()` for URL values; whitelist class/attribute values with a safe default; block `javascript:` (`preg_match('/^https?:\/\//i',$url)`).
- **CSRF:** every state-changing form/link needs a token — the `Form` class adds it automatically; hand-built ones use `formSecurityToken()`/`getSecurityToken()`, then **you must call `checkSecurityToken()`** before acting.
- **ACL:** gate page access with `auth_quickaclcheck($id)` vs `AUTH_NONE/READ/EDIT/CREATE/UPLOAD/DELETE/ADMIN`. Remote plugins throw `AccessDeniedException`. Treat all input as hostile, even from admins.

### Forms

Use `dokuwiki\Form\Form` (since Elenor; old `Doku_Form` deprecated): `$f=new Form(); $f->addTextInput($name,$label); $f->addDropdown/addCheckbox/addPasswordInput/setHiddenField(...); echo $f->toHTML();` — security token auto-included; modifiable by other plugins via form events.

### Localization

`getLang('k')` (+`sprintf` for `%s`); `locale_xhtml('file')` for markup text. `lang.php` strings carry **no** wiki syntax. JS strings: `$lang['js']['k']` → `LANG.plugins.<name>.k`. Users override via `conf/plugin_lang/<name>/<iso>/`. ISO-639-1 dir names, UTF-8, missing keys fall back to English.

### Caching & metadata

- Don't disable caching wholesale — add page dependencies via the `PARSER_CACHE_USE` event (or store state in metadata). `~~NOCACHE~~` forces per-request re-render.
- `p_get_metadata($id,$key,$render)` / `p_set_metadata($id,$data,$render,$persistent)`; space-separate nested keys (`'relation references'`). Flags: `METADATA_DONT_RENDER`, `METADATA_RENDER_USING_CACHE` (default), `_SIMPLE_CACHE`, `_UNLIMITED`. Store plugin data under a `plugin.<name>` key; adjust during render via `PARSER_METADATA_RENDER`. `ft_backlinks($id)` lists backlinks.

### Front-end (DokuWiki platform; FF78 floor is separate, above)

- **JS:** all bundles load `defer` (since Hogfather) — no `document.write` that depends on core. Include extra files with `/* DOKUWIKI:include[_once] file.js */`. Inline code via `tpl_inlineScript('…')` (adds the CSP nonce) — never hand-write `<script>`. jQuery is **only** `jQuery()`, never `$()`; prefix jQuery vars with `$`; old shims (`$()`,`getElementsByClass`,`addEvent`,`tw_sack`) are gone. Inject `JSINFO` keys from an action plugin on `DOKUWIKI_STARTED` (under a `plugin_<name>` sub-key).
- **CSS/LESS:** LESS parses in `.css` and `.less` (lesserphp — no JS exprs, stricter CSS). `style.ini` placeholders → `@ini_background` etc. RTL via `[dir=rtl] .sel {…}` (not `rtl.css`). Use `<name>__id` (double underscore) for IDs to dodge section-header clashes. `url()`/`@import` resolve relative to the template; `@import` only works from `all.css`. Plugin CSS loads on every page even when the plugin isn't used.
- **Toolbar:** action plugin on `TOOLBAR_DEFINE`/AFTER pushes a button array (`type`: `format`/`picker`/`mediapopup` or custom `addBtnAction<Type>`); icons in `lib/images/toolbar/`.
- **Menus:** `\dokuwiki\Menu\{Site,User,Page,Detail,Mobile}Menu`; add items via `MENU_ITEMS_ASSEMBLY`. Items extend `AbstractItem`; `getSvg()` icon < 2 KB, single path. Legacy `tpl_action*()` helpers deprecated.

### Environment & debugging

- **Globals:** `$ID`, `$INFO`, `$ACT` (may be an **array** — guard with `!is_array()`), `$conf`, `$lang`, `$INPUT`, `$USERINFO`, `$auth`, `$REV`, `$TEXT`, `$JSINFO`. User id via `$INPUT->server->str('REMOTE_USER')`.
- **Constants:** `DOKU_INC`, `DOKU_PLUGIN`, `DOKU_BASE`/`DOKU_REL`/`DOKU_URL`, `DOKU_CONF`; `DOKU_TPL*` deprecated → `tpl_basedir()`/`tpl_incdir()`.
- **Debug:** never leave `error_log()` — DokuWiki logs via `Logger::debug($head,$msg)` (LogViewer, `data/log/`); `dbglog()` deprecated. Mark removed APIs with `dbg_deprecated()`. Enable with `allowdebug` + `?do=debug`; `?do=check` for env/permission info.

## Review Process

"Review a plugin" means all of the following:
1. Check for bugs, logic errors, security issues
2. Modernize for PHP 8.3 (types, null safety, deprecated functions)
3. Verify Librarian compatibility (namespace changes, API correctness — check actual DokuWiki source)
4. Adapt JS/CSS for Firefox 78 ESR floor
5. Test on live DokuWiki instance when behavior verification is needed
6. Apply the checklist below
7. Check/add translations (de, ru, ja)
8. Update `plugin.info.txt` per conventions below
9. Update `README.md` — document changes made; if README contradicts code, fix it (code takes priority)
10. Save any tests written during review to `_test/` in DokuWiki PHPUnit format (not a priority — only save if tests were needed during review)

### Checklist

1. **Fatal errors on Librarian:** `class_exists()` probes for namespace changes
2. **DOKU_INC guards** on every non-namespaced `.php` except `conf/` and `lang/` (pure data)
3. **Visibility keywords** on all methods
4. **Docblocks** on public/protected methods
5. **Input handling:** no `$_REQUEST`/`$_POST`/`$_GET`
6. **No `@` suppression**
7. **FF78 JS/CSS:** scan for forbidden patterns (strip comments first)
8. **Global JS/CSS bloat:** `script.js` at root = bundled everywhere. Use `TPL_METAHEADER_OUTPUT` for conditional loading
9. **File writes:** `io_saveFile()` for simple writes (already atomic). `io_lock()`/`io_unlock()` wrapping read-modify-write cycles. Never wrap `io_saveFile()` in extra lock/unlock — it deadlocks
10. **XSS:** user output through `hsc()`
11. **Null safety:** unguarded array access on user input
12. **`conf/metadata.php`:** no `require_once`
13. **Debug leftovers:** no `var_dump`/`print_r`/`error_log`
14. **TODO/FIXME:** flag for resolution
15. **`plugin.info.txt`:** date/version current
16. **CSRF:** state-changing forms/actions verify `checkSecurityToken()` (or use the `Form` class)
17. **URL output:** `rawurlencode()`; user-supplied URLs reject non-`http(s)` (block `javascript:`)
18. **No PHP `?>` closing tag;** prefer `protected` over `private`

## plugin.info.txt

**Modernized plugins** (existing repos you adapted — i.e. with commits from authors other than you):
- Year → `2077`, keep original month/day
- Add your name/handle to the `author` field if not already present
- No comments in this file

**Custom plugins** (built from scratch by you):
- `date` → today's date (YYYY-MM-DD)
- Add `phpmin X.Y` if plugin requires a minimum PHP version
- No comments in this file

## Translations

Provide `de`, `ru`, `ja` translations for all reviewed or custom plugins.
- Polite, natural style in each language
- If translations already exist, verify all English strings are covered — add missing ones
- Extract any hardcoded user/admin-facing strings to `lang/en/lang.php` first, then translate

## Tests

When tests are written during review/development, save them in `<plugin>/_test/` using DokuWiki's standard PHPUnit format. Tests are a byproduct of review work, not the goal — don't spend usage writing tests that weren't needed.

### Conventions

- Namespace: `\dokuwiki\plugin\<name>\test\`
- Extend `DokuWikiTest`, annotate `@group plugin_<name>` and `@group plugins`
- File naming: `*Test.php` (PSR-4, preferred) or `*.test.php` (legacy)
- Enable plugin: `protected $pluginsEnabled = ['<name>'];`
- Include a `GeneralTest.php` — validates `plugin.info.txt` and `conf/default.php` ↔ `conf/metadata.php` key sync (template: `move/_test/GeneralTest.php`)
- Config overrides in `setUp()`: `$conf['plugin']['<name>']['option'] = value;` (not in `setUpBeforeClass`)
- No `require_once` needed — DokuWiki autoloader handles everything
- Integration tests: `TestRequest` to fake HTTP requests, `TestResponse::queryHTML()` for DOM inspection

### Running

Plugins are symlinked from `storage/lib/plugins/` into `dokuwiki-stable/lib/plugins/`. Run from host:

```
cd F:\Projects\dokuwiki-stable\_test
composer run test -- --group plugin_<name>
```

Note the `--` separator before `--group` (required by composer).

## Common Pitfalls

- **`script.js` auto-bundling:** file at plugin root gets inlined into global JS for ALL pages. Rename and inject conditionally via `TPL_METAHEADER_OUTPUT`.
- **`style.less` auto-loading:** same for CSS. Feature-specific CSS should be injected conditionally.
- **`DOKUWIKI:include`:** processed at JS build time, concatenated into global bundle.
- **usersettings first-registration-wins:** duplicate keys silently ignored.
- **`class_exists()` with autoload:** default triggers autoload. Use `class_exists($class, false)` only to skip it explicitly.
- **Event priorities:** default 0. Negative (e.g., `-10`) runs before other handlers.
- **`metaFN()`:** `metaFN($id, '.myplugin')` gives a path in `data/meta/` namespaced to the page.
