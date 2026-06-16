---
name: xss-comment-breakout-and-validator-gaps
description: "More syntax-plugin XSS variants: HTML-comment breakout, missing $-anchor / dead validators, lessphp accidental sanitizer"
metadata:
  node_type: memory
  type: reference
---

Variants of the attr-injection XSS class ([[syntax-attr-injection-xss]]) seen across a
third-party plugin audit. Plugin names and payloads are omitted (responsible disclosure); the
reusable part is the *shape*. Beyond the plain "token → attribute unescaped" case, watch for:

1. **HTML-comment breakout.** A plugin emits user text inside `<!-- ... -->`. Browsers end a
   comment on `-->` **and on `--!>`**. So a regex/strip that only guards `-->` is bypassed by
   `--!>`. Seen with a non-greedy `<!--.*?-->` lexer (stops at the first `-->`, but a crafted
   `--!>` mid-comment breaks out) and with raw user data dropped straight into a comment. Fix:
   strip/encode `--` sequences or `hsc()` — never emit user data in a comment as a trust
   boundary.

2. **Validator missing the `$` end-anchor.** A `_isValid()` regex anchored only at `^` passes
   anything with a valid *prefix*, so a value shaped like `<valid-prefix><breakout>` slips
   through and escapes a `style="..."` / attribute context. Always anchor **both** ends
   (`^...$`); also reject `*` / `?` / `\d*` quantifiers that can match the empty string. (A
   sibling plugin from the same author family, whose validator *was* anchored `^...$`, was
   safe — the anchor is the whole difference.)

3. **Dead validator.** A plugin *defines* `_isValid()` / a style-checker but never calls it,
   relying on something else — so the value reaches output unvalidated. One such case was
   directly injectable (vulnerable); another was saved only by an accidental sanitizer: its
   sole filter ran the value through lesserphp (`spin(<color>,0)`), and **lesserphp rejects any
   value containing `"` `;` `}` `<` etc.** (parse error → returns false), so it was *not*
   exploitable despite the dead validator. Fragile luck, not a reportable defence.

4. **Safe because core escapes.** Pushing user data into `$event->data['meta']`
   (TPL_METAHEADER_OUTPUT) is safe — core `buildAttributes()`-escapes meta attrs. JSON-LD via
   `json_encode(..., JSON_PRETTY_PRINT)` WITHOUT `JSON_UNESCAPED_SLASHES` is safe — default
   `/`→`\/` makes `</script>` impossible. Adding `JSON_UNESCAPED_SLASHES` would break that (see
   [[jsinfo-embed-xss]]).
