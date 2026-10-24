---
name: improve-codebase-architecture
description: Find deepening opportunities in a codebase, informed by the domain language in CONTEXT.md and the decisions in docs/adr/, or lay out the first architecture of a new project and, when asked, choose its stack. Use when the user wants to improve architecture, find refactoring opportunities, consolidate tightly-coupled modules, make a codebase more testable and AI-navigable, design the initial architecture of something new, or decide which stack to build it on.
---

# Improve Codebase Architecture

Surface architectural friction and propose **deepening opportunities** — refactors that turn shallow modules into deep ones. The aim is testability and AI-navigability.

## Glossary

Use these terms exactly in every suggestion. Consistent language is the point — don't drift into "component," "service," "API," or "boundary." Full definitions in [LANGUAGE.md](LANGUAGE.md).

- **Module** — anything with an interface and an implementation (function, class, package, slice).
- **Interface** — everything a caller must know to use the module: types, invariants, error modes, ordering, config. Not just the type signature.
- **Implementation** — the code inside.
- **Depth** — leverage at the interface: a lot of behaviour behind a small interface. **Deep** = high leverage. **Shallow** = interface nearly as complex as the implementation.
- **Seam** — where an interface lives; a place behaviour can be altered without editing in place. (Use this, not "boundary.")
- **Adapter** — a concrete thing satisfying an interface at a seam.
- **Leverage** — what callers get from depth.
- **Locality** — what maintainers get from depth: change, bugs, knowledge concentrated in one place.

Key principles (see [LANGUAGE.md](LANGUAGE.md) for the full list):

- **Deletion test**: imagine deleting the module. If complexity vanishes, it was a pass-through. If complexity reappears across N callers, it was earning its keep.
- **The interface is the test surface.**
- **One adapter = hypothetical seam. Two adapters = real seam.**

This skill is _informed_ by the project's domain model. The domain language gives names to good seams; ADRs record decisions the skill should not re-litigate.

## Choose the report

Three situations, three reports. Decide which one before doing anything else, from the user's words first and the repository second.

| Situation | Signals | Report | Process |
|---|---|---|---|
| **Review**: a codebase exists and the user wants it better | "refactor", "improve the architecture", "what should we clean up"; a repo with history | [HTML-REPORT.md](HTML-REPORT.md) | the steps below |
| **Foundation**: something new is being built and its stack is already given | "set up", "design the architecture", "how should we structure"; an empty or scaffold-only repo; the user names the language or framework | [FOUNDATION-REPORT.md](FOUNDATION-REPORT.md) | the steps in that file |
| **Stack**: something new is being built and the user asks you to choose what it is built on | "which stack", "what should we use", "pick the framework"; nothing settled about language, runtime or storage | [STACK-REPORT.md](STACK-REPORT.md) | the steps in that file |

When the signals disagree (an empty repo but the user asks for refactoring ideas, or a stack question about an existing codebase), ask which report they want. A Stack report usually ends by offering a Foundation report for the stack the user picks.

All three share the vocabulary below, [LANGUAGE.md](LANGUAGE.md), [DEEPENING.md](DEEPENING.md)'s dependency categories, and the delivery, primitives, style and tone rules in [HTML-REPORT.md](HTML-REPORT.md). Only the review uses CI mode.

## Process (review)

### 1. Explore

Read the project's domain glossary and any ADRs in the area you're touching first.

Then use the Agent tool with `subagent_type=Explore` to walk the codebase. Don't follow rigid heuristics — explore organically and note where you experience friction:

- Where does understanding one concept require bouncing between many small modules?
- Where are modules **shallow** — interface nearly as complex as the implementation?
- Where have pure functions been extracted just for testability, but the real bugs hide in how they're called (no **locality**)?
- Where do tightly-coupled modules leak across their seams?
- Which parts of the codebase are untested, or hard to test through their current interface?

Apply the **deletion test** to anything you suspect is shallow: would deleting it concentrate complexity, or just move it? A "yes, concentrates" is the signal you want.

### 2. Present candidates as a report

When running in CI or other non-interactive automation (`CI=true`, `GITHUB_ACTIONS=true`, or the prompt explicitly asks for a CI artifact), write a persistent Markdown report to `ARCHITECTURE_AUDIT.md` in the repo root. This file is consumed by the biweekly audit workflow, so do not leave the architecture analysis only in `/tmp` or only in an opened browser. Print the report path and enough summary to stdout for the run log.

