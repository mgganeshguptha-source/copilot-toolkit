---
name: analyze-bug
description: >
  Analyses a reported bug WITHOUT fixing it. Produces one bug-analysis document:
  the symptom, the execution path, the root cause with file and line, the causes
  ruled out, fix options, an impact analysis of the recommended fix (callers,
  shared objects, data, other repositories, UI), the regression scope, and what
  could not be established. Use when a developer shares a bug report, a stack
  trace or a log extract and asks why it happens, what a fix would affect, or
  for a root-cause or impact analysis; and when the harness runs its
  bug_analysis phase (analysis mode).

  EVIDENCE FIRST. Every statement about the code cites a file and line that was
  actually read. Every "nothing else uses this" names the search that showed it.

  NO GUESSING. When the evidence does not establish the root cause, the skill
  says so and asks for the specific evidence it needs. A plausible root cause
  that is wrong costs more than a question.

  NO FIX. The skill never edits source, tests, build or configuration files.
  A person reviews the analysis; the fix is made afterwards, from the approved
  document.

  In NON-INTERACTIVE / CI mode (no human to answer), the skill does NOT ask
  questions. It records each blocking gap as a `[NEEDS CLARIFICATION]:` line in
  the open questions section and writes the file. The harness halts on those
  lines.
---

# Analyze Bug Skill

Turns a bug report into a **bug-analysis document** a lead can review and
approve before any code is changed. The document answers two questions:

1. **Why does it happen?** The defect, located in the code, with the path from
   the symptom to it.
2. **What does fixing it affect?** Everything that uses the code a fix would
   change, and how each of those is verified.

Stack assumed: **Java Spring Boot / WebFlux** on the backend, **Angular** (and
Ionic) on the frontend.

The output follows `assets/bug-analysis-template.md` exactly: same sections,
same order, same headings. The method below maps onto those sections.

---

## Why this document exists

Two failures it is designed to prevent:

1. **The confident wrong cause.** A fix is written for a cause that reads well
   and is not the real one. The symptom moves or returns, and the real defect is
   still there.
2. **The missed impact.** The fix is right, and something else that used the
   changed code breaks. It is found in a later test phase, or in production.

A document that names a root cause without a traced path, or says "no other
impact" without saying what was searched, prevents neither. It is worse than no
document, because it looks like the analysis was done.

---

## What this skill never does

- **Never edits** source, test, build or configuration files. Not even an
  obvious one-line fix.
- **Never cites code it did not read.** No file names, line numbers, method
  names or behaviour from memory or from what such code "usually" does.
- **Never names a root cause the evidence does not support.** See "When the
  evidence is not enough".
- **Never fills the Lead approval section.** The reviewer does.
- **Never writes a commit id or branch name.** In the harness these are recorded
  by the harness. Outside it, ask the developer.

---

## Method — in this order

### 1. State the symptom

Expected behaviour against actual behaviour, where it shows (endpoint, screen,
job), for whom, and how often. Use the reporter's own error text, exactly.

### 2. List the evidence supplied

Every item the report and its attachments give you: stack trace, log lines,
reproduction steps, request and response samples, screenshots. For each, one
line on what it shows. Then list what is **absent** (for example: no stack
trace, no log lines around the failure, no request payload).

This list decides how far the analysis can go. Do it before reading code.

### 3. Trace the execution path

Start from the evidence, not from a guess:

- **Stack trace:** start at the top frame in this repository's own code.
- **Error text or log line:** search for the literal text to find where it is
  produced.
- **Endpoint or screen:** start at the controller / route / component and
  follow the call.

Follow the path from the entry point to the failure, one step per line, each
with `path:line`. Stop reading when the path is established; this is not a
survey of the repository.

### 4. Establish the root cause

The root cause is a specific defect in specific code: a wrong condition, a
missing null/empty check, a wrong mapping, a blocking call in a reactive chain,
a subscription that is never cleaned up, a contract the code does not follow.

It must link three things, each cited:

**symptom → execution path → defective code**

