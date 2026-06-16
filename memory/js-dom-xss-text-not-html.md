---
name: js-dom-xss-text-not-html
description: "DOM XSS: injecting untrusted AJAX data via jQuery .html()/innerHTML executes markup — use .text()/textContent"
metadata:
  node_type: memory
  type: reference
---

DOM XSS in plugin JS: injecting untrusted AJAX response data with jQuery `.html()` (or
`innerHTML`) executes any markup it contains. Use `.text()` / `textContent` for anything
that is not trusted HTML.

Found in **move** (e5e0783): `script/rename.js` and `script/progress.js` wrote AJAX error
messages into the DOM via `.html()` → a server-echoed message carrying markup became XSS.
Fix: `.text()`.

When reviewing plugin JS, grep for `.html(`, `.append(`/`.prepend(`/`.before(`/`.after(`
with string args, `innerHTML =`, `insertAdjacentHTML`, `document.write(`, and `eval(`, and
trace whether the argument carries server- or user-derived data (an AJAX response field is
the prime suspect). Remember the Grep tool is .gitignore-blind on plugin dirs — sweep with
PowerShell `Select-String` ([[grep-gitignore-blind]]). See [[syntax-attr-injection-xss]],
[[jsinfo-embed-xss]].
