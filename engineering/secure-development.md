# Secure development

How HealthAlyst builds security and privacy into its software, mapped to the
[NIST Secure Software Development Framework (SP 800-218)](https://csrc.nist.gov/projects/ssdf)
and measured with [OpenSSF Scorecard](https://scorecard.dev/). HealthAlyst
uses AI coding tools but does not train models, so the generative AI profile
(SP 800-218A) applies mainly as a reminder: treat AI-written code as
untrusted until reviewed.

## Practices and where they live

| SSDF practice                        | What HealthAlyst does                                                                                                       | Where                                                              |
| ------------------------------------ | --------------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------ |
| PO.1 Define security requirements    | Engineering rules, review rules and this page                                                                               | `AGENTS.md`, `REVIEW.md`, this handbook                            |
| PO.2 Roles and responsibilities      | Roles per lifecycle stage; code owners                                                                                      | [Lifecycle](software-development-lifecycle.md#roles), `CODEOWNERS` |
| PO.3 Supporting toolchains           | Pinned toolchains, shared lint and type presets, the HealthAlyst Claude plugin                                              | `package.json`, shared config packages, `.claude/settings.json`    |
| PO.4 Criteria for security checks    | Required checks and code scanning thresholds on protected branches                                                          | Repository rulesets                                                |
| PO.5 Secure development environments | Agent guardrail hooks, secret deny rules, no production data locally                                                        | [AI agent policy](ai-agent-policy.md), the plugin's hooks          |
| PS.1 Protect code from tampering     | Rulesets on `main` and `dev`, pull requests only, no force pushes, actions pinned to commit SHAs, read-only workflow tokens | `.github/settings/`, workflows                                     |
| PS.2 Release integrity               | Releases built by Vercel from reviewed `main`; a release record per deploy                                                  | `release.yml`                                                      |
| PS.3 Archive releases                | GitHub Releases with generated notes                                                                                        | Releases page                                                      |
| PW.1 Secure design                   | Specifications with privacy and security sections; architecture decision records                                            | `docs/changes/`, `docs/adrs/`                                      |
| PW.2 Design review                   | Specification acceptance by the lead developer                                                                              | Change records                                                     |
| PW.4 Reuse well-secured components   | Dependency review on pull requests; Dependabot with a cooldown                                                              | `dependency-review.yml`, `dependabot.yml`                          |
| PW.5 Secure coding                   | Rules in `AGENTS.md`, enforced by lint and hooks                                                                            | `AGENTS.md`, lint configuration                                    |
| PW.6 Secure build configuration      | Strict TypeScript, CI builds with public configuration only                                                                 | Shared presets, `ci.yml`                                           |
| PW.7 Review and analyse code         | Code owner review, Claude review, CodeQL with security-extended queries                                                     | Rulesets, `codeql.yml`                                             |
| PW.8 Test executable code            | Unit tests in CI, browser verification of changed screens                                                                   | `ci.yml`, pull request evidence                                    |
| PW.9 Secure defaults                 | Fail-closed hooks, deny-by-default access checks on the server                                                              | Plugin hooks, backend guards                                       |
| RV.1 Identify vulnerabilities        | Private vulnerability reporting, Dependabot alerts, secret scanning, CodeQL, Scorecard                                      | `SECURITY.md`, repository security settings                        |
| RV.2 Assess and remediate            | Fix targets below                                                                                                           | This page                                                          |
| RV.3 Find root causes                | Blameless incident reviews that add a test, eval, rule or check                                                             | `docs/incidents/`                                                  |

## Dependencies

- Every repository commits its lockfile and installs with it frozen in CI.
- Dependabot proposes updates weekly into `dev`, after a seven-day cooldown so
  a compromised release has time to be caught. Security updates skip the
  cooldown.
- Dependency review blocks pull requests that add a dependency with a known
  high or critical vulnerability, or a licence incompatible with the
  repository's licence.
- A new dependency needs a reason in the pull request: what it does that the
  platform or an existing dependency does not, and that it is maintained.
- GitHub Actions are pinned to full commit SHAs with a version comment.

## Secrets

- Secrets never enter a repository. Only `.env.example` files with names and
  safe placeholder values are committed.
- Secret scanning with push protection is on for every repository. A push
  protection bypass needs a written reason and is reviewed.
- Public values only belong in `NEXT_PUBLIC_*` variables; everything there is
  visible to visitors.
- An exposed secret is revoked and rotated within 24 hours, then handled as
  an incident.

## Vulnerabilities

| Severity | Fix in production within                           |
| -------- | -------------------------------------------------- |
| Critical | 7 days                                             |
| High     | 30 days                                            |
| Medium   | 60 days                                            |
| Low      | 90 days, or a recorded decision to accept the risk |

Dismissing a CodeQL, Dependabot or secret scanning alert always needs a
written reason, so the same finding is not raised again as new.

## Privacy by design

HealthAlyst processes personal data and, through its products, health data,
which the [Nigeria Data Protection Act 2023](https://ndpc.gov.ng/) treats as
sensitive personal data. The NDPC's General Application and Implementation
Directive (GAID) 2025 adds detail on assessments, emerging technology and
breach handling.

- **Minimise:** collect only the fields a feature needs, and say why in the
  specification.
- **Restrict:** decide access on the server for every request; show data
  only to people whose role needs it.
- **Keep it out of side channels:** never send personal or health data to
  logs, analytics, error trackers, service worker caches, URLs or query keys.
- **Retain deliberately:** each kind of stored data has a stated retention
  period and a way to delete it.
- **Know where it lives:** hosting regions for data stores are recorded,
  because transfers out of Nigeria need a lawful basis.

### When a data protection impact assessment is needed

Ask the data protection lead before building, and link the decision in the
pull request, when a change:

- collects a new kind of personal or health data, or uses existing data for
  a new purpose;
- shares personal or health data with a new third party or sends it to a new
  country;
- profiles people, makes automated decisions about them, or uses AI on their
  data;
- processes health data at a larger scale or for a wider group than before.

The pull request template's **Personal or health data** checkbox is the
trigger.

## Regulatory scope

> **Proposed: needs sign-off by HealthAlyst's leadership before it is relied on.**

HealthAlyst's current products (organisation onboarding, scheduling, queues,
billing and administration) support the running of health facilities. They
do not diagnose, treat, monitor or predict any medical condition, so they are
not software as a medical device. The medical device software standards IEC
62304 and ISO 13485, and NAFDAC's software-as-a-medical-device registration,
therefore do not apply today.

This must be reassessed, and recorded as an architecture decision, before
building any feature that:

- triages or prioritises patients from clinical information;
- recommends a diagnosis, treatment or dose;
- interprets vital signs, test results or images.

IEC 62304's second edition, in drafting, may widen its scope to health
software generally; the lead developer watches for its publication.

## OpenSSF Scorecard

Public repositories run Scorecard weekly. The targets, and the known gaps:

| Check                                                                             | Target         | Notes                                                                                                                        |
| --------------------------------------------------------------------------------- | -------------- | ---------------------------------------------------------------------------------------------------------------------------- |
| Branch-Protection, Code-Review                                                    | Pass           | Lowered while a single developer uses the recorded administrator bypass; rises when a second reviewer approves pull requests |
| Token-Permissions, Pinned-Dependencies, Dangerous-Workflow                        | 10             | Read-only workflow tokens, SHA-pinned actions, no untrusted input in scripts                                                 |
| SAST, Dependency-Update-Tool, Security-Policy, CI-Tests, License, Vulnerabilities | 10             | CodeQL, Dependabot, `SECURITY.md`, CI on every pull request                                                                  |
| Signed-Releases, Packaging, SBOM                                                  | Not applicable | The applications are deployed by Vercel, not published as packages                                                           |
| Contributors, Fuzzing, CII-Best-Practices                                         | Not pursued    | Revisit as the team grows                                                                                                    |
