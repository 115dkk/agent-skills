---
name: tdd
description: Test-driven development, red then green at pre-agreed seams. Use when the user wants to build a feature or fix a bug test-first, mentions red-green-refactor, or wants integration tests, and when an audit resolution (such as resolve-audit's behaviour-change category) calls for a test-first fix.
---

# Test-Driven Development

TDD is the red → green loop. This skill is the reference that makes that loop produce tests worth keeping: what a good test is, where tests go, the anti-patterns, and the rules of the loop. Every section applies on every cycle: consult them before and during the loop, not after.

When exploring the codebase, read the project's domain glossary (`CONTEXT.md`, or `GLOSSARY.md` where the project uses that name) so test names and interface vocabulary match the domain language, and respect ADRs in the area you're touching.

## Vocabulary

These words mean what they mean in improve-codebase-architecture. When both skills come from this plugin, its full definitions are in [`../improve-codebase-architecture/LANGUAGE.md`](../improve-codebase-architecture/LANGUAGE.md); the short versions below are enough to work with.

- **Interface**: everything a caller must know to use a module correctly: types, invariants, ordering, error modes, configuration. Not just the signature.
- **Seam**: where an interface lives; a place where behaviour can be observed or altered without editing in that place.
- **Adapter**: a concrete thing that satisfies an interface at a seam. A production adapter and a test adapter together make a seam real.
- **Deep module**: a lot of behaviour behind a small interface. The interface is the test surface.

## What a good test is

Tests verify behaviour through public interfaces, not implementation details. Code can change entirely; tests shouldn't. A good test reads like a specification: "user can checkout with valid cart" tells you exactly what capability exists, and it survives refactors because it doesn't care about internal structure. It asserts the effect a caller cares about, not merely that a call returned without error.

See [tests.md](tests.md) for examples and [mocking.md](mocking.md) for mocking guidelines.

## Seams: where tests go

Tests live at seams, never against internals.

**Test only at pre-agreed seams.** Before writing any test, write down the seams under test and confirm them with the user. No test is written at an unconfirmed seam. You can't test everything, so agreeing the seams up front is how testing effort lands on the critical paths and complex logic instead of every edge case.

Ask: "What's the public interface, and which seams should we test?" Give each proposed seam a one-line note on what it catches and what it misses. When a question tool exists, offer the proposed seams as multi-select options with that note as each option's description.

Agreement may already exist:

- **An approved plan that names the seams** counts for those seams: resolve-audit's confirmed classification table, a spec, an architecture candidate the user picked. Confirm only the seams it doesn't name.
- **With nobody to ask** (a scheduled or otherwise unattended run), take the narrowest public interface that observes the behaviour, state it at the top of the work and in the commit message, and go on.

Prefer a seam the code already exposes. When the right seam doesn't exist yet, that is a design question (how deep the module is, where its interface belongs, what it should expose): settle it with the user before the first test, in the vocabulary above.

### Seams onto code you don't control

Where the code meets something you don't control (a plugin, a driver, an SDK, a remote service), the test adapter at that seam must be able to misbehave the way real counterparts have been seen to misbehave, not only the way the contract says they behave. Give each observed misbehaviour its own named mode and record which issue or audit finding it came from (see [mocking.md](mocking.md)).

When the fix for such a violation is a defence in production code, its comment names the violator and the test that reproduces it. That pair is the evidence a later audit cites instead of flagging the defence as paranoid.

## Anti-patterns

- **Implementation-coupled**: mocks internal collaborators, tests private methods, or verifies through a side channel (querying the database instead of using the interface). The tell: the test breaks when you refactor but behaviour hasn't changed.
- **Tautological**: the assertion recomputes the expected value the way the code does (`expect(add(a, b)).toBe(a + b)`, a snapshot derived by hand the same way, a constant asserted equal to itself), so it passes by construction and can never disagree with the code. Expected values must come from an independent source of truth: a known-good literal, a worked example, the spec.
- **Call-success**: the assertion checks that a call was accepted (`kResultOk`, `Ok(())`, no exception) instead of the effect it was supposed to have. A component that ignores its input still reports success. Assert the result through the interface: the state read back, the samples produced, the value a later call returns.
- **Horizontal slicing**: writing all tests first, then all implementation. Bulk tests verify _imagined_ behaviour: you test the _shape_ of things rather than user-facing behaviour, the tests go insensitive to real changes, and you commit to test structure before understanding the implementation. Work in **vertical slices** instead: one test → one implementation → repeat, each test a **tracer bullet** that responds to what the last cycle taught you.

## Rules of the loop

- **Red before green.** Write the failing test first and watch it fail for the reason you expect, then write only enough code to pass it. Don't anticipate future tests or add speculative features.
- **One slice at a time.** One seam, one test, one minimal implementation per cycle.
- **Refactoring is not part of the loop.** Structural cleanup happens later and on a schedule: improve-codebase-architecture's review and tech-debt-audit find it, and resolve-audit's refactor pass carries it out under the existing tests. Within a cycle, tidy only the lines the cycle just wrote (names, leftovers) before committing. When a caller's procedure has a refactor step, as resolve-audit's behaviour-change category does, that is its scope.
- **Deepening test-first.** When the change is to make a module deeper, write the tests at the deepened module's interface first (red), move the behaviour behind it until they pass (green), then delete the old tests on the shallow modules that the new ones now cover. Replace them; don't layer new tests on top.

## Stack notes

- **Rust.** Integration tests under `tests/` can reach only the crate's public API, so the compiler enforces the seam. Put a test inside a module (`#[cfg(test)]`) only for an internal seam that module's own tests need, and never make an item `pub` just to test it.
- **C++.** Test through the public headers. No `#define private public` and no `friend` test hooks; if a test needs an internal, the seam is in the wrong place.
- **Native hosts and FFI.** Make the test double a real loadable module built from the repository (a test plugin, a fake shared library), so tests exercise the same ABI path as production.
