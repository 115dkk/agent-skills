# 115dkk skills

Agent skills packaged as a Claude Code plugin marketplace.

| Skill | What it does |
|---|---|
| `improve-codebase-architecture` | Finds deepening opportunities in a codebase (refactors that put more behaviour behind a smaller interface), presents them as a visual report with before/after diagrams, then grills through the one you pick. |

## Install

In Claude Code:

```
/plugin marketplace add 115dkk/agent-skills
/plugin install 115dkk-skills@115dkk
```

The skill is then available as `/115dkk-skills:improve-codebase-architecture`.

## Where it comes from

`improve-codebase-architecture` is adapted from [mattpocock/skills](https://github.com/mattpocock/skills) (MIT), as it stood at commit [`694fa30`](https://github.com/mattpocock/skills/tree/694fa30311e02c2639942308513555e61ee84a6f/skills/engineering/improve-codebase-architecture): the self-contained version with its own `LANGUAGE.md`, `DEEPENING.md` and `INTERFACE-DESIGN.md`. `CONTEXT-FORMAT.md` and `ADR-FORMAT.md` come from `grill-with-docs` at the same commit, so the skill's links resolve without that skill installed. Upstream has since split the vocabulary into separate skills and renamed `CONTEXT.md` to `GLOSSARY.md`; this copy keeps the older layout.

Changes from upstream:

- **CI output.** Under `CI=true`, `GITHUB_ACTIONS=true`, or when asked for a CI artifact, the report is written to `ARCHITECTURE_AUDIT.md` in the repo root for a scheduled audit workflow to consume.
- **Artifact delivery.** In an interactive session that can publish an Artifact page, the report is published there by default, so it can be read on a phone. The HTML guide covers the page contract that brings: no document shell, pinned script versions, Mermaid rendered natively, light and dark tokens, before/after stacked at phone width.
- **React for hand-built diagrams.** Repeated diagram primitives may be React components (React 18 UMD plus `htm`, no build step).
- **Pinned CDN versions** in the local-file scaffold too (`cdn.tailwindcss.com/3.4.17`, `mermaid@11.4.1`).
- **The user's language.** The report is written in the user's language with the glossary terms kept in English, because translated they turn back into the metaphors they were defined to replace.
- **Asking which candidate to explore** goes through a question tool when one exists.
- Links to `grill-with-docs` files point to the bundled copies.

## License

MIT. See [LICENSE](LICENSE); the original copyright notice is kept.
