---
name: build-contract
description: >
  Writes the OPTIONAL short summary of a contract change for the next story in
  another repository. The contract itself is the repo's own files (the OpenAPI
  file for a backend repo, the NgRx store folder for an Angular repo), which the
  harness publishes; this summary only says what changed and how a consumer must
  read it. Use when the harness asks for a contract summary (documentation
  phase), or when a developer asks "what does this change expose" or for a
  consumer-facing note on an API / NgRx store change.
---

# Build Contract Skill

A requirement that spans repositories is split into one story per repository.
The downstream story cannot see the upstream code, so it gets the upstream
repo's **own contract files**:

| Upstream repo | Contract files (published by the harness, or copied by the developer) |
|---|---|
| Backend | The OpenAPI file (e.g. `<service>-openapi-code/src/main/resources/<service>.yml`) |
| Angular | The NgRx store folder `src/app/store/` (state, actions, selectors, reducers, effects) |

These files work whether or not the upstream story went through the harness.
This skill writes only an **optional summary** next to them — the part the
files do not say: what changed, and the rules a consumer must follow.

## Where it goes

The harness gives the exact path, normally `.harness/contract/contract-<repo>.md`.
Do **not** write a header block — the harness stamps one, scans the file for
personal data and publishes it with the contract files to the `harness-metrics`
branch only, never into the PR. Do **not** write samples — the downstream run
generates mocked samples from the OpenAPI schema itself.

## Sections, in this order (keep it short)

```markdown
# Contract: <one line naming what is provided>

## What changed
<added / renamed / removed / retyped fields, new status values, new actions or
selectors — compared with before this story>

## Rules for the consumer
<e.g. "Read visit summary only through selectVisitSummaryState.",
"status is a string: SUCCESS | FAILED | NOT_FOUND.",
"Handle exactly this shape; there is no legacy {body: ...} wrapper.">
```

Describe what **is implemented**, from the changed files. If you cannot confirm
something from the code, write `UNCONFIRMED: <what>`.

## Mocked values only

If you show a value, invent it (`Jane Sample`, `MEM-0001`, `2026-01-01`). Never
copy a value from a log, a test resource, a Postman collection or the story —
they may hold real member data. Nothing that looks like an SSN, a date of birth,
a phone number or an email address: the harness will not publish it.
