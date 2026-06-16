# Adding Plugin Support to ProseMirror

This guide explains how to make a DokuWiki syntax plugin render properly in the
ProseMirror WYSIWYG editor instead of showing its raw syntax as text.

**Note:** The prosemirror plugin's extension API is in ALPHA state and may change.
This is a **local fork**, so modifying the prosemirror plugin's own files is an
acceptable and sometimes necessary approach (see Path B).

**Plugins with existing upstream Prosemirror support** (reference implementations):
ImgPaste, Diagrams, CatMenu, VisualIndex.

---

## Two Integration Paths

There are two fundamentally different ways to integrate, and **which one you need
is dictated by the plugin's behavior, not by preference.**

### Path A — Public-API bridge (upgrade-safe, no core changes)

Works entirely through the published event + JS API. All the code lives **inside
the target plugin**; the prosemirror plugin's files are untouched. Use this when
the plugin produces **self-contained output** — a block or inline atom that
replaces its syntax (include, gallery, RSS-like substitutions, info boxes, etc.).

### Path B — Core modifications (edit prosemirror's own files)

Required when the plugin's output cannot be expressed as a self-contained node.
Two important cases fall here:

- **Inline wrappers / marks with attributes** (e.g. **typography** `<fc #f00>…</fc>`):
  these are ProseMirror *marks*, and a mark that carries an attribute (the color)
  cannot be produced through the public API — the renderer's `$marks` array is
  `protected` and `cdata()` builds marks **without** attributes. You must edit
  `renderer.php` and `parser/Mark.php`.

- **Attributes on an existing node** (e.g. **cellbg** `@color:` colouring a table
  cell): the renderer offers no public way to reach the *parent* cell node, and the
  table cell's schema attributes are defined inside `schema.js`. You must edit
  `schema/NodeStack.php`, `schema.js`, and `parser/TableCellNode.php`.

### Routing table

| Plugin behaviour | ProseMirror primitive | Path |
|------------------|-----------------------|------|
| Standalone block replaced by rendered output (include, gallery) | block node (`atom`) | A |
| Standalone inline token | inline node (`atom`) | A |
| Wraps inline text with formatting, **no** attributes | mark | B (renderer `$marks` is protected) |
| Wraps inline text with formatting, **with** attributes (typography) | mark + attrs | B |
| Adds/changes an attribute on a built-in node (cellbg → table cell) | node attribute | B |

> Even an attribute-less wrapper needs Path B, because there is no public setter
> for the renderer's mark stack. Path A genuinely covers only self-contained nodes.

---

## Architecture Overview

### Data flow

```
Wiki Syntax  --(PHP renderer)-->  ProseMirror JSON  --(JS schema)-->  Editor DOM
Editor DOM   --(JS state)------>  ProseMirror JSON  --(PHP parser)-->  Wiki Syntax
```

