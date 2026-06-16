---
name: css-var-rgba-lesserphp
description: "rgba(var(--x), a) in plugin CSS compiles to opaque black via lesserphp; LESS-escape it"
metadata: 
  node_type: memory
  type: reference
---

DokuWiki compiles ALL plugin CSS (`.css` and `.less`) through lesserphp via `lib/exe/css.php`. lesserphp evaluates `rgba()` at compile time. If an rgba() contains a CSS custom property — `rgba(var(--ann-open-rgb), 0.35)` — lesserphp reads `var()` as 0 and bakes the colour to an opaque `#000000` (borders came out `#000100`). Symptom: highlight/background renders **black** even though the injected `:root{--x:…}` variables are correct (inline `<style>` is raw, not compiled, so vars look fine in page source).

Fix: LESS-escape the whole declaration so lesserphp passes it through verbatim:
`background-color: ~"rgba(var(--ann-open-rgb), 0.35)";`
The browser then resolves the custom property at render time. A bare `var(--x)` as a full value (not inside rgba()) is fine unescaped — lesserphp only mangles it inside a colour function it recognises.

Verify via the compiled output, not the source: `Invoke-WebRequest http://dokuwiki.local/lib/exe/css.php?t=dokuwiki` and grep the rule. Applies to any plugin doing config-driven colours through CSS variables (annotations does). See [[js-cachebuster]] (touch conf/local.php / the .css mtime change busts the css.php cache).
