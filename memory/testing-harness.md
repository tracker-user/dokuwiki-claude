---
name: testing-harness
description: DokuWiki plugin test setup — PHPUnit via symlinked plugins
metadata: 
  node_type: memory
  type: reference
---

Tests use DokuWiki's standard PHPUnit infrastructure. Plugin directories are symlinked from `storage/lib/plugins/` into `dokuwiki-stable/lib/plugins/` so PHPUnit can discover them.

Run from host:
```
cd F:\Projects\dokuwiki-stable\_test
composer run test -- --group plugin_<name>
```

Test format: `<plugin>/_test/`, extend `DokuWikiTest`, namespace `\dokuwiki\plugin\<name>\test\`, annotate `@group plugin_<name>` + `@group plugins`. See CLAUDE.md Tests section for full conventions.

Integration tests available via `TestRequest`/`TestResponse` for faking HTTP requests and inspecting rendered HTML with `queryHTML()`.

Verified working: move plugin (44 tests, 202 assertions through symlinks).

**How to apply:** Reference when writing or running tests. Not needed for review-only sessions.
