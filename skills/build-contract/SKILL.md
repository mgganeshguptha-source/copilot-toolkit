---
name: build-contract
description: >
  Writes the hand-off contract a story in one repository produces for the next
  story in another repository — the API response, the store shape, the event
  payload, the schema or the config another repo will read. Use when the harness
  asks for a contract hand-off (documentation phase, or a contract-only run), or
  when a developer asks for "the contract for the next story", "what does this
  change expose", or a consumer-facing description of an API / NgRx store /
  event.

  The contract describes what IS implemented in the change, read from the code —
  never what the story hoped for. Values in it are mocked: no real member data.
---

# Build Contract Skill

A requirement that spans repositories is split into one story per repository,
because the harness changes one repository per run. The downstream story cannot
see the upstream code. If it does not get the contract, it guesses — and a
guess shows up as defensive code that accepts two shapes, which reviewers then
reject. This skill writes the contract that removes the guess.

## Where it goes

The harness gives the exact paths. Normally:

- `.harness/contract/contract-<repo>.md` — the contract (this file's sections)
- `.harness/contract/sample-<repo>.json` — mocked samples, when data crosses
  the boundary: `{"success": {...}, "failure": {...}}`

Do **not** write a header block. The harness stamps one (repo, feature id, run
id, commit, base branch, boundary files), scans both files for personal data,
and publishes them to the `harness-metrics` branch only — never into the PR.

## Read before writing

1. The changed files the harness lists (controllers, DTOs, OpenAPI; NgRx
   actions / reducer / selectors / effects / state; host-bridge services).
2. The types they use, to the leaf fields — a response class that wraps
   another class needs both.
3. The story and its acceptance criteria, only to name what changed and why.

Describe the contract **as implemented**. If you could not confirm something
from the code, write `UNCONFIRMED: <what>` — never fill the gap with what the
story says should be there.

## Sections, in this order

```markdown
# Contract: <one line naming what is provided>

## Boundary type
<REST API | Shared store (NgRx) | Event/message | Shared library/DTO | DB schema | Config/flag>

## Contract
<what a consumer can rely on — see the table below for what each type needs>

## Samples
<a success sample and at least one failure sample, mocked values only;
the same samples go into sample-<repo>.json>

## Negative cases
<each error / empty / not-found case and exactly what a consumer sees>

## Rules for the consumer
<e.g. "Read visit summary only through selectVisitSummaryState.",
"status is a string: SUCCESS | FAILED | NOT_FOUND; there is no numeric code.",
"Handle exactly this shape; do not accept the legacy {body: ...} wrapper.">

## What changed
<compared with the contract before this story: added / renamed / removed /
retyped fields, new status values, new actions>
```

### What "Contract" must contain per boundary type

| Boundary type | Required content |
|---|---|
| REST API | Method and path; base URL per environment if the code shows it; request parameters and body; response schema — every field with type and nullability, nested objects expanded; status codes and when each is returned |
| Shared store (NgRx) | State interface (every field, type, nullability); selectors and what each returns; actions (load / success / failure) and their payloads; what a failure leaves in state; bridge streams a micro-frontend subscribes to and what they emit |
| Event/message | Topic / queue; key; payload schema; ordering and retry behaviour |
| Shared library/DTO | Package and version; changed types with every changed field |
| DB schema | Tables, columns, types, nullability, migration name |
| Config/flag | Key, allowed values, value per environment if known, default |

## Mocked values only

Use invented values that are obviously not real: `Jane Sample`, `MEM-0001`,
`GRP-001`, `2026-01-01`. Never copy a value from a log, a test environment, a
fixture that looks like real data, or the story. Do not include anything that
looks like an SSN, a date of birth, a phone number or an email address — the
harness refuses to publish a contract that does.

## Quality bar

- A developer in the other repo can write the consuming code from this file
  alone, without opening this repo.
- Every field in the samples appears in the Contract section, and vice versa.
- There is exactly one shape per element. If the code still supports an old
  shape for backward compatibility, say so explicitly under "What changed" and
  give both shapes — never leave the consumer to discover it.
- No implementation detail the consumer does not need (internal class names,
  private methods, logging).
