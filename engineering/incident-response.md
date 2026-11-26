# Incident response

What to do when production breaks, a release fails, or data may have been
exposed. Act first, then review without blame.

## Severity

| Severity   | Meaning                                                           | Examples                                                                             |
| ---------- | ----------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| 1 Critical | Many users cannot work, or personal or health data may be exposed | An application is down; a page shows another organisation's data; a secret is leaked |
| 2 Major    | A key task is broken for some users, with no workaround           | Booking fails for one branch; sign-in fails on one browser                           |
| 3 Minor    | Degraded, with a workaround                                       | A slow page; a broken filter                                                         |

## Handling an incident

1. **Declare it.** Open an issue labelled `incident` with the severity, what
   users see, and when it started. Never put personal data, patient data,
   credentials or tokens in the issue.
2. **Choose an incident lead.** They coordinate and keep the issue updated;
   in a small team, that is whoever noticed first until someone takes over.
3. **Contain it.** Prefer the fastest safe step:
   - roll back by promoting the previous production deployment in Vercel,
     then revert the change on `main` through a pull request;
   - turn the feature off, or revoke and rotate an exposed secret.
4. **Tell people.** Let affected organisations know what is happening and
   when to expect the next update.
5. **Resolve it** and close the issue with what fixed it.
6. **Review it** within five working days with
   `/healthalyst-sdlc:review-incident`. The review goes in the repository's
   `docs/incidents/` and ends with what will catch it next time.

Claude can help diagnose, read-only: reading logs, failed runs and past
reviews. It never changes production, deploys, or contacts anyone outside the
team.

## Personal data breaches

If personal or health data may have been accessed, changed, lost or
disclosed without authorisation, treat it as severity 1 and tell the data
protection lead at once.

- Under the Nigeria Data Protection Act 2023, the Nigeria Data Protection
  Commission must be notified within **72 hours** of becoming aware of a
  breach that is likely to put people's rights and freedoms at risk.
- If the risk to people is high, tell them without undue delay, in plain
  language, with what they can do to protect themselves.
- Record every breach, including ones not reported, in the breach register:
  what happened, the data involved, the effects and the action taken.
- Preserve evidence: logs, deployment history and access records. Do not
  copy the exposed data anywhere to investigate it.
- The data protection lead decides on notification. Engineering supplies the
  facts quickly; it does not wait for the review.

## Blameless reviews

Reviews look for what in the system allowed the problem: missing checks,
unclear rules, gaps in tests or monitoring. They never blame a person. A
review is finished when something now catches the problem next time: a test,
an agent eval, a `CLAUDE.md` correction, a `REVIEW.md` rule, a lint rule, a
hook or a monitoring alert.
