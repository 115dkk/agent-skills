# 115dkk skills

Agent skills packaged as a Claude Code plugin marketplace.

| Skill | What it does |
|---|---|
| `improve-codebase-architecture` | Architecture reports with diagrams, in three kinds. **Review**: deepening opportunities in an existing codebase (refactors that put more behaviour behind a smaller interface), with before/after diagrams, then a grilling session on the one you pick. **Foundation**: the first architecture of a new project on a given stack: the module map, how each hard part will be built, the alternatives that were rejected and why, and the first slice to build. **Stack**: when asked to choose the stack, the plausible stacks each drawn with the architecture it implies, compared on the project's own criteria, with one recommended. |

## Install

In Claude Code:

```
/plugin marketplace add 115dkk/agent-skills
/plugin install 115dkk-skills@115dkk
```

The skill is then available as `/115dkk-skills:improve-codebase-architecture`.

The plugin's manifest pins no version, so every commit to `main` is a new version. Claude Code leaves auto-update off for marketplaces outside Anthropic's own; turn it on under `/plugin` → **Marketplaces** → `115dkk` → **Enable auto-update**, or run `claude plugin update 115dkk-skills@115dkk` to update by hand.

## Where it comes from

`improve-codebase-architecture` is adapted from [mattpocock/skills](https://github.com/mattpocock/skills) (MIT), as it stood at commit [`694fa30`](https://github.com/mattpocock/skills/tree/694fa30311e02c2639942308513555e61ee84a6f/skills/engineering/improve-codebase-architecture): the self-contained version with its own `LANGUAGE.md`, `DEEPENING.md` and `INTERFACE-DESIGN.md`. `CONTEXT-FORMAT.md` and `ADR-FORMAT.md` come from `grill-with-docs` at the same commit, so the skill's links resolve without that skill installed. Upstream has since split the vocabulary into separate skills and renamed `CONTEXT.md` to `GLOSSARY.md`; this copy keeps the older layout.

Changes from upstream:

- **YAGNI scoping, taken from upstream later.** The review's Explore step first decides where to look: the direction the user named, or else the hot spots in recent `git log`. This is upstream's [`45afd80`](https://github.com/mattpocock/skills/commit/45afd80) (2026-07-13), merged into this older layout.
- **CI output.** Under `CI=true`, `GITHUB_ACTIONS=true`, or when asked for a CI artifact, the report is written to `ARCHITECTURE_AUDIT.md` in the repo root for a scheduled audit workflow to consume.
- **Artifact delivery.** In an interactive session that can publish an Artifact page, the report is published there by default, so it can be read on a phone. The HTML guide covers the page contract that brings: no document shell, pinned script versions, Mermaid rendered natively, light and dark tokens, before/after stacked at phone width.
- **React for hand-built diagrams.** Repeated diagram primitives may be React components (React 18 UMD plus `htm`, no build step).
- **Pinned CDN versions** in the local-file scaffold too (`cdn.tailwindcss.com/3.4.17`, `mermaid@11.4.1`).
- **The user's language.** The report is written in the user's language with the glossary terms kept in English, because translated they turn back into the metaphors they were defined to replace.
- **Asking which candidate to explore** goes through a question tool when one exists.
- Links to `grill-with-docs` files point to the bundled copies.
- **Foundation and Stack reports** (`FOUNDATION-REPORT.md`, `STACK-REPORT.md`), with `SKILL.md` choosing between the three reports from the user's words and the repository's state. They are new in this repository, not from upstream.

## License

MIT. See [LICENSE](LICENSE); the original copyright notice is kept.
