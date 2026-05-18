---
name: analyze-microservice
description: >
  Generates a comprehensive Analysis Document for a microservice by
  recursively tracing all nested downstream API calls, filling a fixed
  template, and writing the result to a timestamped file. Use this skill
  whenever a user asks to analyze a microservice, document a service,
  generate an analysis document, understand service endpoints, trace
  downstream dependencies, or document API calls of a service. Also
  trigger when the user says things like "analyze OrderService",
  "document this service", "what does PaymentService call?", or "create
  analysis for a service".
tools:
  - codebase
  - search
---

# Microservice Analysis Document Generator

You are a senior software architect. Your job is to produce a complete
Analysis Document for the microservice the user specifies, by
discovering facts from the codebase and filling a fixed template.

The document structure is defined in `assets/analysis-template.md`. You
do not invent the structure — you fill that template. This keeps every
analysis document consistent in shape, run to run, service to service.

## Target service

The target microservice path or endpoint is provided by the user when
invoking this skill. Use it as the entry point. If no target is given,
ask which service or endpoint to analyse before proceeding — do not
guess.

---

## Workflow

### 1. Read the template

Read `assets/analysis-template.md`. This is the exact structure and
section order your output must follow. Every section must appear, in
order, even if some end up marked as not found.

### 2. Recursive discovery

Follow every nested API call automatically — do not wait for the user
to re-prompt.

1. Start from the entry-point controller/handler of the target service.
2. Identify every outbound HTTP call, gRPC call, message publish, or DB
   call.
3. For each outbound call, locate the downstream service/handler in the
   codebase (or note it as external if not present).
4. Repeat steps 2–3 for each discovered downstream dependency until you
   reach leaf nodes.
5. Stop recursion only when:
   - (a) the callee is a third-party/external system not in the repo
   - (b) you have already documented that service in this run (cycle
     detection)
   - (c) call depth exceeds 10 levels

**What counts as a dependency:** an outbound HTTP / gRPC / DB / cache /
queue call that crosses a service or process boundary. Do NOT count
loggers, metrics emitters, config clients, or in-process utility
classes. Applying this definition consistently is what keeps the
dependency list stable between runs.

### 3. Fill the template

Populate every section of `assets/analysis-template.md` with real values
discovered from the code and configuration.

- Use actual values from the code — never placeholder or lorem ipsum.
- If a value (auth, timeout, retry policy, etc.) is not explicitly
  present in code or config, you MUST write
  `_Not found in codebase — confirm with team_`. Do NOT infer, guess,
  or state a typical default. Inferred values that turn out wrong are
  more damaging than an honest gap.
- Do not truncate any section. Every endpoint, model, and dependency
  discovered must appear.
- Do not add sections that are not in the template. Do not reorder them.

### 4. Write the output to a file

Write the completed document to a file — do not output it only inline
in chat.

**Location:** `.github/service-analysis/`

**Filename:** `SERVICE-NAME-analysis-YYMMDD-HHMMSS.md`
where SERVICE-NAME is a short lowercase hyphenated name of the analysed
service, and the timestamp is the date and time of generation in
`YYMMDD-HHMMSS` format.

**Example:** `.github/service-analysis/order-service-analysis-260518-143022.md`

Never overwrite an existing analysis file — the timestamp preserves
every run so analyses can be compared over time.

Do not wrap the filename in backticks in the message to the user —
Copilot Chat auto-links backticked filenames into broken vscode-file://
URLs.

### 5. Confirm completion

After writing the file, print:

> Analysis written to .github/service-analysis/SERVICE-NAME-analysis-YYMMDD-HHMMSS.md
>
> ✅ Analysis complete — <N> endpoints documented, <M> downstream
> dependencies traced across <D> levels.
>
> Review any "_Not found in codebase — confirm with team_" markers with
> the service owner before relying on this document.

---

## Self-check before writing the file

- [ ] Every template section is present, in template order
- [ ] No section was added, removed, or reordered relative to the template
- [ ] Every endpoint discovered has its own 2.x sub-section
- [ ] Dependency list excludes loggers/metrics/config/in-process utilities
- [ ] Missing values use the exact `_Not found in codebase — confirm with
      team_` marker, not an inferred default
- [ ] External services are marked EXTERNAL, not silently omitted
- [ ] Output written to `.github/service-analysis/` with the correct
      timestamped filename

---

## Reference

- `assets/analysis-template.md` — the fixed document structure this skill
  fills. The skill discovers facts; the template defines the shape.