If you cannot write that chain with a citation at every link, you do not have
the root cause yet. "The service fails to handle the response correctly" is a
restatement of the symptom, not a cause.

Distinguish the **cause** from the **trigger**: the data or sequence that makes
the defect show. Name both.

### 5. Rule out the alternatives

List the other causes a competent developer would suspect for this symptom, and
for each one say why the evidence rules it out, with a citation (code or log
line). If the evidence cannot separate two causes, do not pick one: keep both
as open, and ask for the evidence that would separate them.

### 6. Describe the fix options

One or more options. For each:

- what changes, in words, with the **symbols it touches** (class, method,
  field, endpoint, DTO, configuration key, template);
- its trade-off;
- a short before/after snippet only where words are unclear.

Recommend one and say why. Prefer the smallest change that removes the cause.
Do not redesign.

### 7. Analyse the impact of the recommended option

This is the part teams most often get wrong. Do it by **searching**, not by
reasoning about what probably uses the code.

For **every symbol the recommended option touches**, search the repository and
record what you find:

| Look for | How |
|---|---|
| Direct callers and usages | Search the method / class / field name |
| Interface and inheritance | Implementations of the interface, overrides of the method, subclasses |
| Injection | Where the type is injected or constructed (constructor parameters, `@Autowired`, `@Bean` methods, Angular providers) |
| Names used as text | The name as a string: JSON property names, `@Value` / configuration keys, route paths, query names, reflection, Angular templates (`.html`), selectors, i18n keys |
| Contracts | The OpenAPI file, DTOs and generated models, NgRx actions / reducers / selectors / effects, event or message payloads |
| Data | Entity mappings, repository queries, SQL or migration files that use the field or table |
| Tests | Existing tests that cover the touched code, and tests that would break |

Then write one impact item per affected thing, grouped under the template's
headings. Each item has: what is affected, where (`path:line`), why it is
affected, and **how it was found** (the search term or the reference followed).

When a search finds nothing, write that explicitly with the search term:
`Callers of OwnerMapper.toDto: none found (searched "toDto(", "OwnerMapper")`.
Never write "no impact" without the search behind it.

**Other repositories.** You can only read this repository (and any reference
repository the harness provides). If the touched code is part of a contract
others consume (an API response, a shared store, an event, a library), say so
and list it as **not verified from this repository**. Do not write "no
consumers".

### 8. Set the regression scope

For each impact item, say how it will be verified. Exactly one of:

- **Automated test (existing):** name the test class.
- **Automated test (to add):** say what it must assert.
- **Manual check:** the steps.
- **Cannot be verified today:** say why.

Include one item for the bug itself: the test that fails before the fix and
passes after it.

### 9. State the limits

What you could not establish, and why: code in another repository, runtime
data, environment configuration, timing, generated classes not in the source
tree, searches that cannot see dynamic references.

### 10. List the files and symbols relied on

Every file and symbol the root cause and the impact items rest on. A later
check compares this list with the code at the time of the fix, so be complete
and exact.

### 11. Open questions and assumptions

Blocking questions and non-blocking assumptions, kept apart. See below.

---

## Evidence rules

- Cite as `repo/relative/path.ext:LINE` (or `:START-END`). One citation per
  claim about the code.
- Quote log lines and error text exactly.
- Cite only what you read in this session. If you did not open the file, you
  cannot cite it.
- "None found" always carries the search term.
- Never copy real personal or health data from logs or test resources into the
  document. Refer to the field, not the value.

---

## When the evidence is not enough

Do not guess. The analysis is blocked when any of these holds:

- The report has no stack trace, no log lines and no reproduction steps, and
  reading the code does not show a defect that produces the symptom.
- The bug depends on **data** you cannot see (a specific record, a payload).
- The bug depends on **environment or configuration** not in the repository.
- The bug depends on **timing or concurrency** (intermittent, load-related) and
  the code path does not show a definite race or leak.
- Two or more causes fit and the evidence cannot separate them.
- The failing path leaves this repository before the failure.

Write each blocking gap as one line in the open questions section:

