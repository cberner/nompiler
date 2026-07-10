# Agent Instructions

Implement the bootstrap MVP specified by `charter.md`, `ui.md`, and `bootstrap.md`.

- Ensure the `design/bootstrap` branch and the `code/bootstrap` branch exist as described in `bootstrap.md`. Preserve
  and reconcile any existing state; never reset either branch merely to manufacture a clean starting point.
- Perform architecture and canonical Design work on the `design/bootstrap` branch.
- During the initial human-driven bootstrap, perform implementation and QA work on the `code/bootstrap` branch, using
  internal task branches when useful. Keep `BOOTSTRAP-TASKS.md` synchronized with completed Design tasks, but never use
  it as runtime state in the generated Nom implementation. Append Design- and product-level findings to
  `DESIGN-FINDINGS.md` for the Architect.
- Prioritize an early working `nom` executable with the read-only `nom bootstrap` advisor required by `bootstrap.md`.
- Run every test inside Podman against disposable repository copies. Never expose the real repository or its worktrees
  to tests as writable storage, and never fall back to host test execution.
- The manual bootstrap may advance `design/bootstrap` and `code/bootstrap` directly because Nom does not exist yet. The
  generated Nom implementation must enforce the Charter's review, stale-work rejection, and safe integration
  requirements.
- Do not implement the bootstrap MVP directly on `master`, `main`, or another Charter branch.
- Treat `charter.md`, `ui.md`, and `bootstrap.md` as requirements.
- Carry work through implementation, tests, review, and verification against the acceptance criteria in `bootstrap.md`.
