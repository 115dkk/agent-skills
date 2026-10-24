---
name: tech-debt-audit
description: Whole-repository tech debt audit that writes TECH_DEBT_AUDIT.md, with file-cited findings across nine dimensions, severity and effort, compared against the previous audit when one is available. Use when the user asks for a tech debt audit, a codebase health check or a code quality assessment of a whole repository, or when a scheduled audit asks for one. Not for reviewing a diff, and not for choosing deepening candidates (that is improve-codebase-architecture).
---

# Tech Debt Audit

A deliberate, opinionated audit of a whole codebase. It writes `TECH_DEBT_AUDIT.md` in the repository root, with a finding for every concrete problem, each cited to `file:line`.

It complements improve-codebase-architecture rather than repeating it. That skill asks one question, where a module could be deeper, and leaves the decision to a person. This one sweeps nine dimensions line by line, most of which the architecture review never looks at (security, dependencies, error handling, documentation drift, test debt), and produces findings small enough to fix one commit at a time.

## Operating principles

Find what is actually wrong. Not diplomatic, not surface-only. Do not pattern-match to generic best practices without grounding in this repository. No sycophancy and no "overall the codebase is well-structured" filler.

Cite `file:line` for every concrete finding; "the code generally..." does not count. Read code before judging it: a pattern that looks wrong in isolation may be carrying weight, and the reason may be written down somewhere (a comment, a test, an ADR).

## Phase 0: Mode, scope and baseline

- **Mode.** CI mode applies when `CI=true` or `GITHUB_ACTIONS=true`, or when the prompt asks for a CI artifact. In CI mode nobody can answer a question: every question goes into the Open questions section, nothing is committed, and stdout gets a short summary (the report path, counts by severity and status, the Top 5 IDs). Otherwise, publish progress the way the session allows (see Phase 1).
- **Scope.** If the user or the prompt names a subtree (`/tech-debt-audit src/payments`), audit only that subtree.
- **Baseline.** If `TECH_DEBT_AUDIT.md` already exists in the repository root, it is the previous audit: read it before anything else. A scheduled workflow places the last run's file there without committing it, so its presence does not mean it is current. Note its date and commit, then follow the repeat-run rules in Phase 3. With no baseline, the header says so.
- **Architecture review.** If `ARCHITECTURE_AUDIT.md` exists (a scheduled run usually writes it just before this audit), read its **Fine as is** section. Those are modules a deliberate architecture review decided to leave alone.

## Phase 1: Orient

Do not skip this. Forming opinions before understanding the system produces bad audits.

1. Read the README, the package manifests (`package.json`, `pyproject.toml`, `Cargo.toml`, `go.mod`, `*.vcxproj`/`*.sln`, `CMakeLists.txt`), and the recorded decisions: `docs/adr/`, any `ARCHITECTURE` file, lists of failure modes that must not return, the domain glossary (`CONTEXT.md` or `GLOSSARY.md`).
2. Map the directory structure and identify the major modules.
3. Run `git log --oneline -200` and `git log --stat --since="6 months ago"` to see what is actually changing and where churn concentrates. With a baseline, also run `git log --stat <baseline commit>..HEAD`.
4. Identify entry points, hot paths and cold corners.
5. List the 20 largest files by line count and the 20 files most often modified in the last 6 months. The intersection is where debt usually hides.
6. Publish a plan so progress through the phases is visible: the session's task-list tool when there is one, otherwise a short checklist in the response.

Write a one-to-two-paragraph mental model of the architecture before going on. If it contradicts the README, flag that; it is itself a finding.

## Phase 2: Audit across these dimensions

Use `rg`, `ast-grep` and the stack's own tools ([STACK-TOOLS.md](STACK-TOOLS.md)) to find concrete examples. Cite `path/to/file.ext:LINE` for every finding.

