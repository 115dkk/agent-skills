# 115dkk skills

Agent skills packaged as a Claude Code plugin marketplace.

| Skill | What it does |
|---|---|
| `improve-codebase-architecture` | Architecture reports with diagrams, in three kinds. **Review**: deepening opportunities in an existing codebase (refactors that put more behaviour behind a smaller interface), with before/after diagrams, then a grilling session on the one you pick. **Foundation**: the first architecture of a new project on a given stack: the module map, how each hard part will be built, the alternatives that were rejected and why, and the first slice to build. **Stack**: when asked to choose the stack, the plausible stacks each drawn with the architecture it implies, compared on the project's own criteria, with one recommended. |
| `tech-debt-audit` | A whole-repository audit across nine dimensions (architectural decay, consistency, types and contracts, tests, dependencies, performance, error handling, security, documentation), written to `TECH_DEBT_AUDIT.md` with a `file:line` citation, severity and effort for every finding. Compares against the previous audit when one is placed in the repository, and turns "looks bad but is fine" calls into written reasons instead of a list that repeats every run. |

## Install

In Claude Code:

```
/plugin marketplace add 115dkk/agent-skills
/plugin install 115dkk-skills@115dkk
```

The skills are then available as `/115dkk-skills:improve-codebase-architecture` and `/115dkk-skills:tech-debt-audit`.

The plugin's manifest pins no version, so every commit to `main` is a new version. Claude Code leaves auto-update off for marketplaces outside Anthropic's own; turn it on under `/plugin` → **Marketplaces** → `115dkk` → **Enable auto-update**, or run `claude plugin update 115dkk-skills@115dkk` to update by hand.

## Where it comes from

### improve-codebase-architecture

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

### tech-debt-audit

`tech-debt-audit` is adapted from [ksimback/tech-debt-skill](https://github.com/ksimback/tech-debt-skill) (MIT), as it stood at commit [`5a15c1c`](https://github.com/ksimback/tech-debt-skill/tree/5a15c1ca4a929b2759461c218478de391a8bda0f) (2026-04-25), that repository's only commit so far. The orientation phase, the nine dimensions, the citation rule and the shape of the report are upstream's.

Changes from upstream:

- **Protocol only.** The installation, philosophy and contribution notes, about half of upstream's `SKILL.md`, are left out so they no longer load into every audit, and the stack tooling moved to `STACK-TOOLS.md`. The harness-neutral wording for the plan and subagent steps and the smaller finding count for small repositories follow upstream's open [PR #2](https://github.com/ksimback/tech-debt-skill/pull/2).
- **Model-invocable.** Upstream's `disable-model-invocation` is dropped, as in the architecture skill; the description limits it to whole-repository audit requests. A scheduled workflow no longer has to strip that line before the session can run the skill.
- **CI mode.** Under `CI=true`, `GITHUB_ACTIONS=true`, or when asked for a CI artifact, questions go into the report, nothing is committed, and stdout gets a short summary.
- **A baseline that works without committing the report.** A `TECH_DEBT_AUDIT.md` found in the repository root is read as the previous audit, so a scheduled workflow can put the last run's file there from its artifacts. Findings keep stable `TD-NN` IDs across audits with `OPEN` and `NEW` status, and the report gains a **Resolved since the baseline** section.
- **"Looks bad but is fine", rewritten.** The section is no longer required to be non-empty, which pushed the model to pad it. Each entry cites the evidence in the repository that makes the code right; a correct call whose reason is written nowhere becomes a Documentation drift finding to write that reason at the site; entries the baseline already lists are not repeated.
- **Reads the architecture review.** When `ARCHITECTURE_AUDIT.md` exists, modules its **Fine as is** section names are not proposed for splitting or merging; a disagreement goes to Open questions.
- **Native code.** C and C++ tooling (cppcheck, clang-tidy with a compilation database, MSVC code analysis, lizard), results of analyzers the project already runs, and dimension additions for this kind of code: integers and `void*` carrying several meanings, validation of what third-party code hands back, call-success tests, several failures reported as one, memory-safety hazards.

## License

MIT. See [LICENSE](LICENSE); the original copyright notices are kept.
