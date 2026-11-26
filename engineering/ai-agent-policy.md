# AI agent policy

How HealthAlyst uses Claude and other AI assistants to build software. It
follows Anthropic's
[enterprise guide to securing coding agents](https://claude.com/resources/articles/enterprise-guide-to-securing-coding-agents):
agents act within fixed boundaries, and people stay accountable for every
change.

## Approved tools

- **Claude Code**, in the terminal, desktop app, IDE extensions and on the web,
  signed in with a HealthAlyst account.
- **The Claude GitHub app and Claude Code GitHub Action**, for pull request
  review and `@claude` requests, using the organisation's API key.
- **Plugins, skills, MCP servers and other agent extensions** load before any
  work starts, so they are dependencies. Use the HealthAlyst plugin and
  Anthropic's official plugins; anything else needs the lead developer's
  approval in a pull request before a repository enables it.

## Data rules

- Never put patient records, personal details of patients or staff,
  credentials, tokens or production data into a prompt, issue, commit, test
  fixture or screenshot.
- Use synthetic test data. Describe real data by type ("a patient's phone
  number"), never by value.
- Production data never sits on a developer machine or in an agent session.
- Local environment files stay unread by agents; read the matching
  `.env.example` for variable names instead.

## What agents may and may not do

| Agents may                                                           | Agents may not                                                                 |
| -------------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Read the repository, run tests, linters and builds                   | Read environment files, private keys or credential files                       |
| Edit code on a feature branch after a plan is accepted               | Push to `main`, `dev` or `staging`, or force-push without a lease              |
| Open a pull request when a person asks                               | Approve or merge any pull request, including their own                         |
| Comment on pull requests and issues                                  | Skip checks with `--no-verify` or weaken a test, lint rule or type to pass     |
| Diagnose CI failures and incidents read-only                         | Deploy to production without the release manager's approval                    |
| Draft intents, specifications, plans, decisions and incident reviews | Hold production credentials or change cloud, DNS, identity or billing settings |

The HealthAlyst plugin's hooks and each repository's settings enforce the
right-hand column. Native Windows has no Claude Code sandbox, so on Windows
these hooks and deny rules are the enforcement layer; never disable them, and
never use bypass-permissions mode.

## Identity and credentials

- Locally, Claude acts with the developer's own access, which is why the
  guardrails above exist.
- In GitHub Actions, Claude acts as the Claude GitHub app, so its comments
  and commits are attributed to the app, separate from the person who
  triggered it.
- The organisation's Anthropic API key is stored only as a GitHub Actions
  secret, with a spending limit. Rotate it immediately if it is exposed, and
  at least yearly.
- Agents never receive deployment tokens or production credentials. Vercel
  deploys `main` itself.

## Accountability

- The person who opens a pull request is accountable for all of it,
  including what Claude wrote. Tick **AI assistance** in the pull request and
  read every line before asking for review.
- Claude's review informs the code owner; it never replaces their approval.
- Report agent behaviour that broke these rules as an incident, so a hook,
  rule or eval can catch it next time.

## Developer setup

1. Install Claude Code and sign in with your HealthAlyst account.
2. In each repository, install the HealthAlyst plugin once:

   ```sh
   claude plugin marketplace add Healthalyst-Africa/claude-plugins
   claude plugin install healthalyst-sdlc@healthalyst --scope project
   ```

3. Keep automatic updates on. Do not override the repository's
   `.claude/settings.json` in `.claude/settings.local.json` to loosen it.
4. Optional, for measuring adoption: set the OpenTelemetry variables
   (`CLAUDE_CODE_ENABLE_TELEMETRY=1` and an exporter) in your user settings
   or shell, never in project settings. Never enable prompt logging.

## When HealthAlyst moves to a Team or Enterprise plan

An administrator applies [`claude-managed-settings.json`](claude-managed-settings.json)
from the Claude admin console, so nobody can turn the baseline off locally:
secret reads denied, bypass mode disabled, the HealthAlyst plugin forced on
and plugin sources restricted to Anthropic's and HealthAlyst's. Managed Code
Review can then replace the GitHub Action review (it reads `REVIEW.md`
natively), and Claude Security can scan repositories on a schedule. Check
each setting against the
[current settings reference](https://code.claude.com/docs/en/settings) before
applying it.
