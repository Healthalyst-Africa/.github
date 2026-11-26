# Contributing to HealthAlyst Africa repositories

Thank you for helping. Each repository has its own `CONTRIBUTING.md`,
`AGENTS.md` and `CLAUDE.md` with its setup, commands and rules; read those
first. Where a repository has none, this page applies.

All HealthAlyst repositories follow one delivery lifecycle, described in the
[engineering handbook](engineering/software-development-lifecycle.md). In
short:

1. Start with an issue. Use the **Change request** form for new work and the
   **Bug report** form for defects.
2. Features get a change record (`intent.md`, `spec.md`, `plan.md`) in
   `docs/changes/` before they are built.
3. Work on a feature branch and open a pull request into the integration
   branch (`dev`), using the pull request template. Never push to `main`,
   `dev` or `staging` directly.
4. Prove the change before review: the repository's checks, and screenshots
   for any screen you changed.
5. A code owner approves; Claude's review is advisory.

Commit messages follow [Conventional Commits](https://www.conventionalcommits.org/),
for example `feat(health-schedule): add triage queue`.

Never put patient records, personal details, credentials or tokens in an
issue, commit, pull request, test fixture or prompt. Report security problems
privately as described in [SECURITY.md](SECURITY.md).
