---
name: dokuwiki-rest-of-page-gating
description: How to make a ~~MARKER~~ switch the rest of a page into a custom protected render mode (NOTOC-style gating)
metadata: 
  node_type: memory
  type: reference
---

To make a bare `~~MARKER~~` (like `~~NOTOC~~`) activate custom rendering for **everything from the marker to end of page**, register a greedy special pattern:

```php
$this->Lexer->addSpecialPattern('~~MARKER~~[\s\S]*', $mode, 'plugin_<name>');
```

Why it works:
- DokuWiki's lexer compiles its combined regex with the **`s` (DOTALL) flag on** (`inc/Parsing/Lexer/ParallelRegex.php` `getPerlMatchingFlags()` → `msS`/`msSi`), and matches **leftmost**. So content *above* the marker parses as normal wiki; at the marker our pattern wins and `[\s\S]*` consumes to EOF as **one** SPECIAL token.
- A special (one-shot) match means **no inner re-parsing** — DokuWiki never touches the captured text, so `**`/`//`/`[[ ]]`/bare-URLs inside are left literal. Ideal when you must render a foreign markup verbatim (e.g. paste-back fidelity).

`handle()` gets the whole match; strip the marker (`substr($match, strlen('~~MARKER~~'))`) and one leading newline, stash raw, render in `render()`. `getType()` `'substition'`, `getPType()` `'block'`.

Contrast: `~~NOTOC~~`/`~~NOCACHE~~` are just tokens whose handler flips a renderer flag (`handler.php` → `$renderer->notoc()`); core has **no** native "enable other syntax modes only on flagged pages." Per-tag gating via a page flag is not natively supported — use this region approach instead.

First used: unit3dbbcode plugin (`~~BBCODE~~`, UNIT3D parser port). See [[plugin-inventory]].
