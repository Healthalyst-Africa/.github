# Engineering handbook

How HealthAlyst Africa builds software. It applies to every repository in the
organisation; each repository's `AGENTS.md` adds its own rules and never
contradicts these.

| Page                                                                | What it covers                                                                                                                                                         |
| ------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| [Software development lifecycle](software-development-lifecycle.md) | The six stages from idea to production and back, what each produces, who accepts it, and what Claude does at each                                                      |
| [AI agent policy](ai-agent-policy.md)                               | Which AI tools we use, what agents may and may not do, data rules, and who is accountable                                                                              |
| [Secure development](secure-development.md)                         | NIST SSDF practices and where we implement them; dependency, secret and vulnerability rules; privacy by design under the Nigeria Data Protection Act; regulatory scope |
| [Incident response](incident-response.md)                           | Severity levels, handling an incident, personal data breaches, and blameless reviews                                                                                   |
| [Repository standard](repository-standard.md)                       | What every repository must have, and where each repository stands today                                                                                                |
| [`claude-managed-settings.json`](claude-managed-settings.json)      | Organisation-wide Claude Code settings to apply once HealthAlyst uses a Claude Team or Enterprise plan                                                                 |

## How these layers fit together

```text
Organisation  this handbook · default community files · reusable workflows
              Claude plugin (Healthalyst-Africa/claude-plugins): skills, agents, hooks
              managed Claude settings (Team or Enterprise plans)
     │
Repository    AGENTS.md · CLAUDE.md · REVIEW.md · .claude/settings.json
              rulesets · CI · CodeQL · dependency review · Scorecard · evals
     │
Application   apps/<name>/AGENTS.md for rules that apply to one application
     │
Change        docs/changes/<issue>-<name>/ intent.md → spec.md → plan.md
```

A lower layer adds detail; it never weakens a higher one.

## Changing the handbook

Open a pull request in this repository. Changes to policy need an
organisation owner's approval. Record significant decisions as architecture
decision records in the repository they affect.
