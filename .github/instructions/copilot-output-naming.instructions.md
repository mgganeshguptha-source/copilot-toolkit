---
applyTo: "**"
---

# Copilot Output File Naming Conventions

When generating any of the following artefacts, always use the exact
file name, location, and format specified below. Do not invent
alternative names or locations.

---

## Context File (build-context skill output)

- **Location:** `.github/story-context-files/`
- **Format:** `STORY-DESCRIPTION-context-YYMMDD-HHMMSS.md`
  where STORY-DESCRIPTION is a short lowercase hyphenated summary of the
  story (2–4 words maximum), and the timestamp is the date and time of
  generation in `YYMMDD-HHMMSS` format.
- **Never overwrite** an existing context file — the timestamp ensures
  each story's context is preserved even on the same branch.
- **Example:** `.github/story-context-files/doctor-removal-context-260517-143022.md`

---

## Prompt Steps File (build-prompt-steps skill output)

- **Location:** `.github/story-prompt-steps/`
- **Format:** `STORY-DESCRIPTION-prompt-steps-YYMMDD-HHMMSS.md`
  where STORY-DESCRIPTION matches the description used in the
  corresponding context file for that story, so the two files pair
  visibly, and the timestamp is in `YYMMDD-HHMMSS` format.
- **Never overwrite** an existing prompt steps file — the timestamp
  ensures each plan is preserved.
- **Example:** `.github/story-prompt-steps/doctor-removal-prompt-steps-260517-143022.md`

---

## Copilot Instructions File (repo-wide)

- **Location:** `.github/`
- **Format:** `copilot-instructions.md`
- **Fixed name** — do not rename or add suffixes.
- **Example:** `.github/copilot-instructions.md`

---

## Path-Specific Instruction Files (rules)

- **Location:** `.github/instructions/`
- **Format:** `NAME.instructions.md`
  where NAME describes the scope of the rules (language, layer, or tool).
- **Must include** an `applyTo` frontmatter block with a glob pattern.
- **Examples:**
  - `.github/instructions/java-conventions.instructions.md`
  - `.github/instructions/java-testing.instructions.md`
  - `.github/instructions/angular-conventions.instructions.md`

---

## Skill Files

- **Location:** `.github/skills/SKILL-NAME/`
- **Format:** `skill.md` (always lowercase, always this exact name)
- **Assets** (templates, examples) go in `.github/skills/SKILL-NAME/assets/`
- **Examples:**
  - `.github/skills/build-context/skill.md`
  - `.github/skills/build-context/assets/context-template.md`
  - `.github/skills/build-prompt-steps/skill.md`

---

## Pairing Convention

The context file and its prompt steps file for the same story must use
the **same STORY-DESCRIPTION**, so they are immediately recognisable as
a pair:

```
.github/
  story-context-files/
    doctor-removal-context-260517-143022.md
  story-prompt-steps/
    doctor-removal-prompt-steps-260517-143022.md
```

The timestamps will differ (the prompt steps file is generated later
than the context file) — that is expected. The STORY-DESCRIPTION is the
shared key, not the timestamp.

---

## Summary Table

| Artefact | Location | Filename format |
|---|---|---|
| Context file | `.github/story-context-files/` | `STORY-DESCRIPTION-context-YYMMDD-HHMMSS.md` |
| Prompt steps | `.github/story-prompt-steps/` | `STORY-DESCRIPTION-prompt-steps-YYMMDD-HHMMSS.md` |
| Repo-wide instructions | `.github/` | `copilot-instructions.md` |
| Path-specific rules | `.github/instructions/` | `NAME.instructions.md` |
| Skill entry point | `.github/skills/SKILL-NAME/` | `skill.md` |
| Skill assets | `.github/skills/SKILL-NAME/assets/` | any descriptive name |
