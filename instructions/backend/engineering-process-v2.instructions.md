---
applyTo: "**"
---

# Engineering Process — v2

## Static analysis
- Align SonarLint rules with project goals; generate the Sonar report and fix findings.
- Sonar runs in CI with a **custom quality gate**; the build fails when the gate fails. Fortify findings triaged; critical/high fixed. Checkstyle warnings resolved or justified.

## Git
- Branches: `feature/DEV_{Jira story number}`, `bugfix/DEV_{Jira bug number}`, sprint/release branches (`feature/Sprint-138`, `release/MPA-202312`) merging to SIT.
- Do not reverse-merge the sprint branch into a story branch. One branch per story/bug. No bugfix branch for stories.
- Commit message: `<Jira number> - short description`.

## Build and deployment
- New services are onboarded with the infrastructure team to the Jenkins pipeline, JFrog and AWS: merge to `develop` triggers build, publishes the jar to JFrog and deploys.

## Secure-coding checklist (every PR)
- No hard-coded secrets. All SQL parameterised. Input validation/encoding for outbound calls.
- Connect + read timeouts on external calls; circuit breaker/fallback for external SOAP calls.
- No PII in logs. Sonar & Fortify findings triaged.

## PR / code-review checklist
- Description: purpose, components, DB changes, staging steps.
- Package and naming conventions followed; no dead or large commented-out code.
- Logging and correlation/trace id present where appropriate.
- External-call timeouts and retries present.
- Configuration keys documented; Sonar/Checkstyle warnings resolved or justified; Fortify reviewed.
