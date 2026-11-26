---
name: code-review-backend
description: >
  General code review for Java / Spring Boot (WebFlux, reactive) backend
  changes: reuse of existing methods, following the changed class's existing
  patterns, and the backend instruction standards. Use in the harness
  code_review phase, or when a developer asks to "review this change", "check
  my PR" or "does this follow our standards" on .java files.

  Backend only. Security and PHI checks are in security-review-backend.
  Angular code uses review-angular-code.
---

# Code Review — Java / Spring Boot backend

Reviews a backend change for duplicated or bypassed code and for the team's
backend standards. Security and PHI are reviewed by `security-review-backend`;
do not repeat those findings here.

**Review the change, not the whole service.** Use the diff, the full content of
the changed files and the plan. In the harness `code_review` phase that material
is inlined and file reads are disabled — judge only what is in front of you, and
never claim a method exists unless you can see it in that material.

## 1. Reuse — blocking (class `REUSE`)

These are the checks behind the class. Raise each as a blocking `REUSE` issue
only when the evidence is visible in the review material.

- **New method duplicates an existing one.** The change adds a method whose job
  an existing method already does — in the same class, or in a changed class
  whose full content is shown. Example: a new `invokeGetBinaryRequest(...)` on a
  shared request class that already has `invokeGetRequest(...)` able to make the
  same call.
- **New method on a shared or common class without a reason in the plan.** The
  change adds a method to a shared/common class (common request/client classes,
  base classes, shared utils, anything other services or modules also use) and
  the plan's Pre-flight does not name the existing methods considered and why
  none fits.
- **Existing handling bypassed in the same class.** The changed class already
  does a job a certain way — calls downstream through a given method, wraps the
  result in a given type (e.g. `WrappedResponse`), null-checks the wrapped
  response (`getResponse() != null`), maps errors — and the new code in that
  class does the same job without it, so the existing safeguard is skipped.

Each `REUSE` issue names the new code, the existing method or pattern it should
use (file and method name, both visible in the material), and the fix.

**Not REUSE** (write a `[NOTE]` at most): two correct ways of writing the same
logic where nothing is skipped — fluent chaining vs local variables, a stream vs
a loop, naming, ordering. A difference in style is never a blocking reuse issue.

## 2. Standards — usually non-blocking

Check the change against the backend instructions loaded for this phase. A
finding that is also a blocking class elsewhere keeps that class:

| Finding | Class |
|---|---|
| `.block()`, `.collectList()`, business logic inside `doOnNext`/`doOnError`, `subscribe()` inside a chain | `REACTIVE` |
| Wrong response envelope (not `Response<T>`), `200` with a failure body, envelope status not matching HTTP status | `CONTRACT` |
| Exception swallowed, cause lost, broad `catch (Exception)`, local `@ControllerAdvice` without a service-specific reason | `ERROR_HANDLING` |
| Service-level retry added (`.retry()`, `.retryWhen()`, Spring Retry, Resilience4j `Retry`) without the story asking for it | `CORRECTNESS` |
| Cache added, renamed or re-keyed that the plan's cache plan does not list | `CORRECTNESS` |
| `@Log4j2`, logger by hand where `@Slf4j` is used, `Dto` suffix, `I`-prefixed interface, missing method comment, commented-out code, magic values that belong in config | `[NOTE]` |

## Output

**In a harness phase,** follow the phase's own instructions for the output file
and format. A blocking finding is one line:

```
[ISSUE]: (REUSE) <file>:<method> — <what duplicates or bypasses what> — <use this existing method/pattern instead>
```

Other blocking classes use the same shape with their class. Non-blocking
findings are `[NOTE]:` lines.

**Interactive use:** list blocking findings first, then notes, each with
file, method or line, the problem and the fix. End with the standards sections
checked that had no finding.
