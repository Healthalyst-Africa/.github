# HealthAlyst Africa engineering defaults

This repository holds what every HealthAlyst Africa repository shares:

- **Default community files.** GitHub uses [`SECURITY.md`](SECURITY.md),
  [`CONTRIBUTING.md`](CONTRIBUTING.md), the
  [pull request template](PULL_REQUEST_TEMPLATE.md) and the
  [issue forms](ISSUE_TEMPLATE/) for any organisation repository that has none
  of its own.
- **The [engineering handbook](engineering/).** How HealthAlyst plans, builds,
  reviews, releases and maintains software with Claude, and the standard
  every repository meets.
- **Reusable workflows** in [`.github/workflows/`](.github/workflows/):
  - `claude-code-review.yml` reviews pull requests against each repository's
    `REVIEW.md`.
  - `claude-mention.yml` answers `@claude` requests.

The shared Claude Code skills, agents and guardrail hooks live in
[Healthalyst-Africa/claude-plugins](https://github.com/Healthalyst-Africa/claude-plugins).

## Using the reusable workflows

```yaml
jobs:
  review:
    permissions:
      contents: read
      issues: read
      pull-requests: write
      id-token: write
    uses: Healthalyst-Africa/.github/.github/workflows/claude-code-review.yml@<commit-sha> # <version>
    secrets:
      ANTHROPIC_API_KEY: ${{ secrets.ANTHROPIC_API_KEY }}
```

Pin the reference to a full commit SHA with a version comment, as for any
other action. Both workflows also need the
[Claude GitHub app](https://github.com/apps/claude) installed on the
repository.

When `ANTHROPIC_API_KEY` is not set, `claude-mention.yml` skips with a notice,
and `claude-code-review.yml` becomes a **manual review gate**. Its check fails
until either:

- the pull request description's **Manual review** section has its box ticked
  and the findings written, or
- someone other than the author approves the latest commit.

Make the `review / Claude review` check required in the rulesets so nothing
merges unreviewed. So that ticking the box or approving re-runs the check, the
caller must also trigger on these events:

```yaml
on:
  pull_request:
    types: [opened, synchronize, ready_for_review, reopened, edited]
  pull_request_review:
    types: [submitted, dismissed]
```

Pass `require_manual_review: false` under `with:` to keep the old behaviour,
which skips with a notice.

## Changing this repository

Changes here affect every repository in the organisation. They go through a
pull request reviewed by an organisation owner. Keep this repository public:
GitHub only applies default community files from a public `.github`
repository.
