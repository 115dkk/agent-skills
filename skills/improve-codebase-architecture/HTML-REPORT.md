# HTML Report Format

This file is the review report's format. Its delivery, diagram patterns, style and tone sections are shared with [FOUNDATION-REPORT.md](FOUNDATION-REPORT.md) and [STACK-REPORT.md](STACK-REPORT.md), which replace only the report's sections.

The architectural review is one self-contained page. Mermaid handles graph-shaped diagrams reliably; hand-built visuals (divs, inline SVG, or React components) handle the more editorial ones (mass diagrams, cross-sections). Mix the two — don't lean on Mermaid for everything, it'll start to look generic.

There are two ways to deliver it, and they differ only in the outer shell and in how scripts load.

## Delivery A: Artifact page (default when the session can publish one)

The Artifact tool wraps the file in its own document skeleton, enforces a script allowlist, renders in the viewer's light or dark theme, and is often read on a phone. So:

- Write page content only: no `<!doctype>`, `<html>`, `<head>` or `<body>` tags. Start the file with `<title>` (a two-to-four-word name such as `ClaudeRipple architecture review`, never "X: explainer"), the tool's viewport-sides meta if you also use its side-gutter paddings, then `<style>`.
- **Mermaid renders natively** in Artifact pages through `<pre class="mermaid">` blocks. Do not import Mermaid.
- Every other script names an exact version from an allowed CDN, placed before the inline script that uses it: `https://cdn.tailwindcss.com/3.4.17` for Tailwind's play CDN; `https://cdnjs.cloudflare.com/ajax/libs/react/18.3.1/umd/react.production.min.js` and `.../react-dom/18.3.1/umd/react-dom.production.min.js` for React; `https://unpkg.com/htm@3.1.1/dist/htm.umd.js` to write React without a build step. Never a bare name, a range or `@latest`.
- Colours are tokens on `:root`, redefined for dark under `@media (prefers-color-scheme: dark)` guarded by `:root:not([data-theme="light"])` and again under `:root[data-theme="dark"]`. `body` sets an explicit background from a token. Mermaid themes itself; hand-built diagrams take their strokes and fills from the tokens.
- At phone width the before/after pair stacks (before above after), the page never scrolls sideways, and a wide Mermaid graph sits in its own `overflow-x: auto` container.
- Follow the tool's design guidance for everything else. Its rules win over the style notes below where they disagree.

```html
<title>{{Repo}} architecture review</title>
<meta name="claude:viewport-fit-sides" content="cover">
<style>
  /* Layout: one column of candidate cards; before/after side by side above 720px, stacked below. */
  :root { --bg: …; --fg: …; --muted: …; --accent: …; --leak: …; --warn: …; --line: …; }
  @media (prefers-color-scheme: dark) { :root:not([data-theme="light"]) { --bg: …; --fg: …; /* … */ color-scheme: dark } }
  :root[data-theme="dark"] { --bg: …; --fg: …; /* … */ color-scheme: dark }
  body { background: var(--bg); color: var(--fg); }
  main { padding-left: max(16px, env(safe-area-inset-left, 0px)); padding-right: max(16px, env(safe-area-inset-right, 0px)); padding-block: 40px; }
  .pair { display: grid; gap: 16px; grid-template-columns: 1fr; }
  @media (min-width: 720px) { .pair { grid-template-columns: 1fr 1fr; } }
  .scroll { overflow-x: auto; }
  .seam { stroke-dasharray: 4 4; }
</style>
<main>
  <header>…</header>
  <section id="candidates">…</section>
  <section id="top-recommendation">…</section>
</main>
```

## Delivery B: local file (no publisher)

A complete document in the OS temp directory, opened in the user's browser. Same content and diagrams; Mermaid has to be imported here.

```html
<!doctype html>
<html lang="{{user's language}}">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>{{Repo}} architecture review</title>
    <script src="https://cdn.tailwindcss.com/3.4.17"></script>
    <script type="module">
      import mermaid from "https://cdn.jsdelivr.net/npm/mermaid@11.4.1/dist/mermaid.esm.min.mjs";
      mermaid.initialize({ startOnLoad: true, theme: "neutral", securityLevel: "loose" });
    </script>
    <style>
      .seam { stroke-dasharray: 4 4; }
      .leak { stroke: #dc2626; }
    </style>
  </head>
  <body>
    <main>…</main>
  </body>
</html>
```

## Header

Repo name, date, and a compact legend: solid box = module, dashed line = seam, red arrow = leakage, thick dark box = deep module. When the report is not in English, the legend also says that the glossary terms (module, interface, depth, deep, shallow, seam, adapter, leverage, locality) stay in English on purpose. No introduction paragraph — straight into the candidates.

## Candidate card

The diagrams carry the weight. Prose is sparse, plain, and uses the glossary terms ([LANGUAGE.md](LANGUAGE.md)) without ceremony.

Each candidate is one `<article>`:

