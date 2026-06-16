# AI Can Be Used for Good — A Fact-Based Reply

*A response to the criticism in the forum thread ["Generative AI usage policy and general LLM discussion"](https://forum.dokuwiki.org/d/26031-generative-ai-usage-policy-and-general-llm-discussion/). Written by me (Nick / TrackerUser). I wrote it carefully and checked the numbers, so it's longer than a forum post — links to every source are at the bottom. Disagree with any of it and I'll happily be corrected.*

---

## What I am — and am not — arguing

Let me be precise, because the thread kept sliding between very different things.

I am **not** defending "vibe coding" — letting a model emit software nobody reads, understands, or intends to maintain. Andi's distinction between *vibe coding* and *agentic engineering* is a good one, and I sit firmly on the engineering end: I read and review every line, I debug and test on a live instance, I run a security checklist, and I keep the artifacts public. The README of the repo I shared says it plainly:

> AI tooling is a force multiplier on top of your own understanding, not a substitute for it.

I am also **not** asking the core team to accept AI-assisted pull requests. Andi maintains DokuWiki's quality bar and that is entirely his call. If he and CosmoCode don't want such PRs, that's legitimate, and I said so in the thread.

What I **am** arguing is narrower and, I think, harder to dismiss:

1. The environmental case against an individual using AI ("burning down forests, draining the water") does not survive contact with the published numbers.
2. The "AI output can't be copyrighted, so it can't be GPL" claim is a real legal question that has already been answered — and the answer is not what was stated.
3. For the **plugin repository specifically** — which nobody reviews or approves — reviewed, tested, AI-assisted work is a net improvement over the actual baseline, not a degradation of it.

I'll take these in order, then say where I agree.

---

## 1. The environmental claims

The strongest version in the thread was that every use of AI "requires … burning down forests, draining the water that is rightfully needed by rural or native peoples" and that "there is no positive way to use AI." (ryan-chappelle, post #14.) These are moral-sounding absolutes, so they deserve actual figures rather than vibes.

### What one query actually costs

The most rigorous public measurement to date is Google's, published in August 2025 with a documented methodology that includes the *full* serving stack — accelerators, host CPU/RAM, idle capacity, and data-center overhead:

| Source (method) | Energy / prompt | Water / prompt | CO₂e / prompt |
|---|---|---|---|
| **Google Gemini**, median text prompt (measured, Aug 2025) | **0.24 Wh** | **0.26 mL** (≈ 5 drops) | **0.03 g** |
| **OpenAI / Altman**, "average" ChatGPT query (stated, Jun 2025; not peer-reviewed) | ~0.34 Wh | ~0.32 mL | — |
| **UC Riverside**, GPT-3 era (estimated, 2023; the figure critics usually cite) | — | ~10–50 mL (500 mL per 10–50 replies) | — |

The honest range per query is therefore roughly **0.26 mL to ~50 mL of water** and **~0.24–0.34 Wh of energy**, depending on model generation, region, and whether you count only on-site cooling or the whole electricity supply chain. The newer the measurement, the lower the number — Google reports a **33× drop in energy and 44× drop in carbon per prompt in a single year**. Efficiency is improving fast, not getting worse.

### Putting that in scale

- **0.24 Wh** is about **nine seconds of television**, or roughly **one web search**. A 10 W LED bulb running for an hour (10 Wh) is ~40 prompts; a single hour of HD video streaming (tens of Wh) is several hundred.
- **0.26 mL** is **five drops**. Even at the old high-end estimate (~25 mL), a 500 mL bottle of water covers ~20 prompts. For comparison, the embedded water in **one almond is ~3 litres**, a **cup of coffee ~140 litres** bean-to-cup, a **hamburger ~1,700+ litres**, and a **cotton T-shirt ~2,700 litres**. A few drops per prompt is not what's draining anyone's aquifer.
- GPT-3's **~700,000 L** training figure sounds enormous in isolation, but it's a *one-time* cost amortized across billions of later queries, and it's about a quarter of one Olympic pool.

### The aggregate concern is real — and it's not about you

I won't pretend there's nothing here. The IEA estimates data centres used **~415 TWh in 2024 (~1.5 % of global electricity)** and could reach **~945 TWh by 2030 (~3 %)**, with the US and China making up ~80 % of that growth. That's a genuine grid, siting, and water-stewardship issue worth pushing providers and regulators on.

But notice what it is: an **aggregate infrastructure and policy** problem. It is not moved by one hobbyist deciding whether to modernize a DokuWiki plugin. The data centres are built to a demand curve that a single contributor's abstention doesn't bend. If you care about the footprint, the levers are clean-energy procurement, cooling efficiency (PUE/WUE), and honest reporting — exactly the things the Google paper measures — not gatekeeping an individual.

### The comparison is never "AI vs. nothing"

The implicit math in "AI wastes energy" assumes the alternative is free. It isn't. A developer workstation draws ~150–300 W *the whole time it's on*. A task that takes one person 20 hours by hand is **3–6 kWh** of workstation power alone. The same task done in 2–3 hours with AI assistance is roughly **0.5–1 kWh** at the desk, plus inference — and even a heavy multi-thousand-prompt session is well under a kilowatt-hour at the figures above. Done responsibly, the assisted path can use **less** total energy than the slow manual one, because human hours are themselves powered.

"Burning down forests" is rhetoric. Electricity comes from a grid mix, not from setting forests alight, and the per-use carbon (~0.03 g) is less than the email you'd send to complain about it.

---

## 2. "AI output can't be copyrighted, so it can't inherit the GPL"

This was raised twice (ryan-chappelle, post #14; echoed by sudokuwiki). It's a serious-sounding legal claim, and it's worth getting right rather than hand-waving — but as stated it's wrong.

The U.S. Copyright Office addressed exactly this in **Part 2 of its 2025 AI report (29 January 2025)**. Its position:

- A work **entirely** generated by AI, with no human creative input, is **not** copyrightable.
- A work where AI is used as an **assistive tool** and a human contributes sufficient creative expression **is** copyrightable — assessed case by case.
- Bare **prompt selection**, by itself, isn't enough.

Reviewing, editing, restructuring, integrating, and testing code — choosing the architecture, fixing what the model got wrong, deciding what ships — is far more than "selecting a prompt." That's the human authorship the Office describes, and it's protectable. When I **modernize an existing GPL plugin**, the situation is even simpler: the original is already GPL, my edits are a derivative work, and the GPL carries straight through. There is no copyright vacuum that voids the license.

It's also not globally settled the way the claim implies — jurisdictions differ (the UK has long had a provision for computer-generated works; Chinese courts have granted protection to some AI-assisted outputs). So "AI code is automatically public domain and a GPL violation" is not a fact you can build a ban on. The reviewed, human-directed work this thread is actually about is the *most* defensible case for authorship, not the weakest.

---

## 3. "AI can't understand, it only makes slop"

pop (post #11) argues current models have no real knowledge or understanding, so assuming they "reliably produce usable software is highly naive." sascha-leib (post #9) warns about technical debt and "ticking time bombs."

On the philosophy, I partly agree and think it's beside the point. A **compiler** doesn't "understand" your program either; we judge it by whether its output is correct. The right test for software is **verifiable**: does it run, is it secure, is it tested, is someone accountable for it? The "understanding" is supplied by the human in the loop — which is the whole point of the engineering end of the spectrum.

And on the empirical question of whether this produces slop, I have concrete, checkable evidence the other way. Working this way, I:

- built **five plugins** from scratch and **modernized sixteen-plus** abandoned ones for PHP 8.3 and current DokuWiki ("Librarian");
- added **German, Russian, and Japanese** translations to each, which widens access rather than narrowing it;
- and found and **privately reported XSS vulnerabilities in fifteen third-party plugins** — fixing my own and disclosing the rest to their authors before publishing anything.

That last point is the one I'd ask critics to sit with. **Slop doesn't find cross-site-scripting bugs in other people's code — careful, AI-assisted review does.** Those fifteen vulnerabilities were written by humans, by hand, and shipped into the same repository everyone trusts. The method under attack here is the one that *caught* them. The guide and memory I used to do all of this are public in the repo this document lives in — that's the worked example, open for anyone to judge the output directly.

I fully concede sascha-leib's underlying point: unsupervised generation *does* create debt and time bombs. That's an argument against vibe coding. It isn't an argument against the reviewed, tested workflow, and it isn't evidence that "there is no positive way to use AI."

---

## 4. The point that actually matters: the plugin repo is already unmoderated

Here's where I think the discussion has the altitude wrong. The DokuWiki plugin repository is a **wiki**. Anyone can add or edit a plugin page. There is no code review, no approval gate, and the project says so itself, on every install screen:

> Plugins are authored by developers not directly related to the DokuWiki project — they may be inexperienced, have malicious intent or may host the plugin source code on a server that has been compromised.

So the repository **already** runs on caveat emptor, and it already contains the consequences: abandoned, broken-on-Librarian, and — as I found fifteen times over — outright **vulnerable** plugins, all human-written. Someone has been looking for a CodeMirror maintainer for **six years**. Plugins throw PHP errors on the current stable release and no one updates them.

Against *that* baseline — not against some imaginary pristine hand-coded ideal — reviewed, tested, AI-assisted maintenance is an improvement. The honest question isn't "AI slop vs. perfect human code." It's "a modernized, security-reviewed plugin vs. the broken one that's been sitting there since 2020." Banning by *tool* doesn't raise quality; **verification** raises quality, and verification is tool-agnostic.

---

## 5. Where I agree, and a proportionate proposal

I'm not arguing for a free-for-all. Several things in this thread are right:

- **Human-to-human communication should be human** (Andi, post #6). Forum posts, issues, and PR descriptions should be in your own voice. I wrote my posts myself, and I agree completely.
- **Low-effort, unverified submissions are a real burden** (the matplotlib policy sudokuwiki cited; the kind of AI-generated bug-report noise that has frustrated maintainers like curl's). The fix the matplotlib maintainers chose is "a human in the loop who can demonstrate understanding of the changes" — which is exactly the bar I'm describing, not a blanket ban.
- **Disclosure is reasonable** (gry, post #16; sudokuwiki, post #15). I support a simple **"AI-assisted" tag on plugin pages and in the extension manager**, and a filter for those who want to avoid them. Treat the AI like a named collaborator: note that it helped, note the model, let the admin make an informed choice. That respects the skeptics without locking anyone out.
- **The core repository is the maintainers' call.** I'm not contesting it.

What I'd push back on is jumping from those reasonable measures to "no positive use exists" and "ban the tool." That throws away the translations, the modernizations, and the fifteen security fixes along with the slop.

---

## In closing

AI is a tool. Used carelessly it produces debt; used with a human who reads, tests, and stands behind the result, it produces real, checkable value — and the evidence is public in this repo for anyone to inspect. The environmental objection, measured rather than asserted, comes out to a few drops of water and seconds of TV per query against everyday footprints that dwarf it. The copyright objection is already answered by the Copyright Office in favor of human-directed work. And for the unmoderated plugin ecosystem in particular, the realistic alternative to a reviewed AI-assisted plugin is not a better human one — it's the abandoned, sometimes-vulnerable one that's already there.

Disclose it, tag it, let admins choose — and judge the output, not the toolbox.

— Nick (TrackerUser)

---

## Sources

- Google, *Measuring the environmental impact of delivering AI at Google Scale* (Aug 2025): [arXiv:2508.15734](https://arxiv.org/abs/2508.15734) · [Google Cloud blog summary](https://cloud.google.com/blog/products/infrastructure/measuring-the-environmental-impact-of-ai-inference/) — 0.24 Wh, 0.26 mL, 0.03 g CO₂e per median Gemini text prompt; 33×/44× year-over-year efficiency gains.
- OpenAI / Sam Altman, *The Gentle Singularity* (Jun 2025), as reported by [Data Center Dynamics](https://www.datacenterdynamics.com/en/news/sam-altman-chatgpt-queries-consume-034-watt-hours-of-electricity-and-0000085-gallons-of-water/) — ~0.34 Wh and ~0.000085 gal (~0.32 mL) per average query (not peer-reviewed).
- Li, Yang & Islam, *Making AI Less "Thirsty"* (UC Riverside, 2023): [arXiv:2304.03271](https://arxiv.org/abs/2304.03271) · [UCR News](https://news.ucr.edu/articles/2023/04/28/ai-programs-consume-large-volumes-scarce-water) — the older, higher-end water estimates (~500 mL per 10–50 GPT-3 responses; ~700,000 L on-site for GPT-3 training).
- IEA, *Energy and AI* (2025), via [S&P Global](https://www.spglobal.com/energy/en/news-research/latest-news/electric-power/041025-global-data-center-power-demand-to-double-by-2030-on-ai-surge-iea) and [IEA executive summary](https://www.iea.org/reports/energy-and-ai/executive-summary) — data centres ~415 TWh in 2024 (~1.5 %), projected ~945 TWh by 2030 (~3 %).
- U.S. Copyright Office, *Copyright and Artificial Intelligence, Part 2: Copyrightability* (29 Jan 2025), summarized by [Jones Day](https://www.jonesday.com/en/insights/2025/02/copyrightability-of-ai-outputs-us-copyright-office-analyzes-human-authorship-requirement) and [Skadden](https://www.skadden.com/insights/publications/2025/02/copyright-office-publishes-report) — AI-assisted works with sufficient human authorship are copyrightable; purely AI-generated output is not.
