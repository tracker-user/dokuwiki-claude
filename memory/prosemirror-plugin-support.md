---
name: prosemirror-plugin-support
description: How to add ProseMirror WYSIWYG support to other DokuWiki plugins; guide location and the two-path rule
metadata: 
  node_type: memory
  type: reference
---

To make a DokuWiki syntax plugin render in the ProseMirror editor (instead of showing raw syntax), follow `prosemirror/PROSEMIRROR_PLUGIN_SUPPORT.md` (in the plugin's repo, linked from its README).

Key non-obvious finding (verified against source, not upstream docs):

- **Path A (public-API bridge, no core changes):** only works for *self-contained* nodes (block/inline atoms — include, gallery, RSS-like). Uses `PROSEMIRROR_RENDER_PLUGIN` + `PROSEMIRROR_PARSE_UNKNOWN` events and the `window.Prosemirror.*` JS API.
- **Path B (must edit prosemirror's own files):** required for inline wrappers/marks-with-attributes (**typography** `<fc>`) and attributes on built-in nodes (**cellbg** `@color:`). The public API cannot express these: `renderer.php::$marks` and `clearBlock()` are `protected`, `cdata()` builds marks without attrs, and `NodeStack` has no parent accessor. Recipes touch `renderer.php`, `parser/Mark.php`, `schema/NodeStack.php`, `schema.js`, `parser/TableCellNode.php`.

Editing the prosemirror fork's core is fine — it's a local fork the user already modifies. Commit plugin changes from inside the plugin's own git repo ([[feedback-plugin-git-commit]]).