- **Title** — short, names the deepening (e.g. "Collapse the Order intake pipeline").
- **Badge row** — recommendation strength (`Strong` = emerald, `Worth exploring` = amber, `Speculative` = slate), plus a tag for the dependency category (`in-process`, `local-substitutable`, `ports & adapters`, `mock`).
- **Files** — monospaced list.
- **Before / After diagram** — the centrepiece. Two columns on a wide screen, stacked on a phone. See patterns below.
- **Problem** — one sentence. What hurts.
- **Solution** — one sentence. What changes.
- **Wins** — bullets, ≤6 words each (in English; a little longer in languages that need more words for the same idea). e.g. "Tests hit one interface", "Pricing logic stops leaking", "Delete 4 shallow wrappers".
- **ADR callout** (if applicable) — one line in an amber-tinted box. Where the project keeps its decisions somewhere other than `docs/adr/` (an ARCHITECTURE file, a list of failure modes that must not come back), cite that.

No paragraphs of explanation. If the diagram needs a paragraph to be understood, redraw the diagram.

## Diagram patterns

Pick the pattern that fits the candidate. Mix them. Don't make every diagram look the same — variety is part of the point.

### Mermaid graph (the workhorse for dependencies / call flow)

Use a Mermaid `flowchart` or `graph` when the point is "X calls Y calls Z, and look at the mess." Wrap it in a styled card so it doesn't feel parachuted in. Style with classDef to colour leakage edges red and the deep module dark. Sequence diagrams work well for "before: 6 round-trips; after: 1."

```html
<div class="card scroll">
  <pre class="mermaid">
    flowchart LR
      A[OrderHandler] --> B[OrderValidator]
      B --> C[OrderRepo]
      C -.leak.-> D[PricingClient]
      classDef leak stroke:#dc2626,stroke-width:2px;
      class C,D leak
  </pre>
</div>
```

### Hand-built boxes-and-arrows (when Mermaid's layout fights you)

Modules as boxes with borders and labels; arrows as inline SVG `<line>` or `<path>` elements over a relative container. Reach for this when you want the "after" diagram to feel like one thick-bordered deep module with greyed-out internals — Mermaid won't render that with the right weight.

### React primitives (when the same visuals repeat)

When several candidates draw the same things, write them once as small components (`Module`, `Seam`, `Leak`, `MassBar`, `BeforeAfter`) and compose each diagram from them. Use React 18 UMD plus `htm` (pinned, see above) so there is no build step, mount into one root, and keep the content (candidate data) as a plain array at the top of the script. Interaction stays small: a before/after toggle on a phone, a hover that highlights a seam. Everything meant to be read is visible without interaction.

### Cross-section (good for layered shallowness)

Stack horizontal bands to show the modules a call passes through. Before: 6 thin bands each doing nothing. After: 1 thick band labelled with the consolidated responsibility.

### Mass diagram (good for "interface as wide as implementation")

Two rectangles per module — one for interface surface area, one for implementation. Before: interface rectangle is nearly as tall as the implementation rectangle (shallow). After: interface rectangle is short, implementation rectangle is tall (deep).

### Call-graph collapse

Before: a tree of function calls rendered as nested boxes. After: the same tree collapsed into one box, with the now-internal calls shown faded inside it.

## Style guidance

- Lean editorial, not corporate-dashboard. Generous whitespace. Choose the palette and type for the subject; avoid the stock looks (warm cream with a terracotta accent, a purple-to-blue gradient, `rounded-lg` and a shadow on every block).
- Colour sparingly: one accent plus red for leakage and amber for warnings, all as tokens so both themes read.
- Keep diagrams ~320px tall so a before/after pair sits side by side on a wide screen without scrolling.
- Use small uppercase labels with a little letter-spacing for module labels inside diagrams — they should read as schematic, not as UI.
- Scripts are limited to Tailwind, React and htm when the report uses them (and Mermaid's import in Delivery B). No app code beyond the diagrams.

## Top recommendation section

One larger card. Candidate name, one sentence on why, anchor link to its card. That's it.

## Tone

Plain, concise — but the architectural nouns and verbs come straight from [LANGUAGE.md](LANGUAGE.md). Concision is not an excuse to drift.

**Use exactly:** module, interface, implementation, depth, deep, shallow, seam, adapter, leverage, locality.

**Never substitute:** component, service, unit (for module) · API, signature (for interface) · boundary (for seam) · layer, wrapper (for module, when you mean module).

**Phrasings that fit the style:**

- "Order intake module is shallow — interface nearly matches the implementation."
- "Pricing leaks across the seam."
- "Deepen: one interface, one place to test."
- "Two adapters justify the seam: HTTP in prod, in-memory in tests."

**Wins bullets** name the gain in glossary terms: *"locality: bugs concentrate in one module"*, *"leverage: one interface, N call sites"*, *"interface shrinks; implementation absorbs the wrappers"*. Don't write *"easier to maintain"* or *"cleaner code"* — those terms aren't in the glossary and don't earn their place.

No hedging, no throat-clearing, no "it's worth noting that…". If a sentence could be a bullet, make it a bullet. If a bullet could be cut, cut it. If a term isn't in [LANGUAGE.md](LANGUAGE.md), reach for one that is before inventing a new one.

In a report written in another language, the same discipline applies to that language's prose: plain sentences, no translated idioms, and the glossary terms left in English.