For interactive use, pick the delivery by what the session has:

- **An Artifact tool (or another private page publisher) is available**: build the report as an Artifact page and publish it, then give the user the link. This is the default, because the user may read the report on a phone, away from the machine. Follow the page contract the tool and its design guidance give (no `<html>`/`<head>`/`<body>` of your own, a two-to-four-word `<title>`, light and dark tokens, pinned script versions, phone width). Keep the source file in the session's scratchpad or temp directory, not in the repo, so a republish to the same URL is one edit away.
- **No publisher**: write a self-contained HTML file to the OS temp directory so nothing lands in the repo. Resolve the temp dir from `$TMPDIR`, falling back to `/tmp` (or `%TEMP%` on Windows), and write to `<tmpdir>/architecture-review-<timestamp>.html` so each run gets a fresh file. Open it for the user — `xdg-open <path>` on Linux, `open <path>` on macOS, `start <path>` on Windows — and tell them the absolute path.

The report uses **Tailwind** (or plain CSS) for layout and styling, and **Mermaid** for diagrams where a graph/flow/sequence reliably communicates the structure. Mix Mermaid with hand-crafted visuals — use Mermaid when relationships are graph-shaped (call graphs, dependencies, sequences), and hand-built ones when you want something more editorial (mass diagrams, cross-sections, collapse animations). Hand-built visuals may be inline SVG, or **React** components when the same primitives (module box, seam line, leak arrow) repeat across candidates or the diagram needs a little interaction, such as a before/after toggle on a phone. Each candidate gets a **before/after visualisation**. Be visual.

**Language**: write the report in the user's language. Keep the glossary terms ([LANGUAGE.md](LANGUAGE.md)) in English, with a one-line legend in the header: translated, words like *deep*, *shallow* and *seam* turn back into the physical metaphors they were defined to replace, and the reader can no longer tell a term from a figure of speech.

For each candidate, the same template as before, but rendered as a card:

- **Files** — which files/modules are involved
- **Problem** — why the current architecture is causing friction
- **Solution** — plain English description of what would change
- **Benefits** — explained in terms of locality and leverage, and how tests would improve
- **Before / After diagram** — side-by-side, custom-drawn, illustrating the shallowness and the deepening
- **Recommendation strength** — one of `Strong`, `Worth exploring`, `Speculative`, rendered as a badge

End the report with a **Top recommendation** section: which candidate you'd tackle first and why.

**Use CONTEXT.md vocabulary for the domain, and [LANGUAGE.md](LANGUAGE.md) vocabulary for the architecture.** If `CONTEXT.md` defines "Order," talk about "the Order intake module" — not "the FooBarHandler," and not "the Order service."

**ADR conflicts**: if a candidate contradicts an existing ADR, only surface it when the friction is real enough to warrant revisiting the ADR. Mark it clearly in the card (e.g. a warning callout: _"contradicts ADR-0007 — but worth reopening because…"_). Don't list every theoretical refactor an ADR forbids.

See [HTML-REPORT.md](HTML-REPORT.md) for the full HTML scaffold, diagram patterns, and styling guidance.

Do NOT propose interfaces yet. After the report is published or written, ask the user: "Which of these would you like to explore?" When a question tool (such as AskUserQuestion) is available, ask with it and offer the candidates as its options, so the answer reaches you even when the user reads the report away from the terminal.

### 3. Grilling loop

Once the user picks a candidate, drop into a grilling conversation. Walk the design tree with them — constraints, dependencies, the shape of the deepened module, what sits behind the seam, what tests survive.

Side effects happen inline as decisions crystallize:

- **Naming a deepened module after a concept not in `CONTEXT.md`?** Add the term to `CONTEXT.md` — same discipline as `/grill-with-docs` (see [CONTEXT-FORMAT.md](CONTEXT-FORMAT.md)). Create the file lazily if it doesn't exist.
- **Sharpening a fuzzy term during the conversation?** Update `CONTEXT.md` right there.
- **User rejects the candidate with a load-bearing reason?** Offer an ADR, framed as: _"Want me to record this as an ADR so future architecture reviews don't re-suggest it?"_ Only offer when the reason would actually be needed by a future explorer to avoid re-suggesting the same thing — skip ephemeral reasons ("not worth it right now") and self-evident ones. See [ADR-FORMAT.md](ADR-FORMAT.md).
- **Want to explore alternative interfaces for the deepened module?** See [INTERFACE-DESIGN.md](INTERFACE-DESIGN.md).
