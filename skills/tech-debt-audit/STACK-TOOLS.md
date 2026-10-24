# Stack tooling

Detect the stack from the manifests and run the matching tools, in parallel when you can. Fold their output into findings with `file:line` citations; a tool's warning is evidence for a finding, not a finding by itself.

If a tool is not installed, note it in the report header and move on rather than blocking. Never install anything system-wide. A quick user-local install (`pip install --user lizard`, `cargo install cargo-machete`) is fine when the session has network access; in CI mode, prefer what the workflow provisioned.

## TypeScript / JavaScript

`npm audit`, `npx knip` (dead exports), `npx madge --circular <src>` (circular dependencies), `npx depcheck` (unused dependencies), `tsc --noEmit` (type drift).

## Python

`pip-audit`, `ruff check`, `vulture` (dead code), `pydeps --show-cycles`, `mypy --strict` (type drift).

## Rust

`cargo audit`, `cargo machete` (unused dependencies), `cargo udeps` (needs a nightly toolchain; skip it when there is none), `cargo clippy --all-targets -- -W clippy::pedantic`, and `cargo deny check` when the repository has a `deny.toml`. Check where `unsafe` lives against the repository's own rules (workspace lints, `-sys` crate conventions).

## Go

`govulncheck ./...`, `go vet ./...`, `staticcheck ./...`, `golangci-lint run`.

## C / C++

- `cppcheck --enable=warning,style,performance,portability --inline-suppr -q <source dirs>`.
- `clang-tidy` through `run-clang-tidy -p <build dir>`, only when a `compile_commands.json` exists. Do not guess compile flags; without the database the results are noise.
- Visual Studio projects: `msbuild <project> /p:RunCodeAnalysis=true` runs MSVC code analysis with the project's own settings. Build one project at a time rather than the whole solution.
- `lizard -w -C 15 -L 150 <source dirs>` for functions over the cyclomatic-complexity and length thresholds. Raise the thresholds if the codebase's baseline is higher, and say so.
- Native plugin or library hosts: also check that every foreign callback and every pointer handed back by foreign code is validated where it enters, which no tool here does for you.

## Analyzers the project already runs

Read the CI configuration for analyzers that already run (CodeQL, sanitizer jobs, clippy or clang-tidy settings, fuzzers). When the session can reach their latest results (for example `gh api repos/<owner>/<repo>/code-scanning/alerts` with the right permission), use those instead of re-deriving them, and cite the alert next to the code location. Do not report an alert the project has dismissed with a reason unless the code it covers has changed.
