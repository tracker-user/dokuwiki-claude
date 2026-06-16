---
name: xss-review-batch-2026-06
description: "Method for an XSS-only sweep across many third-party DokuWiki plugins, and the finding-classes it surfaces"
metadata:
  node_type: memory
  type: reference
---

Notes from an XSS-only audit pass over a large batch of downloaded third-party DokuWiki
plugins (June 2026). Specific plugin names and proof-of-concept payloads have been removed
for responsible-disclosure reasons — what's kept is the *method* and the recurring *classes
of bug*, which is the reusable part.

**Method.** PowerShell `Select-String` sweeps (the built-in Grep tool is .gitignore-blind on
plugin dirs — see [[grep-gitignore-blind]]) for the usual sinks:
- syntax plugins that emit a handled token with no `hsc()`,
- attribute concatenation into `class` / `style` / `title` / `href`,
- raw `$renderer->doc .= $var`,
- JS DOM sinks (`.html()`, `innerHTML` — see [[js-dom-xss-text-not-html]]),
- standalone scripts that bootstrap `inc/init.php` themselves (see [[xss-standalone-init-endpoints]]).

Then trace `handle()` → `render()` on every hit to confirm reachability and whether the value
is editor- or request-controlled.

**Recurring finding-classes (stored, editor-triggered, hits every reader):**
- id / content / label / colour tokens written into an attribute unescaped;
- HTML-comment breakout, validators missing an end-anchor, and dead (defined-but-uncalled)
  validators — see [[xss-comment-breakout-and-validator-gaps]];
- `<area coords>` and `<svg>` / title-attribute breakouts;
- unescaped external feed / remote content (sometimes with an SSRF angle).

**Recurring finding-classes (unauthenticated reflected):** standalone PHP endpoints that echo
request input with no ACL / CSRF — see [[xss-standalone-init-endpoints]].

**Frequently cleared as safe:** plugins whose action layer strictly validates attributes,
those that route IDs through `cleanID()`, those building attributes via the core
`buildAttributes()` helper, and those relying on core meta-escaping. One accidental-sanitizer
case worth knowing: pushing a colour through lesserphp (`spin()`) rejects quotes/semicolons as
a parse error, so a value that *looks* unvalidated can still be non-exploitable — fragile luck,
not a defence to rely on.

**Discipline:** report confirmed issues to each plugin's author privately and give them time to
fix before any public mention. Tactics: [[syntax-attr-injection-xss]],
[[xss-comment-breakout-and-validator-gaps]], [[js-dom-xss-text-not-html]].
