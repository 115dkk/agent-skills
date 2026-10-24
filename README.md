# 115dkk skills

<img src="assets/klchan-cleaning.png" alt="클쨩 enthusiastically sweeping with a broom" width="160" height="160">

Agent skills packaged as a Claude Code plugin marketplace.

| Skill | What it does |
|---|---|
| `improve-codebase-architecture` | Architecture reports with diagrams, in three kinds. **Review**: deepening opportunities in an existing codebase (refactors that put more behaviour behind a smaller interface), with before/after diagrams, then a grilling session on the one you pick. **Foundation**: the first architecture of a new project on a given stack: the module map, how each hard part will be built, the alternatives that were rejected and why, and the first slice to build. **Stack**: when asked to choose the stack, the plausible stacks each drawn with the architecture it implies, compared on the project's own criteria, with one recommended. |
| `tech-debt-audit` | A whole-repository audit across nine dimensions (architectural decay, consistency, types and contracts, tests, dependencies, performance, error handling, security, documentation), written to `TECH_DEBT_AUDIT.md` with a `file:line` citation, severity and effort for every finding. Compares against the previous audit when one is placed in the repository, and turns "looks bad but is fine" calls into written reasons instead of a list that repeats every run. |
| `tdd` | Test-driven development as a red → green loop at seams agreed before any test is written: what a good test is, where tests go, the anti-patterns (implementation-coupled, tautological, call-success, horizontal slicing), test adapters that misbehave like real third-party code, and Rust and C++ notes. |
| `resolve-audit` | Turns an audit issue (the architecture review and the tech debt audit together) into one pull request: classifies each finding as a behaviour change (test-first with `tdd` at a seam agreed in the classification table), a pure refactor (the existing tests are the gate) or a config, type or documentation fix (the compiler or linter is the gate), then commits one finding at a time with its audit ID, keeping its state in a file so a long run survives compaction. |

## Install

In Claude Code:

```
/plugin marketplace add 115dkk/agent-skills
/plugin install 115dkk-skills@115dkk
```

The skills are then available as `/115dkk-skills:improve-codebase-architecture`, `/115dkk-skills:tech-debt-audit`, `/115dkk-skills:tdd` and `/115dkk-skills:resolve-audit`.

### Updates

Every change to the plugin reaches installs as a new version: after each push to `main` that touches `skills/` or `.claude-plugin/`, the [plugin version workflow](.github/workflows/plugin-version.yml) sets `version` in `plugin.json` to `1.0.N`, where N counts the commits on `main` other than its own. Claude Code compares that string with the installed one, so a version that stays the same across commits would keep every install on its old copy; that is why nobody edits the field by hand.

Whether an install picks up a new version by itself depends on how the marketplace was added:

- **Added from claude.ai**: auto-update is on by default.
- **Added with `/plugin marketplace add`** (or `claude plugin marketplace add`): auto-update is off by default. Turn it on under `/plugin` → **Marketplaces** → `115dkk` → **Enable auto-update**, or update by hand with `claude plugin update 115dkk-skills@115dkk`.

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

### tdd

`tdd` is adapted from [mattpocock/skills](https://github.com/mattpocock/skills) (MIT), as it stood at commit [`3f59913`](https://github.com/mattpocock/skills/tree/3f599139104dfceabc7754eeb91319802f85e042/skills/engineering/tdd) (2026-10-07), its latest change: the reference-only version with pre-agreed seams, the tautological-test anti-pattern, a red → green loop without a refactor stage, and a note on what each proposed seam catches and misses. `tests.md` and `mocking.md` start from upstream's.

Changes from upstream:

- **Self-contained vocabulary.** Upstream calls a separate `codebase-design` skill for the meaning of interface, seam and adapter. This plugin has no such skill, so `tdd` carries short definitions that match `improve-codebase-architecture`'s `LANGUAGE.md` and points to it.
- **`CONTEXT.md`** as the glossary name, as in this plugin's architecture skill, with `GLOSSARY.md` also read.
- **Where seam agreement comes from.** Proposed seams are offered through a question tool when one exists. A plan the user already approved (resolve-audit's classification table, a spec, a picked architecture candidate) counts as agreement for the seams it names, and an unattended run picks the narrowest seam and records it.
- **Where refactoring goes.** Upstream sends it to a `code-review` skill that this plugin doesn't have. Here it goes to the scheduled architecture review and tech-debt audit and resolve-audit's refactor pass; a refactor step inside a cycle covers only the lines that cycle wrote.
- **Deepening test-first**: tests at the deepened interface first, then the old tests on the shallow modules are deleted, not layered over.
- **The call-success anti-pattern**: asserting that a call was accepted instead of what it did.
- **Test adapters that misbehave like real third-party code**, as real loadable modules with one named mode per observed violation, and production defences whose comments cite them.
- **Rust and C++ notes**, with examples in those languages next to upstream's TypeScript ones.

### resolve-audit

`resolve-audit` is this repository's own skill. It was kept and installed separately until it moved here; the first commit that adds it is that copy unchanged. Changes since then:

- **Reads the new audit reports.** Findings keep their audit IDs (`TD-07`, `C2`), which go into the classification table, the commit messages and any code comment written for a finding. Entries under **Looks bad but is fine**, **Fine as is** and **Resolved since the baseline** are set aside, and a carried-over (`OPEN`) finding is checked against the earlier attempt before it is planned again.
- **Seams in the classification table.** Each behaviour change names the seam its test goes through, with what that seam catches and misses, so the confirmed table is the seam agreement `tdd` asks for.
- **Tidy, not refactor, inside a test-first cycle.** As in `tdd`, the step after green touches only the lines the cycle wrote; structural cleanup stays a refactor item or the next audit's.
- **Deepenings and defences have their own shape.** A deepening writes tests at the new interface and deletes the shallow tests they cover; a defence against third-party code first adds the test-adapter mode that reproduces the violation, and its comment cites that test and the finding.
- **Documentation findings that ask for a written reason** are config-type items: the comment goes at the line and cites the audit ID.
- **Model-invocable and harness-neutral.** The lock against model invocation is dropped, as in the other skills; task-list and subagent steps no longer name one harness's tools, and the subagents use the main session's model instead of a model named in the text. Project rules such as test commands come from the project's `CLAUDE.md` or `AGENTS.md` instead of a project-level copy of the skill.

## License

MIT. See [LICENSE](LICENSE); the original copyright notices are kept.

The plugin icon is a new AI-generated illustration of 클쨩 sweeping, based on the character reference in [this post](https://arca.live/b/characterai/167764605). See [the asset notes](assets/README.md) for its generation prompt.
