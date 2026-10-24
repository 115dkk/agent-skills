---
name: resolve-audit
description: Resolve automated audit findings from a GitHub issue into a PR. Classifies each finding (behavior change → /115dkk-skills:tdd, pure refactor → test-gate, config/type → lint-gate), processes them in order, commits individually, and creates a PR linked to the issue. Use when the user says "resolve audit", "fix audit findings", "process audit issue", references an automated audit issue (Biweekly Audit, Alpha Audit) by number, or wants to act on /115dkk-skills:improve-codebase-architecture or /115dkk-skills:tech-debt-audit output.
---

# Resolve Audit

A Claude Code skill that takes an automated audit issue (produced by
`/115dkk-skills:improve-codebase-architecture` + `/115dkk-skills:tech-debt-audit`) and methodically
resolves every finding into a single PR with per-finding commits.

When invoked, follow the protocol below.

---

## Phase 0: Read and Classify

Do not write any code in this phase.

1. Get the issue number from the user. Read it, including the
   comments, where a scheduled audit posts its reports:
   ```
   gh issue view <NUMBER> --json title,body,comments
   ```
   Also read the project's `CLAUDE.md` or `AGENTS.md` for its build
   and test commands and any project-specific classification rules.

2. Parse every finding from both the Architecture Analysis and
   Tech Debt Audit parts. Include findings from all severity
   levels, and keep each finding's audit ID (`TD-07`, `C2`, ...).
   Exclude only:
   - entries under **Looks bad but is fine** (tech debt audit) and
     **Fine as is** (architecture review): acknowledge and set aside;
   - entries under **Resolved since the baseline**: already done.
   A tech-debt finding marked `OPEN` was carried over from an
   earlier audit: check whether an earlier PR left it unresolved
   and why before planning it again.

3. Classify each finding into exactly one category. See
   [classification-guide.md](references/classification-guide.md)
   for detailed criteria and edge cases. The short version:

   **Category A — Behavior Change**: observable output changes,
   new error handling, API changes, shallow→deep module rewrites.
   These require `/115dkk-skills:tdd` (red → green at an agreed
   seam).

   **Category B — Pure Refactor**: god file splits, circular dep
   removal, dead code deletion, module relocation, duplication
   consolidation. External behavior must not change. Existing test
   suite is the gate. New tests are forbidden.

   **Category C — Config/Type/Dependency**: lint rules, type
   annotations, package updates, CI config, documentation fixes.
   Compiler or linter is the gate.

4. Print the classification table. For every Category A item, name
   the seam its test will go through, with a few words on what that
   seam catches and misses; `/115dkk-skills:tdd` writes no test at a
   seam nobody agreed to.
   ```
   | # | Audit ID | Finding (≤60 chars) | Cat | Severity | File(s) | Seam (A only) |
   |---|----------|---------------------|-----|----------|---------|---------------|
   | A-1 | TD-07 | Missing error handling in parser | A | High | src/parser.cpp | `parse()` result; misses the CLI's exit code |
   | B-1 | TD-12 | Split god file utils.py | B | Medium | src/utils.py | |
   | C-1 | TD-21 | Add type annotations to config | C | Low | src/config.ts | |
   ```

5. Present the table and wait for user confirmation before
   proceeding. The user may reclassify items, exclude some, or
   change a seam. The confirmed table is the seam agreement for
   every Category A item; a cycle that needs a seam the table does
   not name stops and asks.

6. After confirmation, create a working branch:
   ```
   git checkout -b fix/audit-<ISSUE_NUMBER>
   ```

## Phase 1: Category A — /115dkk-skills:tdd

For EACH Category A finding, in order:

1. Announce: `── A-{n}: {finding summary} ──`
2. Activate the `/115dkk-skills:tdd` skill methodology.
3. **Red**: Write a failing test at the agreed seam that captures
   the missing or incorrect behavior. Run the test. Confirm it
   fails, and fails for the reason the finding describes.
4. **Green**: Write the minimum code to make the test pass.
   Run the test. Confirm it passes.
5. **Tidy**: Clean up only the lines this cycle wrote (names,
   leftovers) while keeping the test green. Structural cleanup
   elsewhere is a Category B item or the next audit's business.
6. Run the full test suite to verify no regressions.
7. Commit: `fix(audit): A-{n} {concise description} (#<ISSUE> {Audit ID})`
8. Proceed to the next item.

Two kinds of Category A item have their own shape:

- **Deepening a module** (an architecture candidate): write the tests
  at the deepened module's interface first, move the behaviour behind
  it until they pass, then delete the old tests on the shallow modules
  that the new ones cover. Deleting those tests is part of the item.
- **A defence against code the project does not control** (a plugin,
  a driver, an SDK that breaks its contract): first add a mode to the
  project's test adapter that reproduces the violation, as
  `/115dkk-skills:tdd`'s mocking guide describes. The defence's
  comment names the violator, that test, and the finding
  (`audit #<ISSUE> {Audit ID}`), so later audits read it as earned.

If a `/115dkk-skills:tdd` cycle reveals that a Category B item actually changes
behavior, reclassify it as A and process it here.

If there are zero Category A findings, skip to Phase 2.

## Phase 2: Category B — Refactor with Test Gate

