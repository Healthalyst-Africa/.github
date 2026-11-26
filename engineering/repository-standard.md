# Repository standard

What every HealthAlyst repository has, and where each stands. Repositories
adopt it in the order of the [lifecycle's adoption steps](software-development-lifecycle.md#adopting-it).
`healthalyst-frontend` is the reference implementation; copy from it.

## The standard

| Area              | Requirement                                                                                                                                                                      | Why                                                               |
| ----------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ----------------------------------------------------------------- |
| Instructions      | `AGENTS.md` with the repository's engineering rules; root `CLAUDE.md` that starts with `@AGENTS.md` and holds commands, **Verifying your work** and **Things Claude gets wrong** | Claude only reads `AGENTS.md` natively when no `CLAUDE.md` exists |
| Claude settings   | `.claude/settings.json` registering the `healthalyst` marketplace, enabling `healthalyst-sdlc` and `security-guidance`, and denying reads of `.env` files                        | Shared skills and guardrails in every session                     |
| Review            | `REVIEW.md` with bug, security and policy passes; Claude review workflow calling this organisation's reusable workflow                                                           | Consistent, advisory first review on every pull request           |
| Ownership         | `.github/CODEOWNERS` covering everything, with `.github/`, `.claude/` and agent instruction files listed explicitly                                                              | Code owner approval on protected branches                         |
| Branch protection | Rulesets on `main` and the integration branch: pull requests only, required checks, code scanning results, no force pushes or deletion; one code owner approval on `main`        | Nothing reaches production unreviewed                             |
| CI                | A required check that runs format, lint, types, tests and build on every pull request                                                                                            | One gate everyone trusts                                          |
| Security scanning | CodeQL, dependency review, secret scanning with push protection, Dependabot alerts and security updates, private vulnerability reporting                                         | Vulnerabilities found before release                              |
| Dependencies      | `dependabot.yml` for the package manager and GitHub Actions, into the integration branch, grouped, with a cooldown                                                               | Updates are small, regular and reviewed                           |
| Workflows         | Actions pinned to commit SHAs; read-only top-level permissions; timeouts; no untrusted input in scripts                                                                          | Supply-chain safety                                               |
| Records           | `docs/changes/`, `docs/adrs/`, `docs/incidents/`                                                                                                                                 | Why the code is the way it is                                     |
| Evals             | `evals/` with tasks and the agent evals workflow                                                                                                                                 | Instructions changes don't make Claude worse                      |
| Maintenance       | CI health, release records and delivery metrics workflows                                                                                                                        | The loop from production back to planning                         |
| Scorecard         | OpenSSF Scorecard workflow and badge (public repositories)                                                                                                                       | Independent measure of the above                                  |

## Where each repository stands

| Requirement                                | healthalyst-frontend           | healthalyst-backend                                                  | Marketing-module                                                     |
| ------------------------------------------ | ------------------------------ | -------------------------------------------------------------------- | -------------------------------------------------------------------- |
| `AGENTS.md` and `CLAUDE.md`                | Done                           | `AGENTS.md` holds only the Turborepo block; no `CLAUDE.md`           | `AGENTS.md` present; no `CLAUDE.md`                                  |
| Claude settings and plugin                 | Done                           | Missing                                                              | Missing                                                              |
| `REVIEW.md` and Claude review              | Done (needs the API key)       | Missing                                                              | Missing                                                              |
| `CODEOWNERS`                               | Done                           | Missing                                                              | Missing                                                              |
| Rulesets                                   | Written as code; to be applied | One ruleset on `dev` and `main` with no required approvals or checks | Classic protection on `main` requiring the `verify` check, no review |
| Required CI check                          | Client CI                      | CI exists; not required                                              | `verify` required                                                    |
| CodeQL and dependency review               | Done                           | Missing                                                              | Missing                                                              |
| Secret scanning and push protection        | To be applied                  | Unknown (private repository)                                         | Off                                                                  |
| `dependabot.yml`                           | Done                           | Missing                                                              | Missing                                                              |
| Actions pinned to SHAs                     | Done                           | Pinned by tag                                                        | Pinned by tag                                                        |
| `docs/adrs/`                               | Done                           | Done                                                                 | Missing                                                              |
| `docs/changes/`, `docs/incidents/`         | Done                           | Missing                                                              | Missing                                                              |
| Evals, CI health, release records, metrics | Done                           | Missing                                                              | Missing                                                              |
| Scorecard                                  | Done                           | Not applicable (private)                                             | Missing                                                              |

To bring a repository up to standard, copy the matching files from
`healthalyst-frontend`, adjust the commands and check names, and apply the
settings with its `scripts/github/apply-repository-settings.sh` pointed at
the repository (`REPOSITORY=Healthalyst-Africa/<name>`).
