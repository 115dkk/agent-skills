# Finding Classification Guide

This reference helps classify audit findings into the correct
category. When in doubt, apply the **observability test**: if a
user or caller of the system could detect the change through any
public interface (API, CLI, UI, file output, log output), it is
Category A. If not, it is Category B.

## Category A — Behavior Change

The system's observable output changes. Requires `/115dkk-skills:tdd`.

### Definite Category A

- Missing error handling (currently throws/crashes, should return
  error or recover gracefully)
- Uncovered edge case (valid input produces wrong output)
- New validation at a trust boundary
- API signature change (parameters added/removed/retyped)
- Shallow module deepening that changes the public interface
- Race condition fix that changes observable timing behavior
- Security fix that changes access control behavior
- New feature required by audit (e.g., "add structured logging
  to payment path" — the log output is observable behavior)

### Borderline → Classify as A

- Error message text changes (callers may parse error strings)
- Return type narrowing (e.g., `any` → `Result<T, E>`)
- Adding retry logic (changes observable timing and success rate)
- Changing default values

## Category B — Pure Refactor

The system's observable output must be identical before and after.
Existing test suite is the only gate. No new tests.

### Definite Category B

- God file split into modules (same exports, same behavior)
- Circular dependency removal (restructure imports)
- Dead code deletion (unreachable branches, unused exports)
- Module relocation (move files, update imports)
- Duplication consolidation (extract shared function)
- Rename internal variables/functions (not in public API)
- Extract private helper from long function
- Reorder code for readability

### Borderline → Usually B, verify carefully

- Replacing a library with another (e.g., moment → date-fns):
  B if the public interface is unchanged, A if behavior differs
- Changing internal data structures: B if all tests pass without
  modification, A if tests need updating
- Performance optimization: B if output is identical, A if it
  changes error behavior under load

### When B becomes A during execution

If you perform a Category B change and a test fails, STOP.
This means the change altered observable behavior. Options:

1. Undo the change and reclassify as A
2. Investigate whether the failing test was testing implementation
   (in which case the test is wrong, not the refactor — but this
   judgment requires user confirmation)

## Category C — Config/Type/Dependency

No runtime behavior change. Compiler or linter is the gate.

### Definite Category C

- Lint rule additions or fixes
- Type annotation additions (adding types to untyped code)
- `tsconfig.json` / `ruff.toml` / `.eslintrc` changes
- Package version bumps (minor/patch, no API changes)
- CI workflow modifications
- `.gitignore` / `.editorconfig` changes
- Documentation fixes (README, comments, docstrings)
- A reason written at a site, for a tech-debt finding that found a
  deliberate oddity with no written reason (the comment cites
  `audit #<ISSUE> <ID>`)
- License file updates
- Dependency removal (unused, confirmed by `knip` / `depcheck`)

### Borderline → Check carefully

- Major version dependency upgrade: C if no code changes needed,
  A if API changes require code adaptation
- Adding a new dependency: C if it replaces inline code with
  identical behavior, A if it enables new functionality
- Compiler flag changes (e.g., stricter null checks): C if code
  already passes, A if code changes are needed to satisfy the flag

## Quick Decision Flowchart

```
Does any public interface change?
├─ Yes → Category A (/115dkk-skills:tdd)
└─ No
   ├─ Does the code structure change? 
   │  ├─ Yes → Category B (test-gate)
   │  └─ No → Category C (lint-gate)
   └─ Unsure? → Run existing tests
      ├─ Tests fail → Category A
      └─ Tests pass → Category B
```
