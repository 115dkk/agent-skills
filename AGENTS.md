# Working agreement

This repository is a Claude Code plugin marketplace whose single plugin is the repository itself (`.claude-plugin/marketplace.json` lists it with source `./`).

## Never edit the plugin version

`version` in `.claude-plugin/plugin.json` and `.codex-plugin/plugin.json` belongs to `.github/workflows/plugin-version.yml`. Do not change it, do not remove it, and do not add a `version` to the marketplace entry.

Why: Claude Code reads the manifest's `version` first and keeps every install on its cached copy until that string changes. Until October 2026 the manifest said `1.0.0` and never changed, so no commit after the first install reached anyone, and the marketplace had to be removed and added again to recover. A manifest with no version keeps updates flowing but makes `claude plugin validate` warn, which invites someone to "fix" it with a fixed version again. The workflow sets `1.0.N` after each push that touches `skills/`, `.claude-plugin/`, `.codex-plugin/` or `assets/`, so the field is always present and always changes.

Consequences for your work:

- After you push a change to `skills/`, `.claude-plugin/`, `.codex-plugin/` or `assets/`, the workflow adds a `chore(version): 1.0.N` commit. Fetch and rebase before your next push.
- If `claude plugin validate .` reports a missing version, the workflow did not run or failed. Look at its run; do not write a version yourself.

## Adapted skills

Each skill adapted from another repository keeps a section under "Where it comes from" in `README.md`: the upstream repository, the exact commit it was taken from, and a list of the changes. Update that section in the same commit as the change, and keep the upstream copyright line in `LICENSE`.

## Commits

Conventional prefixes (`feat:`, `fix:`, `docs:`, `ci:`), English descriptions. Work on `main` directly.