```
[NEEDS CLARIFICATION]: <the specific evidence or answer needed, and what it would settle>
```

A good question names the exact thing to supply:

- Good: `[NEEDS CLARIFICATION]: the stack trace or the ERROR log lines for the failing GET /owners/{id} request — needed to tell a mapping fault from a downstream timeout`
- Bad: `[NEEDS CLARIFICATION]: more information is needed`

Still fill every section you can. Where a section depends on the missing
evidence, write what is known and that the rest is blocked by the question. In
the Root cause section write "Not established" and the candidate causes, not a
chosen one.

**When nothing blocks**, write `None.` under blocking questions and do not use
the `[NEEDS CLARIFICATION]` marker anywhere in the document.

Assumptions that do not block (for example, "assumed the production
configuration matches `application.yml`") go under Assumptions, plainly stated.

### Interactive mode (IDE chat)

Ask the developer the blocking questions, one at a time, and continue when they
answer. Record each answer in the document.

### Non-interactive / CI mode (the harness)

Ask nothing. Write the `[NEEDS CLARIFICATION]:` lines and write the file. The
harness halts on them and the reporter adds the evidence to the bug report.

---

## Banned phrases

These must not appear in the root cause, impact or regression sections. Each
hides missing work.

| Banned | Replace with |
|---|---|
| "may affect other areas" / "could impact related functionality" | The named area, with `path:line`, or "none found" with the search term |
| "should be tested thoroughly" / "regression test all flows" | The specific tests and manual checks |
| "likely" / "probably" / "seems to" in the Root cause section | The citation that makes it certain, or a blocking question |
| "no impact" / "no other callers" without a search term | "none found (searched …)" |
| "handles it incorrectly" / "does not work properly" | The exact wrong behaviour: input, what the code does, what it should do |

---

## Stack notes

**Spring Boot / WebFlux**

- A stack trace through reactive operators often hides the caller. Find the
  assembly point: the method that builds the `Mono` / `Flux`.
- Check for `.block()`, `.subscribe()` inside a chain, swallowed errors
  (`onErrorResume` returning empty), and missing `switchIfEmpty`.
- Injection hides callers from a name search. Search the **type** name and the
  interface it implements.
- Generated classes (OpenAPI models, MapStruct implementations) may not be in
  the source tree. Trace to the source they are generated from (the OpenAPI
  file, the mapper interface) and say so.
- A changed DTO or response field is a contract change: list the OpenAPI file
  and treat consumers as not verified from this repository.

**Angular / Ionic**

- Template bindings do not show in a TypeScript search. Search `.html` files for
  the property, method, selector and pipe names.
- NgRx: a changed action, state shape or selector affects every reducer, effect
  and component that uses it. Search each by name.
- Check subscriptions without teardown, `async` pipe misuse, and change
  detection (`OnPush`) when the symptom is "the screen does not update".
- Routes, guards and lazy-loaded modules reference components by name or path.
- Shared services and stores consumed by other microfrontends are contracts:
  treat consumers as not verified from this repository.

---

## Preflight — before writing the file

- [ ] Every section of the template is present, in order, with its heading.
- [ ] Root cause links symptom → path → defect, with a citation at each link.
- [ ] Every citation is to a file read in this session.
- [ ] Every symbol the recommended fix touches has an impact entry or an
      explicit "none found (searched …)".
- [ ] Every impact item has a regression item with a verification method.
- [ ] Contract surfaces are listed with consumers marked "not verified from
      this repository".
- [ ] No banned phrase.
- [ ] Blocking questions use the exact `[NEEDS CLARIFICATION]:` form, or the
      section says `None.` and the marker appears nowhere.
- [ ] Lead approval section left unfilled.
- [ ] No source, test, build or configuration file was edited.

## Output

- **In the harness:** write the single file the harness names. Nothing else.
- **In the IDE:** write `stories/<story-id>/<story-id>-bug-analysis.md`, where
  `<story-id>` is the bug's tracker id. Ask for the id if it was not given.
