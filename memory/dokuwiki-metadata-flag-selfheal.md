---
name: dokuwiki-metadata-flag-selfheal
description: Syntax-plugin metadata flags should be written to non-persistent (current) meta so they self-clear when the markup token is removed
metadata: 
  node_type: memory
  type: reference
---

A syntax plugin that flags a page (e.g. a marker token an action hook later reads, like nosecedit's `~~NOSECTIONEDIT~~`) should set the flag in the renderer's **current** metadata, NOT persistent:

```php
public function render($format, Doku_Renderer $renderer, $data) {
    if ($format === 'metadata') {
        $renderer->meta['plugin']['nosecedit'] = true; // current, non-persistent
    }
    return true;
}
```

Why: `Doku_Renderer_metadata::document_start()` does `$this->meta = $this->persistent` (inc/parser/metadata.php) — at the start of every metadata render, **current is rebuilt from the persistent baseline**, while persistent carries over untouched. So:
- Write to current → if the token is later removed, the next render rebuilds current without it → flag auto-clears. Self-healing.
- Write to persistent (`p_set_metadata(...,true,true)`) → flag STICKS forever even after the token is removed; you then need ugly reset bookkeeping (nosecedit's old code used a constructor that called `p_set_metadata($ID,['sectionedit'=>'on'])` on *every* parse to undo it — which polluted ~every page's `.meta`).

Note: "non-persistent" only means reset-at-render; current IS still written to the `.meta` file by `p_save_metadata`, so an action hook reading `p_get_metadata($ID,'plugin name',false)` (no render) still sees it.

Do the write inside `render()` gated on `$format==='metadata'` (NOT in the constructor, NOT via `p_set_metadata` during xhtml render). All syntax plugins are instantiated on every parse ([[token-efficiency]] aside: `p_get_parsermodes` loads them all), so constructor side-effects = disk I/O on every page.

Migration gotcha: changing the metadata key means existing flagged pages need a re-render to gain the new key; plugin-code edits do NOT bust the metadata render cache (deps = inc/parser/metadata.php + config files), so touch conf/local.php to force it ([[js-cachebuster]]). Old persistent keys written by a previous version stay orphaned (harmless) — persistent meta is never auto-removed.
