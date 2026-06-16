---
name: syntax-attr-injection-xss
description: "Dominant stored-XSS class in our syntax-plugin reviews: user token from markup emitted into an HTML attribute unescaped"
metadata: 
  node_type: memory
  type: reference
---

The most common stored-XSS we have found in DokuWiki syntax plugins: a user-supplied
token taken from wiki markup and written into an HTML attribute (`class` / `style` /
`lang` / `dir` / `id` / `title`) without charset validation or escaping. An editor
writes markup like `<wrap :en"onmouseover="alert(1)>` and the `"` breaks out of the
attribute, injecting a live event handler that fires in **every reader's** browser
(admins included). Editors are not trusted for this — treat all markup as hostile.

Found and fixed in our reviews:
- **wrap** (9a47fc3): `getAttributes()` used `trim()` on the lang/id token, keeping the
  raw `en"onmouseover="alert(1)`. Fix: extract via a regex **capture group**
  (`$matches[1]`, charset `[A-Za-z0-9_-]`) so only the safe portion survives;
  `buildAttributes()` also wraps `lang`/`xml:lang`/`dir` in `hsc()` as defence-in-depth.
- **searchtablejs** (da8885d): tag attributes emitted unescaped in `render()` ENTER →
  `<searchtable " onXX=...>`. Fix: `hsc()` the attributes.
- **sortablejs** (a4d02cd): bareword-option regex `r?\d*` matched the **empty string**,
  so an unrecognised token passed through verbatim into the class attribute. Fix:
  anchor + require a digit (`r?\d+`), and `hsc()` the class suffix.
- **color** (d00d764) / **cellbg** (e0d3a3b): colour spec → `style` attribute. Fix:
  validate against a charset that can't break out of a single-quoted `style='...'`
  (reject quote, `<`, `>`, `&`, `;`), `hsc()` the value. Leave actual colour validity to
  the browser (forward-compat with new CSS colour syntaxes). color also swapped
  `$renderer->_xmlEntities($match)` → `hsc($match)`.

**Two-layer rule:** (1) validate the token with a strict regex **capture group** — never
`trim()`, and never a quantifier that can match empty (`*`, `?`, `\d*`) which lets junk
through; (2) `hsc()` at output as belt-and-suspenders.

**Counter-lesson (prosemirror 02085f0):** escaping is not always right. `hsc()` on an RSS
URL attr double-encoded `&`→`&amp;` on every WYSIWYG parse↔render round-trip, corrupting
saved wikitext. In round-trip/editor contexts escape at the HTML boundary only, never in
data that gets written back to markup.

When reviewing a syntax plugin, scan its `syntax*.php` for attribute emission (`class=`,
`style=`, `lang=`, `<a href`, `title=`) built from `handle()`-parsed tokens without
`hsc()`/charset validation. See [[jsinfo-embed-xss]], [[js-dom-xss-text-not-html]],
[[csrf-vs-acl-distinction]].