For EACH Category B finding, in order:

1. Announce: `── B-{n}: {finding summary} ──`
2. Run the full test suite BEFORE touching code. Record:
   - Total tests, passed, failed, skipped.
   - If any test fails pre-change, STOP and report it. Do not
     proceed with this item until the pre-existing failure is
     understood.
3. Perform the structural change.
4. Run the full test suite AFTER the change. Compare counts:
   - Pass count must be ≥ previous. Zero new failures.
   - If a test breaks, the change altered observable behavior.
     Undo, reclassify as Category A, and process with `/115dkk-skills:tdd`
     in a later pass.
5. Commit: `refactor(audit): B-{n} {concise description} (#<ISSUE> {Audit ID})`

Do NOT write new tests for Category B. The existing suite is the
contract. Writing "tests that verify the refactor worked" couples
tests to internal structure, which is the anti-pattern `/115dkk-skills:tdd`
explicitly warns against.

If there are zero Category B findings, skip to Phase 3.

## Phase 3: Category C — Config/Type/Dependency Gate

For EACH Category C finding, in order:

1. Announce: `── C-{n}: {finding summary} ──`
2. Apply the change.
3. Verify: compiler passes, linter passes, or dependency
   resolves without conflict.
4. Commit: `chore(audit): C-{n} {concise description} (#<ISSUE> {Audit ID})`

A tech-debt finding that asks for a reason to be written at a site
(the audit found a deliberate oddity whose reason is written
nowhere) is Category C: write the comment at that line, say why the
code is right, and cite `audit #<ISSUE> {Audit ID}`. Once it is
written, later audits stop questioning that code.

If there are zero Category C findings, skip to Phase 4.

## Phase 4: Final Verification and PR

1. Run the full test suite one final time.
2. Run the linter.
3. Print a completion checklist:
   ```
   ✓ A-1: Missing error handling in parser
   ✓ B-1: Split god file utils.py
   ✗ B-3: Remove circular dep (reclassified → A-4)
   ...
   Total: {resolved}/{total} findings resolved
   ```

4. For any unresolved finding, explain why concisely under the
   checklist. Acceptable reasons: pre-existing test failure,
   requires human decision, depends on external system, would
   exceed scope. "Seemed hard" is not acceptable.

5. Create the PR:
   ```
   gh pr create \
     --title "fix: resolve audit findings from #<ISSUE>" \
     --body-file <(generate PR body per template below)
   ```

   PR body template:
   ```markdown
   Resolves #<ISSUE>

   ## Behavior Changes (TDD applied)
   - A-1: {description}
   - A-2: {description}

   ## Refactors (existing test suite verified)
   - B-1: {description}
   - B-2: {description}

   ## Config/Type/Dependency
   - C-1: {description}

   ## Not Resolved
   - B-3: {reason}

   ## Test Results
   - Full suite: {passed} passed, {failed} failed, {skipped} skipped
   - New tests added: {count} (from Category A items)
   ```

6. Link the PR to the issue:
   ```
   gh issue comment <ISSUE> \
     --body "Addressed in PR #<PR_NUMBER>."
   ```

## Rules

- **Never skip a finding.** If unresolvable, mark as Not Resolved
  with a reason. Omission is a defect; non-resolution is a judgment.
- **Never apply /115dkk-skills:tdd to Category B.** The test suite is the gate.
- **Never commit without verification.** Tests for A/B, linter for C.
- **One finding per commit.** Do not squash unrelated changes.
- **Skip what the audit was told to skip.** Files the project's audit
  excludes (for example `.ipynb` notebooks kept as legacy artifacts)
  are not touched here either.
- **Documentation-only findings are always Category C.**
- **Wait for user confirmation after Phase 0.** The classification
  table is the contract for all subsequent work.

## Customization

Project-specific rules belong in the project's `CLAUDE.md` or
`AGENTS.md`, which Phase 0 reads. Common ones:

- Test commands (e.g., `cargo test` vs `pytest` vs `npm test`, or the
  MSBuild and test executables of a native project)
- Project-specific classification rules
- Commit message prefix conventions
- Severity thresholds (e.g., skip Low severity in automated runs)

## Reference Files

Load these as needed. Do not read all at once.

- [classification-guide.md](references/classification-guide.md)
  — Phase 0에서 분류 판단이 어려울 때 참조.
    관측 가능성 테스트, 경계 사례 판정법, 플로차트 포함.

- [agent-config.md](references/agent-config.md)
  — 서브에이전트를 배분할 때 참조.
    단일 세션과 서브에이전트 중 고르는 기준, 모델/effort 원칙,
    병렬 대상, TDD 담당 결정, 메인 모델의 코드 리뷰 프로토콜.

- [task-organization.md](references/task-organization.md)
  — Phase 0 직후, 실행 순서와 배치 그룹을 결정할 때 참조.
    의존성 그래프, 위상 정렬, 체크포인트 전략, 지뢰 탐지.

- [memory-protocol.md](references/memory-protocol.md)
  — 세션 시작 시 반드시 참조.
    audit-state.md 관리, 작업 목록 도구 병용, 갱신 타이밍,
    치매 방지 원칙, 세션 재개 절차.
