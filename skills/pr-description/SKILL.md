---
name: pr-description
description: >
  Writes a pull request body that helps a reviewer understand why the change
  exists, see the shape of the change at a glance, and know where to look
  hardest. Use when a developer asks for a PR description or PR body, or when
  the harness runs its raise_pr phase.

  This skill writes TEXT ONLY. It never commits, pushes, opens, edits or merges
  a pull request; whoever invokes it (the harness or the developer) does that.

  It never claims test, build, coverage or acceptance-criteria results. In the
  harness those are added to the body by the engine from the gate outputs, so
  the reviewer can trust them. Outside the harness the developer adds them.
---

# PR Description Skill

Writes the reviewer-facing part of a pull request body to the file the task
names (in the harness: `.harness/pr-body.md`). Follow
`assets/pr-body-template.md` exactly — same headings, same order.

Adapted from two public MIT-licensed approaches: a "why + reviewer notes +
change outline" description (HumanLayer `visual-pr`) and a "visual summary +
merge danger" description (Matt Pocock `pr`). Only their body formats are used.

---

## Inputs

Read, in this order, only what you need:

1. The story and its acceptance criteria (the newest context file named in the
   task, or the story text in the task).
2. The plan (`.harness/prompt-steps.md`) and, if it exists, the design
   (`.harness/design.md`). They say what the change was meant to do and which
   alternatives were rejected.
3. The list of changed files and the diff given in the task. Open a changed
   file only when the diff alone does not show its shape.

Do not survey the repository. Everything the body describes is in the diff.

---

## What to write

### Why
Exactly ONE sentence: the business reason for the change, in plain words. Name
the story id. Not a summary of the code.

### What changed
One to three SMALL visuals — pick the smallest view that shows the point. A
reviewer should understand the shape of the change in under a minute.

Choose only views that actually changed:

| The change is mainly... | Use |
|---|---|
| a new or changed endpoint / API contract | the request and response shape, as a `diff` block or a short table |
| a changed type, DTO, entity or NgRx state | the type shape, as a `diff` block |
| new business logic | 5–15 lines of pseudocode |
| a changed call path (controller → service → client) | a call tree as an indented text block, or a Mermaid `flowchart` |
| several files with distinct roles | a shallow file tree with one comment per file saying its job |
| an Angular / Ionic component change | a component tree with the inputs, outputs and state that matter |

Rules:
- Use a `diff` block when an existing shape changed; show the full shape only
  when most of it is new.
- A Mermaid diagram must stay under about 15 nodes. If it would be bigger, use
  a text call tree instead.
- No file-by-file changelog and no prose walkthrough of the code.
- Every name in a visual (class, method, field, endpoint) must appear in the
  diff or in a file you read. Do not invent names.

### Reviewer notes
One to three bullets on where the reviewer should look hardest: a surprising
decision, a deliberate omission, a compatibility constraint, a migration, a
place where the story was ambiguous and how it was resolved. Cite the
acceptance criterion id when one applies (AC-2). Write `- None.` if there is
nothing.

### Merge risk (AI assessment — reviewer to confirm)
Two bullets, always both:
- **Reversibility:** `Two-way door` (cheap to undo: revert the PR) or
  `One-way door` (hard to undo: data migration, removed or renamed public API
  field, changed contract other repos consume, deleted data) — with one clause
  saying why.
- **Blast radius:** what would break, and for whom, if this change is wrong.
  Name the consumers when the diff shows them (endpoints, other repos, screens).

When unsure between the two doors, choose `One-way door` and say what you are
unsure about. Over-warning costs the reviewer a minute; under-warning can cost
an incident.

---

## What you MUST NOT write

- Test results, build status, coverage figures or acceptance-criteria verdicts.
  Do not write "all tests pass", "build green", "AC-1 met" or any number of
  tests or percentage. The harness appends an Evidence section from its own
  gate outputs. A claim written by the model is not evidence.
- Run times, token counts or credit costs.
- A merge or deploy instruction. A person owns the merge.
- Patient, member or any other personal data from test fixtures or logs.
- Headings other than those in the template. The harness adds its own sections
  below yours; do not try to write them.

---

## Style

Plain, direct language, as one engineer writing to another. Short sentences.
No marketing words ("robust", "seamless", "comprehensive"), no emoji, no
preamble before the first heading.
