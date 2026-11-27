# Software development lifecycle

HealthAlyst builds software with Claude writing much of the code. Writing
code is now fast; what keeps quality high is clear intent before building,
proof before review, and enforced controls instead of habits. This lifecycle
follows Anthropic's
[AI-native SDLC playbook](https://claude.com/resources/articles/the-ai-native-sdlc-playbook),
sized for a team of one to three developers.

## Principles

1. **Every stage ends with a committed artifact, and the next stage starts by
   reading it.** Git history records who asked, what was produced and who
   accepted it.
2. **People accept; Claude produces.** A person accepts every gate. The agent
   that wrote a change never approves it.
3. **Enforce, don't remind.** A rule that must always hold is a lint rule,
   hook, ruleset or CI check, not a sentence in a document.
4. **Small batches.** Small pull requests are reviewed better and released
   sooner.
5. **Close the loop.** Every incident adds something that catches it next
   time.

## Roles

One person may hold several roles in a small team. The one rule that never
bends: nobody, human or agent, is the only check on their own work when
another person is available.

| Role                 | Responsibility                                                           |
| -------------------- | ------------------------------------------------------------------------ |
| Product owner        | Accepts intents: is this worth doing, and is the outcome right?          |
| Lead developer       | Accepts specifications and architecture decisions; owns code review.     |
| Developer            | Plans, builds with Claude, verifies, and opens pull requests.            |
| Code owner           | Approves pull requests for the paths they own (see `CODEOWNERS`).        |
| Release manager      | Decides when the integration branch goes to production.                  |
| Data protection lead | Decides on data protection impact assessments and leads breach response. |

## The stages

| Stage      | Starts from                                              | Produces                                                                      | Claude's job                                                                              | Accepted by                                                                          |
| ---------- | -------------------------------------------------------- | ----------------------------------------------------------------------------- | ----------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| 1 Plan     | An idea, a **Change request** issue, or an incident      | `docs/changes/<issue>-<name>/intent.md`                                       | `/healthalyst-sdlc:write-intent` interviews the originator and drafts it                  | Product owner                                                                        |
| 2 Design   | An accepted `intent.md`                                  | `spec.md`, with concerns listed                                               | `/healthalyst-sdlc:write-spec` applies the repository's rules and flags conflicts         | Lead developer, plus the area owner for authentication, permissions or personal data |
| 3 Build    | An accepted `spec.md`                                    | `plan.md`, then the code                                                      | `/healthalyst-sdlc:write-plan` in plan mode, then implementation                          | The developer accepts the plan                                                       |
| 4 Test     | Implemented code                                         | Check results, screenshots, reviewer verdicts                                 | `/healthalyst-sdlc:verify-change`, the `verifier` and `security-reviewer` agents          | Evidence in the pull request                                                         |
| 5 Deploy   | A verified branch                                        | A pull request into `dev`, then a release pull request from `dev` into `main` | `/healthalyst-sdlc:open-pull-request`, Claude review, `/healthalyst-sdlc:prepare-release` | A code owner approves each pull request; the release manager merges to `main`        |
| 6 Maintain | Production signals: CI health, incidents, scans, reports | An incident review and a new intent when a fix needs design                   | Read-only diagnosis, `/healthalyst-sdlc:review-incident`                                  | The developer on duty triages                                                        |

Small fixes (a typo, a one-file bug fix, a dependency update) start at
stage 3 with no change record; the pull request explains them.

### 1. Plan

- Capture the problem once, in the originator's words: who has it, the
  outcome they want, affected users and systems, constraints, open
  questions. No screens or code yet.
- Anyone can originate, including non-engineers, using the **Change
  request** issue form or Claude on the web.

### 2. Design

- The specification describes behaviour, every screen's loading, empty,
  error, unauthorised and success states, the API contract, validation,
  accessibility, privacy and offline needs.
- Claude must list conflicts between the intent and the rules as
  **Concerns** for a named person to decide. Settle concerns before
  accepting.
- A change that collects, shows, stores or sends personal or health data in
  a new way needs a data protection impact assessment decision; see
  [secure development](secure-development.md#privacy-by-design).

### 3. Build

- Plan in plan mode. The plan names the files that change, the order of
  work, the risks, rejected alternatives and how each step is proven. It is
  ready when someone who never saw the conversation could build from it.
- **Nothing is built without an accepted plan.** If the work departs from
  it, update `plan.md` in the same commit.
- After the plan is accepted, Claude may apply edits without asking for
  each one, for routine work with good tests.
- Run at most two or three Claude sessions in parallel, each in its own
  worktree, split by the files they touch.
- When Claude makes the same mistake twice, add the correction to
  **Things Claude gets wrong** in the repository's `CLAUDE.md`.

### 4. Test

- Claude proves its own work before a person sees it: the repository's
  checks, a failing-test-first fix for bugs, and every changed screen
  exercised at desktop and mobile widths with screenshots.
- A fresh-context `verifier` checks the change against the specification
  and plan, so its verdict is not coloured by the reasoning that produced
  the code.
- Report only checks that ran. "Not checked" is an honest and required
  section of every pull request.

### 5. Deploy

- Every change arrives as a pull request into `dev`. `main` only takes
  `dev`, through its own release pull request. Rulesets block direct pushes,
  force pushes and deletion of both.
- Claude reviews each pull request against the repository's `REVIEW.md`.
  Its comments inform; a code owner approves. With a single developer, the
  administrator bypass is used and recorded; a second reviewer replaces it
  as soon as the team grows.
- Until the Anthropic API key is in place, the `Claude review` check is a
  manual review gate. It fails until the author ticks the **Manual review**
  box in the pull request and writes the findings, or until someone other
  than the author approves the latest commit. Review against `REVIEW.md`, or
  run `/healthalyst-sdlc:verify-change` and the `security-reviewer` agent.
- Before merging `dev` into `main`, run `/healthalyst-sdlc:prepare-release`.
  Each push to `main` creates a release record, which is the deployment
  history.
- Agents never merge, never deploy to production without the release
  manager's approval, and never hold production credentials.

### 6. Maintain

- **CI health:** a daily, plain-arithmetic check compares the CI failure
  rate with its own baseline. A sustained rise opens an issue and Claude adds
  a read-only diagnosis.
- **Incidents:** every production incident gets an `incident` issue, a
  blameless review in `docs/incidents/`, and something that catches it next
  time: a test, an eval, a `CLAUDE.md` line, a review rule or a lint rule.
- **Scans:** CodeQL, Dependabot, dependency review, secret scanning and
  OpenSSF Scorecard run continuously; fix findings within the targets in
  [secure development](secure-development.md#vulnerabilities).
- **Agent evals:** each repository keeps real tasks with known good outcomes
  in `evals/` and runs them whenever its instructions or Claude settings
  change, so a change that makes Claude worse is caught before it merges.

## Branches and environments

| Branch                                          | Purpose                     | Changes through                                        |
| ----------------------------------------------- | --------------------------- | ------------------------------------------------------ |
| Feature branches (`feat/…`, `fix/…`, `chore/…`) | One change each             | Developers and Claude                                  |
| `dev`                                           | Integration and preview     | Reviewed pull requests                                 |
| `staging` (backend only)                        | Pre-production verification | Reviewed pull requests from `dev`                      |
| `main`                                          | Production                  | Reviewed release pull requests from `dev` or `staging` |

## Definition of done

- The accepted plan was followed, or updated where the work departed.
- The repository's checks pass, and the pull request lists what ran and what
  did not.
- Every new or changed screen has a screenshot, and was checked at mobile and
  desktop widths for keyboard use and errors.
- The verifier passed, and the security reviewer passed for sensitive
  changes.
- A code owner approved, and every review conversation is resolved.

## Measures

Review these monthly as a team, to start conversations rather than as
targets. Each repository's **Delivery metrics** workflow calculates the
first five.

| Measure                         | What it shows                                              |
| ------------------------------- | ---------------------------------------------------------- |
| Deployment frequency            | How often changes reach production                         |
| Change lead time                | Time from opening a pull request to it reaching production |
| Change fail rate                | Share of releases followed by an incident                  |
| Failed deployment recovery time | Time from an incident opening to it closing                |
| Deployment rework rate          | Share of releases that were only unplanned fixes           |
| First-pass CI success           | Share of pull requests whose first push passed CI          |
| Repeated mistakes               | Corrections added to `CLAUDE.md` for the same mistake      |
| Incident to eval                | Time from an incident to a permanent eval or test          |

## Adopting it

Start with what needs nothing else, then add the rest:

1. `CLAUDE.md`, the plugin's guardrail hooks, and `/healthalyst-sdlc:verify-change`
   on every change.
2. `/healthalyst-sdlc:write-plan` in plan mode for anything bigger than a
   small fix.
3. Rulesets, CodeQL, dependency review and secret scanning (see the
   [repository standard](repository-standard.md)).
4. Intents and specifications for new features.
5. Claude review, agent evals, CI health and delivery metrics once an
   Anthropic API key is in place. Until then, Claude review runs as a manual
   review gate.