1. **Architectural decay**: circular dependencies, layering violations, god files (>500 LOC) and god functions, duplicated logic across 3+ sites where an abstraction should exist, abstractions that exist but nobody uses, dead code (unused exports, unreachable branches, stale commented-out blocks). Where `ARCHITECTURE_AUDIT.md`'s Fine as is names a module, do not recommend splitting or merging it. If you disagree with that call, put the disagreement and your evidence under Open questions instead.
2. **Consistency rot**: several ways of doing the same thing (HTTP clients, error handling, logging, config loading, validation, date handling), naming drift, folder structure that no longer reflects what the code does.
3. **Type and contract debt**: `any`, `as any`, `# type: ignore`, `void*` and untyped integers carrying several meanings, loose dicts, untyped API boundaries, missing validation at trust boundaries (including data a third-party plugin, library or service hands back).
4. **Test debt**: coverage gaps on critical paths (run coverage when it is available), tests that assert implementation rather than behaviour, tests that assert a call succeeded instead of what it achieved, skipped or flaky tests, high-churn files with no tests.
5. **Dependency and config debt**: known vulnerabilities, unused or duplicate dependencies, unpinned versions where reproducibility matters, environment variables referenced but not documented or defaulted inconsistently.
6. **Performance and resource hygiene**: N+1 queries, synchronous work on asynchronous or realtime paths, blocking I/O or allocation on hot paths, handles, listeners and threads that are never released.
7. **Error handling and observability**: swallowed exceptions, blanket catches, errors logged but not handled, several unrelated failures reported as one, inconsistent error shapes across modules, missing structured logs on critical paths.
8. **Security hygiene**: hardcoded secrets, string-built SQL or shell commands, missing input validation at trust boundaries, permissive auth or CORS, weak crypto, memory-safety hazards in native code (use after free, unchecked lengths, uninitialised reads).
9. **Documentation drift**: README claims that do not match reality, comments that contradict the code next to them, public APIs without documentation, and deliberate oddities whose reason is written nowhere (see the Looks bad but is fine rules).

## Phase 3: Deliverable

Write `TECH_DEBT_AUDIT.md` in the repository root, in the language of the repository's documentation, with these sections in order.

- **Header**: repository, date, commit, mode (interactive or CI), scope, and the baseline (its date and commit, or "none").
- **Executive summary**: at most 10 bullets ranked by impact, with counts by severity and, when there is a baseline, by status.
- **Architectural mental model**: the system as it actually is.
- **Findings table**: `ID | Status | Category | File:Line | Severity (Critical/High/Medium/Low) | Effort (S/M/L) | Description | Recommendation`. Aim for 30 to 80 findings on a substantial repository; on a small one report only what is material. Padding past the real findings is noise.
- **Resolved since the baseline**: one line per baseline finding that no longer applies, with the evidence (the code as it is now, or the commit that changed it). Omit the section when there is no baseline.
- **Top 5 "if you fix nothing else, fix these"**: concrete diff sketches or refactor outlines, not vague advice.
- **Quick wins**: Low effort with Medium or higher severity, as a checklist.
- **Looks bad but is fine**: see the rules below.
- **Open questions for the maintainer**: what you could not tell was debt or intent, and disagreements with recorded decisions.

### IDs and status

IDs have the form `TD-01`, `TD-02`, and so on. A finding the baseline already had keeps the baseline's ID and gets status `OPEN`, updated if its location or severity changed. A new finding gets the next unused number and status `NEW`. Never reuse the ID of a resolved finding. Stable IDs let code comments and commits cite an audit finding, and let a later audit tell whether that finding was fixed.

### Looks bad but is fine

This section shows the calls you considered and dropped, so the maintainer can check your judgement. It is not a quota.

- **Cite the evidence.** Each entry names the code (`file:line`), why it looks wrong, and the evidence in the repository that makes it right: a comment at the site, a test that pins the behaviour, an ADR or decision record, an earlier audit ID cited in the code.
- **No evidence, no entry.** If the call is right but its reason is written nowhere in the repository, it does not go here. It is a **Documentation drift** finding (Low severity, S effort) whose recommendation is to write the reason at the site, citing the audit ID if the project cites them. A reason that only a reviewer can reconstruct will be questioned again by the next reader and the next audit, and once it is written the code no longer looks wrong to anyone who reads it.
- **Do not repeat the baseline.** Leave out entries the baseline already lists, unless the code they rest on has changed since the baseline commit. Say in one line how many baseline entries still hold.
- **Empty is allowed.** If there is nothing new, say so in one line. Never invent entries to fill the section.

## Rules

- Cite `file:line` for every concrete finding.
- If unsure whether something is debt or intentional, ask under Open questions; do not assert.
- Do not recommend rewrites. Recommend specific, scoped changes.
- Do not pad. If a dimension has nothing material, write "Nothing material" and move on.
- Do not silently contradict a recorded decision. If an ADR or a decision record blocks a fix you think is needed, say which decision and why the friction justifies reopening it.
- No sycophancy. Say what is broken.

## Large repositories

If the repository is over 50k LOC or has more than 5 top-level modules, audit the modules in parallel when the session can run subagents: one per module, each given its scope, the nine dimensions, the citation requirement, the baseline's findings for that module, and a cap of 200 findings. The main agent merges, deduplicates, assigns IDs and ranks. Without subagents, audit module by module and say in the header that the run was serial.