1. **Wiki syntax → JSON (PHP).** DokuWiki parses text into instructions; the
   prosemirror renderer (`renderer.php`) builds a tree of `schema\Node` objects and
   serializes to JSON. For syntax plugins, `plugin()` fires the
   `PROSEMIRROR_RENDER_PLUGIN` event, then otherwise wraps the raw match in a generic
   `dwplugin_inline` / `dwplugin_block` node (shown as `<code>` — this is the "raw
   syntax as text" you currently see).

2. **JSON → editor (JS).** `script/schema.js` defines the allowed node/mark types.
   NodeViews customise DOM rendering. Schema, nodeviews and menu items are extensible
   via `window.Prosemirror.*` hooks.

3. **JSON → wiki syntax (PHP).** `parser\Node::getSubNode()` maps JSON node types to
   PHP parser classes. Unknown types fire `PROSEMIRROR_PARSE_UNKNOWN`. The generic
   `dwplugin_*` types are handled by `PluginNode`, which emits the text verbatim.

### Extension points

**PHP events:**

| Event | Fires in | Purpose |
|-------|----------|---------|
| `PROSEMIRROR_RENDER_PLUGIN` | `renderer.php::plugin()` | Build JSON for your syntax |
| `PROSEMIRROR_PARSE_UNKNOWN` | `parser/Node.php::getSubNode()` | Rebuild syntax from your JSON node |

`PROSEMIRROR_RENDER_PLUGIN` data:

```php
[
    'name'     => string,   // plugin/component name, e.g. 'typography_fontcolor', 'cellbg'
    'data'     => mixed,    // whatever your syntax plugin's handle() returned
    'state'    => int,      // DOKU_LEXER_ENTER | _EXIT | _SPECIAL | _MATCHED ...
    'match'    => string,   // the raw matched text
    'renderer' => renderer_plugin_prosemirror,
]
```

`PROSEMIRROR_PARSE_UNKNOWN` data:

```php
[
    'node'     => array,      // JSON node: type, attrs, content, marks
    'parent'   => parser\Node,
    'previous' => parser\Node | null,
    'newNode'  => null,       // YOU set this to your parser\Node instance
]
```

In both handlers you must call `$event->preventDefault()` and (for parsing) assign
`$event->data['newNode']`.

**JS global API** (`window.Prosemirror`, ready after `PROSEMIRROR_API_INITIALIZED`):

| Property | Type | Purpose |
|----------|------|---------|
| `pluginSchemas` | `Function[]` | Each receives `(nodes, marks)` OrderedMaps, returns `{nodes, marks}` |
| `pluginNodeViews` | `Object` | Map of node-type name → nodeview constructor |
| `pluginMenuItemDispatchers` | `Array` | Dispatchers added **inside the "Plugins" (puzzle) dropdown** |
| `classes.KeyValueForm` | Class | Reusable jQuery-UI form dialog for editing attributes |
| `classes.MenuItem` | Class | Menu-item constructor |
| `classes.AbstractMenuItemDispatcher` | Class | Base for menu-item dispatchers |
| `classes.DOMParser` | Class | ProseMirror DOMParser (clipboard paste) |
| `commands.setBlockTypeNoAttrCheck` | Function | Block-type command skipping attr comparison |

`window.AbstractNodeView` is also exposed (as a side-effect global, not under
`classes`) for nodeviews to extend.

**JS event:** `PROSEMIRROR_API_INITIALIZED` (jQuery, on `document`) fires after the
API object exists but before the editor is built. Register all JS extensions here.

### Timing

`prosemirror/script.js` is auto-bundled globally and includes `lib/bundle.js`. Order:

1. `bundle.js` runs `initializePublicAPI()` → creates `window.Prosemirror.*`, triggers
   `PROSEMIRROR_API_INITIALIZED`.
2. Other plugins' `script.js` files run.
3. On editor open, `enableProsemirror()` calls `getSpec()`, which runs every
   `pluginSchemas` callback.
4. `getNodeViews()` spreads `pluginNodeViews`.
5. `MenuInitializer.collectMenuItems()` folds `pluginMenuItemDispatchers` into the
   Plugins dropdown.

> **Your plugin's `script.js` is shipped to the browser untranspiled** (only
> prosemirror's own `bundle.js` goes through Babel). It must satisfy the Firefox 78
> floor — but ES6 that FF78 supports (`const`/`let`, arrow functions, classes,
> `?.`, `??`, `Map`/`Set`, template literals) is fine. See the JS Compatibility
> section.

---

## OrderedMap API

The `nodes` and `marks` arguments to schema callbacks are **OrderedMap** instances
(persistent/immutable — every method returns a *new* map), not plain objects.

| Method | Description |
|--------|-------------|
| `get(key)` | Value by key, or `undefined` |
| `update(key, value, newKey?)` | Replace/add a binding; optionally rename |
| `remove(key)` | New map without the key |
| `addToStart(key, value)` | Insert at the beginning |
| `addToEnd(key, value)` | Append at the end |
| `addBefore(place, key, value)` | Insert before `place` (appends if not found) |
| `forEach(fn)` | Iterate `fn(key, value)` in order |
| `append(map)` / `prepend(map)` | Merge non-overlapping keys after/before |
| `subtract(map)` | Remove keys present in `map` |
| `size` | Entry count |

Typical usage:

```javascript
window.Prosemirror.pluginSchemas.push(function (nodes, marks) {
    // add a new node type
    nodes = nodes.addToEnd('mynode', { /* NodeSpec */ });

    // modify an existing type (must re-assign — maps are immutable)
    var cell = nodes.get('table_cell');
    cell.attrs.background = { default: null };
    nodes = nodes.update('table_cell', cell);

    // add a mark
    marks = marks.addToEnd('mymark', { /* MarkSpec */ });

    return { nodes: nodes, marks: marks };
});
```

---

## NodeSpec / MarkSpec reference

### NodeSpec

| Property | Type | Description |
|----------|------|-------------|
| `content` | string | Content expression: `"text*"`, `"inline*"`, `"paragraph+"`, `"(a \| b)+"`, `"x{2,5}"` |
| `marks` | string | `"_"` = all (default), `""` = none, `"strong em"` = specific |
| `group` | string | Group name usable in content expressions |
| `inline` | boolean | Inline vs block |
| `atom` | boolean | Treated as a single, non-text-editable unit |
| `code` | boolean | Content is code (no marks, monospace) |
| `defining` | boolean | Type preserved when content is replaced |
| `isolating` | boolean | Cursor can't cross the boundary with arrows |
| `draggable` | boolean | Node can be dragged |
| `attrs` | object | `{ name: { default: value } }`; no `default` ⇒ required |
| `toDOM` | `fn(node)` | `["tag", {attr: val}, 0]`; `0` = content hole; omit for leaf/atom |
| `parseDOM` | array | `[{ tag, style, getAttrs, priority }]` |

### MarkSpec

| Property | Type | Description |
|----------|------|-------------|
| `attrs` | object | As NodeSpec |
| `inclusive` | boolean | Default `true`; set `false` for link-like marks that shouldn't extend |
| `excludes` | string | Space-separated incompatible marks; `"_"` = excludes all others |
| `group` | string | Mark group |
| `toDOM` | `fn(mark)` | `["span", {style:"…"}, 0]` |
| `parseDOM` | array | `[{ tag/style, getAttrs }]` |

### parseDOM rule keys

| Key | Meaning |
|-----|---------|
| `tag` | CSS selector (`"span.x"`, `"div[data-y]"`) |
| `style` | CSS property name (`"color"`) |
| `getAttrs` | `fn(dom)` → attrs object, or `false` to reject, or `null` to match with no attrs |
| `priority` | default 50; higher checked first; use 60+ to beat built-in rules |

---

## Path A — Public-API bridge

End-to-end example: a standalone block plugin with syntax `{{mything>param}}` that
should appear in the editor as its rendered output. **No prosemirror core files are
touched.** All files below live in *your* plugin (`mything/`).

### A1. action.php — register

```php
<?php
if (!defined('DOKU_INC')) die();

use dokuwiki\Extension\ActionPlugin;
use dokuwiki\Extension\EventHandler;
use dokuwiki\Extension\Event;
use dokuwiki\plugin\prosemirror\schema\Node;

class action_plugin_mything_prosemirror extends ActionPlugin
{
    public function register(EventHandler $controller)
    {
        // No availability guard needed: if prosemirror is absent these
        // events simply never fire.
        $controller->register_hook('PROSEMIRROR_RENDER_PLUGIN', 'BEFORE', $this, 'handleRender');
        $controller->register_hook('PROSEMIRROR_PARSE_UNKNOWN', 'BEFORE', $this, 'handleParse');
    }
```

### A2. handleRender — wiki syntax → JSON

Uses **only public** renderer methods: `addToNodestack()`, `addToNodestackTop()`,
`dropFromNodeStack()`, and the public `nodestack` property. Note that
`clearBlock()` is *protected*, so the "close any open paragraph" step is replicated
inline.

```php
    public function handleRender(Event $event)
    {
        if ($event->data['name'] !== 'mything') return;

        $renderer = $event->data['renderer'];

        // Replicate the protected clearBlock(): close an open paragraph so the
        // block node lands at doc level, not inside a <p>.
        if ($renderer->nodestack->current()->getType() === 'paragraph') {
            $renderer->nodestack->drop('paragraph');
        }

        $node = new Node('mything_block');
        // Preserve the raw syntax verbatim — this is what guarantees a clean roundtrip.
        $node->attr('syntax', $event->data['match']);

        // Leaf/atom node → add() (addToNodestack), NOT addTop().
        $renderer->addToNodestack($node);

        $event->preventDefault();
    }
```

### A3. Parser class — a real named class (not an anonymous class)

`mything/MyThingBlockNode.php`, autoloaded as `dokuwiki\plugin\mything\MyThingBlockNode`:

```php
<?php
namespace dokuwiki\plugin\mything;

use dokuwiki\plugin\prosemirror\parser\Node;

class MyThingBlockNode extends Node
{
    protected $syntax;

    // Signature mirrors TableCellNode/RSSNode (Node $parent = null is accepted by getSubNode).
    public function __construct($data, Node $parent = null)
    {
        $this->syntax = $data['attrs']['syntax'] ?? '';
    }

    public function toSyntax()
    {
        return $this->syntax;
    }
}
```

### A4. handleParse — JSON → wiki syntax

```php
    public function handleParse(Event $event)
    {
        if (($event->data['node']['type'] ?? '') !== 'mything_block') return;

        $event->data['newNode'] = new \dokuwiki\plugin\mything\MyThingBlockNode(
            $event->data['node']
        );
        $event->preventDefault();
    }
}
```

> `getSubNode()` accepts your node because, after `preventDefault()`,
> `advise_before()` is false and `is_a($newNode, parser\Node::class)` is true.

### A5. script.js — schema (public API)

```javascript
jQuery(document).on('PROSEMIRROR_API_INITIALIZED', function () {
    window.Prosemirror.pluginSchemas.push(function (nodes, marks) {
        nodes = nodes.addToEnd('mything_block', {
            group: 'substitution_block',
            atom: true,
            attrs: {
                syntax: { default: '' },
                renderedHTML: { default: null },
            },
            toDOM: function (node) {
                var dom = document.createElement('div');
                dom.className = 'mything-node';
                if (node.attrs.renderedHTML) {
                    dom.innerHTML = node.attrs.renderedHTML;
                } else {
                    dom.textContent = node.attrs.syntax; // placeholder until resolved
                }
                return dom;
            },
        });
        return { nodes: nodes, marks: marks };
    });
});
```

### A6. (Optional) NodeView with server-rendered preview

To show real rendered HTML instead of a placeholder, add a nodeview that fetches it.
The built-in `plugin_prosemirror` AJAX endpoint is **not** extensible (its `switch`
ends in `default → 400`), so register **your own** `AJAX_CALL_UNKNOWN` handler.

```javascript
// in the same PROSEMIRROR_API_INITIALIZED handler
window.Prosemirror.pluginNodeViews.mything_block = function (node, view, getPos) {
    return new MyThingView(node, view, getPos);
};
```

`MyThingView` follows `RSSView`: extend `window.AbstractNodeView`, in `renderNode()`
check `attrs.renderedHTML`; if missing, `jQuery.post(DOKU_BASE + 'lib/exe/ajax.php',
{ call: 'mything_render', syntax: attrs.syntax })` and on success
`view.dispatch(view.state.tr.setNodeMarkup(getPos(), null, { ...attrs, renderedHTML }))`.

Server side (your action.php):

```php
$controller->register_hook('AJAX_CALL_UNKNOWN', 'BEFORE', $this, 'ajaxRender');
// ...
public function ajaxRender(Event $event)
{
    if ($event->data !== 'mything_render') return;
    $event->preventDefault();
    $event->stopPropagation();
    global $INPUT;
    $syntax = $INPUT->str('syntax');
    header('Content-Type: text/html; charset=utf-8');
    echo p_render('xhtml', p_get_instructions($syntax), $info);
}
```

---

## Path B — Core modifications

For cases the public API can't express. These edits go into the **prosemirror
plugin's own files** (acceptable on this fork). Each recipe is minimal and
backward-compatible.

### B1. Marks with attributes — typography (`<fc #f00>…</fc>`)

Three core files change, plus the JS schema (the schema part is still public API).

**(1) `renderer.php` — public mark setters + attribute-aware `cdata()`**

Add public entry points (the event handler can't touch the protected `$marks`):

```php
/**
 * Push a formatting mark, optionally carrying attributes, onto the active stack.
 *
 * @param string $type  mark type (must match the JS schema mark name)
 * @param array  $attrs attributes serialised onto every text node it covers
 * @return void
 */
public function addPluginMark($type, array $attrs = [])
{
    $this->marks[$type] = $attrs ?: 1;
}

/**
 * @param string $type
 * @return void
 */
public function dropPluginMark($type)
{
    unset($this->marks[$type]);
}
```

Make `cdata()` carry attributes (replace the existing mark loop). Backward
compatible: existing `$this->marks['strong'] = 1` yields a non-array value and thus
no attributes.

```php
foreach ($this->marks as $markType => $markData) {
    $mark = new Mark($markType);
    if (is_array($markData)) {
        foreach ($markData as $key => $value) {
            $mark->attr($key, $value);
        }
    }
    $node->addMark($mark);
}
```

**(2) `parser/Mark.php` — attribute-driven syntax + tolerant ordering**

Custom marks aren't in the static `$openingMarks`/`$closingMarks`/`$markOrder`
arrays. Prefer the literal syntax stored in attrs, and make `sort()` null-safe:

```php
public function getOpeningSyntax()
{
    if (isset($this->attrs['syntax_open'])) {
        return $this->attrs['syntax_open'];
    }
    if ($this->type !== 'unformatted') {
        return self::$openingMarks[$this->type] ?? '';
    }
    return $this->getUnformattedSyntax('opening');
}

public function getClosingSyntax()
{
    if (isset($this->attrs['syntax_close'])) {
        return $this->attrs['syntax_close'];
    }
    if ($this->type !== 'unformatted') {
        return self::$closingMarks[$this->type] ?? '';
    }
    return $this->getUnformattedSyntax('closing');
}
```

In `sort()`, replace the two `self::$markOrder[...]` reads with
`(self::$markOrder[...] ?? 50)` so an unregistered mark type doesn't warn.

**(3) typography action handler** (`typography/action.php` or a new bridge action)

The closing tag for typography is deterministic, so both syntaxes are known at ENTER:

```php
public function handleRender(Event $event)
{
    if ($event->data['name'] !== 'typography_fontcolor') return;

    $renderer = $event->data['renderer'];
    $state    = $event->data['data'][0] ?? null;

    if ($state === DOKU_LEXER_ENTER) {
        $tagData = $event->data['data'][1] ?? [];   // CSS pairs from handle()
        $renderer->addPluginMark('typo_fontcolor', [
            'color'        => $tagData['color'] ?? '',
            'syntax_open'  => $event->data['match'], // e.g. "<fc #ff0000>"
            'syntax_close' => '</fc>',
        ]);
    } elseif ($state === DOKU_LEXER_EXIT) {
        $renderer->dropPluginMark('typo_fontcolor');
    }

    $event->preventDefault();
}
```

> Inspect the real `handle()` output of the component to know the key
> (`typography_parser::parse_inlineCSS()` returns CSS `property => value` pairs;
> for `<fc>` that's `color`). When in doubt, store only `syntax_open`/`syntax_close`
> for the roundtrip and use the CSS pairs purely for the editor's visual style.

**(4) `script.js` — the mark in the JS schema (public API)**

```javascript
window.Prosemirror.pluginSchemas.push(function (nodes, marks) {
    marks = marks.addToEnd('typo_fontcolor', {
        attrs: {
            color:        { default: '' },
            syntax_open:  { default: '<fc>' },
            syntax_close: { default: '</fc>' },
        },
        toDOM: function (mark) {
            return ['span', { style: 'color:' + mark.attrs.color }, 0];
        },
        parseDOM: [{
            tag: 'span[style]',
            getAttrs: function (dom) {
                var c = dom.style.color;
                return c ? { color: c } : false;
            },
        }],
    });
    return { nodes: nodes, marks: marks };
});
```

Roundtrip: the mark's `syntax_open`/`syntax_close` attrs ride along on every covered
text node in the JSON; on parse, `Mark::getOpeningSyntax()/getClosingSyntax()` read
them back verbatim.

### B2. Attribute on a built-in node — cellbg (`@color:` on a table cell)

cellbg colours the **enclosing table cell**. Four edits.

**(1) `schema/NodeStack.php` — expose the parent node**

There is currently no way to reach the node *below* the top of the stack:

```php
/**
 * Return the node directly beneath the current one (its parent), or null.
 *
 * @return Node|null
 */
public function parent()
{
    if ($this->stacklength < 1) {
        return null;
    }
    return $this->stack[$this->stacklength - 1];
}
```

**(2) cellbg action handler** — when `@color:` is rendered, the stack top is the
cell's paragraph and its parent is the cell:

```php
public function handleRender(Event $event)
{
    if ($event->data['name'] !== 'cellbg') return;

    $renderer = $event->data['renderer'];
    $color    = $event->data['data'][1] ?? 'yellow'; // handle() returns [state, color, match]

    $cell = $renderer->nodestack->parent();
    if ($cell && in_array($cell->getType(), ['table_cell', 'table_header'], true)) {
        $cell->attr('background', $color);
    }

    $event->preventDefault(); // don't emit the literal "@color:" text
}
```

**(3) `script.js` (or `schema.js`) — give cells a `background` attribute**

The cleanest place is the existing `tableNodes({ cellAttributes: { … } })` call in
`schema.js`, using prosemirror-tables' intended `setDOMAttr` hook:

```javascript
// inside tableNodes({ ... cellAttributes: { align: {…}, ... } })
background: {
    default: null,
    setDOMAttr: function (value, attrs) {
        if (value) {
            attrs.style = (attrs.style || '') + 'background-color:' + value + ';';
        }
    },
},
```

(Adding the attribute from a `pluginSchemas` callback is possible but risks clobbering
prosemirror-tables' generated `toDOM`, which also emits colspan/rowspan/colwidth — so
prefer the `cellAttributes` route here.)

**(4) `parser/TableCellNode.php` — re-emit `@color:` from the attribute**

Table layout (the `|`/`^` delimiters and column-aligned padding) is produced
centrally by `TableNode::toSyntax()` so the output matches the edittable plugin
byte-for-byte; an individual cell only supplies its **content** via
`getCellText()`. Prepend the `@color:` marker there:

```php
public function getCellText()
{
    $doc = '';
    foreach ($this->subnodes as $subnode) {
        $doc .= $subnode->toSyntax();
    }

    $content = trim(str_replace("\n", ' ', $doc));
    if (!empty($this->data['attrs']['background'])) {
        // cellbg pattern is ^@#?[0-9a-zA-Z]*: at the start of the cell
        $content = '@' . $this->data['attrs']['background'] . ':' . $content;
    }
    return $content;
}
```

---

## Adding menu items

Items pushed to `pluginMenuItemDispatchers` are folded **into the "Plugins" (puzzle)
dropdown** (`MenuInitializer.collectMenuItems()`), not placed as standalone toolbar
buttons. A dispatcher is any object with `isAvailable(schema)` and
`getMenuItem(schema)` (instance or static both work — they're duck-typed).

### Insert a node

```javascript
jQuery(document).on('PROSEMIRROR_API_INITIALIZED', function () {
    var MenuItem = window.Prosemirror.classes.MenuItem;

    window.Prosemirror.pluginMenuItemDispatchers.push({
        isAvailable: function (schema) { return !!schema.nodes.mything_block; },
        getMenuItem: function (schema) {
            var icon = document.createElement('span');
            icon.innerHTML = '<svg viewBox="0 0 24 24"><path d="…"/></svg>';
            return new MenuItem({
                command: window.Prosemirror.commands.setBlockTypeNoAttrCheck(
                    schema.nodes.mything_block,
                    { syntax: '{{mything>default}}' }
                ),
                icon: icon,
                label: 'My Thing',
            });
        },
    });
});
```

### Toggle a mark

For attribute-less toggles use prosemirror-commands' `toggleMark`. For an
attributed mark (e.g. a colour), open a `KeyValueForm` and `addMark` on submit
(`prompt()` works as a quick stand-in but isn't the established UX):

```javascript
var KeyValueForm = window.Prosemirror.classes.KeyValueForm;
var form = new KeyValueForm('Font colour', [
    { label: 'Colour', type: 'color', name: 'color', value: '#ff0000' },
]);
form.on('submit', function (e) {
    e.preventDefault();
    var attrs = form.$form.serializeArray().reduce(function (a, f) { a[f.name] = f.value; return a; }, {});
    form.hide();
    var view = window.Prosemirror.view;
    var sel = view.state.selection;
    if (!sel.empty) {
        view.dispatch(view.state.tr.addMark(
            sel.from, sel.to,
            view.state.schema.marks.typo_fontcolor.create(attrs)
        ));
    }
});
form.show();
```

---

## Keybindings and input rules

These are **not** currently extensible through the public API.

- **Keybindings** live in `script/plugins/Keymap/keymap.js`; the keymap plugin is
  constructed in `main.js` and not exposed for extension. Adding shortcuts means a
  core change.
- **Input rules** (auto-formatting as you type) live in
  `script/plugins/InputRules/inputrules.js` and likewise require editing the bundle.

Most integrations rely on menu items instead.

---

## Roundtrip safety

Editing in ProseMirror must never lose or corrupt the plugin syntax.

1. **Store the original syntax** in attrs (`syntax` for atoms; `syntax_open` /
   `syntax_close` for marks/wrappers). Reconstruct from these rather than
   regenerating from parsed parameters.
2. **Handle every lexer state** your plugin emits (ENTER/EXIT for formatting,
   SPECIAL for substitutions). A missing state drops syntax.
3. **Test the roundtrip:** open a page using the syntax in ProseMirror, switch to the
   syntax editor, and confirm the text is byte-for-byte preserved.

---

## Renderer / NodeStack API (visibility matters)

`renderer_plugin_prosemirror` (via `$event->data['renderer']`):

```php
// PUBLIC — safe from an external event handler
$renderer->nodestack                      // NodeStack (public property)
$renderer->addToNodestack(Node $node)     // add as child of current (leaf/atom)
$renderer->addToNodestackTop(Node $node)  // add as child AND push (containers)
$renderer->dropFromNodeStack($type)       // pop, verifying type
$renderer->getCurrentMarks()              // returns a COPY of the marks array

// PROTECTED — NOT accessible externally (would fatal)
$renderer->marks                          // use addPluginMark()/dropPluginMark() (Path B)
$renderer->clearBlock()                   // replicate: drop 'paragraph' if current
```

`NodeStack` (public methods): `current()`, `getDocNode()`, `doc()`, `add()`,
`addTop()`, `drop($type)`, `isEmpty()`. There is **no** parent accessor until you add
`parent()` (Path B / B2).

`schema\Node`: `new Node($type)`, `->attr($k,$v?)`, `->addChild(Node)`,
`->setText($s)` (text nodes only), `->addMark(Mark)`, `->getType()`, `->hasContent()`.
`schema\Mark`: `new Mark($type)`, `->attr($k,$v?)`.

---

## JS compatibility (Firefox 78 ESR floor)

Your `script.js` ships untranspiled, so the source itself must be FF78-safe.

**Allowed:** `const`/`let`, arrow functions, ES6 classes, template literals,
destructuring, spread, `?.`, `??`, `Map`/`Set`, `Promise.allSettled()`, `fetch()`,
`IntersectionObserver`, `async`/`await`. jQuery is available as `jQuery` (not `$`).

**Forbidden:** `#privateFields`, `??=`/`||=`/`&&=`, `structuredClone()`, `.at()`,
`Object.hasOwn()`, `.findLast()`/`.findLastIndex()`, native `<dialog>`/`showModal()`.
CSS: no `:has()`, complex `:not()`, `aspect-ratio`, `@container`, CSS nesting.

---

## Checklist

**Path A (public-API bridge):**

1. [ ] Confirm the plugin's output is self-contained (block/inline atom)
2. [ ] `action.php`: `PROSEMIRROR_RENDER_PLUGIN` handler using public renderer methods only
3. [ ] Named parser class extending `parser\Node`; `PROSEMIRROR_PARSE_UNKNOWN` handler
4. [ ] `script.js`: node via `pluginSchemas` (guard on `PROSEMIRROR_API_INITIALIZED`)
5. [ ] (Optional) nodeview + your own `AJAX_CALL_UNKNOWN` for rendered preview
6. [ ] (Optional) menu item via `pluginMenuItemDispatchers` (lands in Plugins dropdown)

**Path B (core modifications):**

1. [ ] Marks-with-attrs: `renderer.php` (`addPluginMark`/`dropPluginMark` + `cdata()`),
       `parser/Mark.php` (`syntax_open`/`syntax_close` + null-safe `$markOrder`)
2. [ ] Node attribute: `schema/NodeStack.php` (`parent()` if needed), `schema.js`
       (attribute + `setDOMAttr`), the relevant `parser/*Node.php` `toSyntax()`
3. [ ] JS schema (mark/attr) via `pluginSchemas`
4. [ ] Action handler wiring the renderer to your syntax

**Both:**

5. [ ] Roundtrip test: syntax → ProseMirror → syntax is identical
6. [ ] Editor rendering matches the published page
7. [ ] Paste test if `parseDOM` is defined
8. [ ] PHP lint: `docker exec dokuwiki-docker-dokuwiki-1 php -l /storage/lib/plugins/<name>/<file>.php`

---

## Reference: built-in node types

| Node | Group | Inline | Content | Notes |
|------|-------|--------|---------|-------|
| `doc` | – | no | `(block\|baseonly\|container\|protected_block\|substitution_block)+` | root; attrs `nocache`,`notoc` |
| `paragraph` | block | no | `inline*` | |
| `heading` | baseonly | no | `text*` | no marks |
| `text` | inline | yes | – | carries marks |
| `hard_break` | inline | yes | leaf | |
| `horizontal_rule` | block | no | leaf | `----` |
| `image` | inline | yes | leaf | internal/external media |
| `link` | inline | yes | leaf/atom | all link types |
| `footnote` | inline | yes | atom | |
| `smiley` | inline | yes | atom | |
| `code_block` | protected_block | no | `text*` | |
| `preformatted` | protected_block | no | `text*` | |
| `blockquote` | container | no | `(block\|blockquote\|protected_block)+` | |
| `bullet_list` / `ordered_list` | container | no | `list_item+` | |
| `list_item` | – | no | `(paragraph\|protected_block\|substitution_block)+ (ordered_list\|bullet_list)?` | |
| `table` | container | no | `table_row+` | |
| `table_cell` / `table_header` | – | no | `(paragraph\|protected_block\|substitution_block)+` | `align` attr |
| `rss` | substitution_block | no | atom | |
| `dwplugin_block` | protected_block | no | `text*` | generic fallback |
| `dwplugin_inline` | inline | yes | `text*` | generic fallback |

## Reference: built-in marks

| Mark | Syntax | HTML |
|------|--------|------|
| `strong` | `**…**` | `<strong>` |
| `em` | `//…//` | `<em>` |
| `code` | `''…''` | `<code>` |
| `underline` | `__…__` | `<u>` |
| `deleted` | `<del>…</del>` | `<del>` |
| `subscript` | `<sub>…</sub>` | `<sub>` |
| `superscript` | `<sup>…</sup>` | `<sup>` |
| `unformatted` | `%%…%%` | `<span class="unformatted">` |

Mark ordering for syntax output is governed by `parser/Mark.php::$markOrder`; register
new attributed marks there (or rely on the null-safe default from B1).
