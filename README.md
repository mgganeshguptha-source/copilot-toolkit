# Copilot Toolkit

Centralised GitHub Copilot assets — instructions, skills, and templates — shared across the engineering teams that consume this toolkit.

---

## What's here

- **Instructions** — rules Copilot follows automatically, scoped by file path (security guardrails, conventions, logging, error handling, performance, API design).
- **Skills** — multi-step workflows Copilot can invoke on demand (build context files, drive an AC-to-tested-code loop with human checkpoints, generate analyses, run security reviews).
- **Templates** — fixed-structure markdown that long-form skills fill in.

Built for three stacks:

| Stack | Tech |
|---|---|
| Backend | Spring Boot, Java, Maven/Gradle |
| Web frontend | Angular |
| Mobile frontend | Ionic + Angular + Capacitor |

---

## Folder structure

```
copilot-toolkit/
├── .github/
│   ├── copilot-instructions.md                                    [per-project repo-wide cross-cutting only]
│   │
│   ├── instructions/
│   │   ├── copilot-output-naming.instructions.md                  [file naming + location rules for all skill outputs]
│   │   ├── hipaa-guardrails.instructions.md                       [PHI rules — 18 identifiers, logging, storage, audit]
│   │   ├── prompt-injection-guardrails.instructions.md            [LLM safety — untrusted input, tool calling, output handling]
│   │   ├── logging-standards.instructions.md                      [correlation ID contract, log levels, mandatory fields]
│   │   │
│   │   ├── backend/
│   │   │   ├── java-conventions.instructions.md                   [code style — package structure, naming, formatting]
│   │   │   ├── java-testing.instructions.md                       [test style — JUnit/Mockito patterns, coverage rules]
│   │   │   ├── owasp-java.instructions.md                         [security — OWASP Top 10 for Spring Boot]
│   │   │   ├── logging-java.instructions.md                       [logging — SLF4J, MDC, structured JSON, parameterised logs]
│   │   │   ├── error-handling-java.instructions.md                [error handling — exception hierarchy, @ControllerAdvice, RFC 7807]
│   │   │   ├── api-design.instructions.md                         [REST conventions — URLs, status codes, pagination, versioning]
│   │   │   └── performance-java.instructions.md                   [performance — N+1, transactions, connection pool, async]
│   │   │
│   │   ├── angular-frontend/
│   │   │   ├── angular-conventions.instructions.md                [code style — component/service structure, naming]
│   │   │   ├── angular-testing.instructions.md                    [test style — Karma/Jasmine, TestBed patterns]
│   │   │   ├── owasp-angular.instructions.md                      [security — OWASP Top 10 for Angular]
│   │   │   ├── angular-security.instructions.md                   [browser security — XSS, CSP, sanitization, route guards]
│   │   │   ├── logging-angular.instructions.md                    [logging — console hygiene, LoggerService, Sentry scrubbing]
│   │   │   ├── error-handling-angular.instructions.md             [error handling — HTTP interceptor, ErrorHandler, user UX]
│   │   │   └── performance-angular.instructions.md                [performance — OnPush, trackBy, lazy load, bundle budget]
│   │   │
│   │   └── mobile-frontend/
│   │       ├── ionic-angular-conventions.instructions.md          [code style — Ionic + Capacitor structure]
│   │       ├── ionic-angular-testing.instructions.md              [test style — mobile-specific test patterns]
│   │       ├── mobile-security.instructions.md                    [security — secure storage, cert pinning, deep links, jailbreak]
│   │       ├── error-handling-mobile.instructions.md              [error handling — offline state, plugin errors, network awareness]
│   │       └── performance-mobile.instructions.md                 [performance — startup time, native bridge, virtualization, battery]
│   │
│   └── skills/
│       ├── build-context/                                         [workflow — JIRA story → structured context file]
│       ├── build-prompt-steps/                                    [workflow — context file → numbered execution plan]
│       ├── execute-prompt-steps/                                  [workflow — plan → code, one step at a time with human approval]
│       ├── generate-unit-tests/                                   [workflow — code + ACs → test files, one at a time with AC coverage map]
│       ├── run-and-fix-tests/                                     [workflow — test execution + propose-approve-apply fix loop]
│       ├── build-copilot-instructions-backend/                    [workflow — scan Spring Boot repo → fill copilot-instructions.md]
│       ├── build-copilot-instructions-angular-frontend/           [workflow — scan Angular repo → fill copilot-instructions.md]
│       ├── build-copilot-instructions-mobile-frontend/            [workflow — scan Ionic repo → fill copilot-instructions.md]
│       ├── analyze-microservice/                                  [workflow — recursive dependency trace → analysis document]
│       ├── sequence-diagram-microservice/                         [workflow — recursive trace → Mermaid sequence diagram]
│       ├── review-angular-code/                                   [checklist — review Angular changes (RxJS, NgRx, control flow, null safety…) → findings]
│       └── (security-review/ — not in this copy; the BCBSM toolkit has it, with assets/security-review-template.md)
│
└── README.md
```

**Note:** the `execute-prompt-steps`, `generate-unit-tests`, and `run-and-fix-tests` skills write timestamped log files into per-story output folders under the project repo's `.github/` directory (e.g. `.github/story-execution-logs/`, `.github/story-test-generation-logs/`, `.github/story-test-run-logs/`). These output folders are created on first use by the skills themselves — they do not live in the toolkit repo. See `copilot-output-naming.instructions.md` for the full set of output paths and naming conventions.

---

## The AC-to-tested-code loop

The five story-driven skills compose into one end-to-end loop:

| # | Skill | Input | Output |
|---|---|---|---|
| 1 | `build-context` | JIRA story | Context file capturing **what** the work is |
| 2 | `build-prompt-steps` | Context file | Numbered execution plan |
| 3 | `execute-prompt-steps` | Plan | Production code, one step at a time |
| 4 | `generate-unit-tests` | Code + ACs | Test files mapped explicitly to acceptance criteria |
| 5 | `run-and-fix-tests` | Test files | Green suite via propose-approve-apply fix loop |

Every skill stops at an explicit human checkpoint between actions — one step at a time, one test file at a time, one fix at a time. No silent multi-step runs. The toolkit's testing and convention guardrails apply automatically at every step via path-scoped instruction files.

---

## For developers — how to use this toolkit

### One-time setup (per project repo)

1. **Clone copilot-toolkit** alongside your project repo
   ```
   ~/work/
     copilot-toolkit/        ← this repo
     my-project-repo/        ← your project
   ```

2. **Copy required files** into your project's `.github/` folder (see "What to copy" matrix below).

3. **Add toolkit paths to your project's .gitignore** to prevent committing them:
   ```
   .github/instructions/
   .github/skills/
   .github/copilot-instructions.md
   ```
   *(Enterprise requirement: toolkit content must not land in project Git history. Each developer maintains a local copy.)*

4. **Verify Copilot discovery** — open your project in VS Code, open Copilot Chat, run a test prompt like "review this file" and confirm Copilot references the instructions.

### What to copy

**Always copy (every team, every project):**

- `.github/copilot-instructions.md`
- `.github/instructions/copilot-output-naming.instructions.md`
- `.github/instructions/hipaa-guardrails.instructions.md`
- `.github/instructions/prompt-injection-guardrails.instructions.md`
- `.github/instructions/logging-standards.instructions.md`

**Backend teams also copy:**

- All files under `.github/instructions/backend/`

**Web frontend teams also copy:**

- All files under `.github/instructions/angular-frontend/`

**Mobile teams also copy:**

- All files under `.github/instructions/mobile-frontend/`

**Skills — copy what your team uses:**

| Skill | Who uses it | Trigger phrases |
|---|---|---|
| build-context | All | "build context for story…", "create context file" |
| build-prompt-steps | All | "build prompt steps", "create execution plan" |
| execute-prompt-steps | All | "execute the plan", "run prompt steps", "implement story", "next step" |
| generate-unit-tests | All | "generate tests for…", "write unit tests for the new code" |
| run-and-fix-tests | All | "run the tests", "make tests pass", "fix failing tests" |
| build-copilot-instructions-backend | Backend | "generate copilot instructions for this Spring Boot repo" |
| build-copilot-instructions-angular-frontend | Web | "generate copilot instructions for this Angular repo" |
| build-copilot-instructions-mobile-frontend | Mobile | "generate copilot instructions for this Ionic repo" |
| analyze-microservice | Backend | "analyze [service]", "document this service" |
| sequence-diagram-microservice | Backend | "sequence diagram for [service]" |
| review-angular-code | Web / Mobile | "review my Angular changes", "check this component", "PR review" |
| security-review | All | "security review of [service]", "audit for HIPAA/OWASP" — not in this copy (see tree note) |

**Note on the loop skills (build-context through run-and-fix-tests):** teams adopting one should adopt all five, since each consumes the previous skill's output. Adopting only some breaks the chain.

### Keeping in sync

**Manual process for now.** Auto-sync (VS Code tasks + IntelliJ hooks) is on the roadmap — until then:

1. `git pull` this repo periodically (recommend: weekly, or whenever you see a Slack announcement)
2. Diff your project's `.github/` against the toolkit
3. Copy any updated files
4. Re-test Copilot in your IDE

When in doubt, **re-copy everything from your stack** — instructions and skills are idempotent.

### Troubleshooting

| Symptom | Likely cause |
|---|---|
| Copilot ignores an instruction | File not in `.github/` of the open workspace; `applyTo` glob doesn't match the file you're editing; Copilot session running off stale context — restart Copilot Chat |
| Skill not triggering | Skill folder name doesn't match `name:` in frontmatter; description too vague; user prompt doesn't match a trigger phrase |
| Skill runs but output looks wrong | Template asset missing or out of date; SKILL.md was updated but assets were not re-copied |
| Loop skill says "I need [file] attached" | Loop skills (execute-prompt-steps, generate-unit-tests, run-and-fix-tests) read from chat attachments, not from disk — drag the required files into the chat each invocation |
| Loop skill stopped at "Attempt limit reached" | Working as designed — beyond 3 fix attempts usually means a structural issue; pause, re-think, or loop back to an earlier skill |
| IntelliJ users: path-specific rules not applying | Verify Copilot plugin version supports `applyTo` globs — older versions do not |
| Skill writes to wrong location | `copilot-output-naming.instructions.md` not copied to project repo |

---

## For contributors — how to add or change toolkit content

### Adding a new instruction file

1. Decide scope: repo-wide / backend / angular-frontend / mobile-frontend
2. Place under `.github/instructions/<scope>/NAME.instructions.md`
3. Include `applyTo:` frontmatter with the correct glob
4. Follow established patterns:
   - Short intro stating scope + which other files it compounds with
   - Rules organised under clear `##` section headings
   - "What never to do" list near the bottom
   - "Refusal pattern" at the very bottom
5. Cross-reference related instruction files where relevant
6. Update this README's "What to copy" matrix
7. Open PR for review

### Adding a new skill

1. Folder: `.github/skills/skill-name/` (lowercase-hyphenated)
2. Entry file: `SKILL.md` (uppercase)
3. Frontmatter `name:` must match the folder name exactly
4. Description must be precise — it drives Copilot's auto-triggering. Include 3–5 example trigger phrases
5. If the output is long-form: create `assets/template.md` and have the skill fill it
6. Add output naming to `copilot-output-naming.instructions.md`
7. Add to the skill table in this README
8. Open PR for review

### Design principles (apply to all new content)

- **Discover, don't guess** — read the repo for what's discoverable, ask the developer for genuine gaps
- **Mark unknowns loudly** — use `[TODO — MUST COMPLETE]` or `_Not found in codebase — confirm with team_`, never substitute defaults
- **File output, not inline** — long multi-section artefacts go to timestamped files in `.github/<folder>/`
- **Chat attachments for story-bearing files** — loop skills (build-prompt-steps onwards) read context, plan, and log files from chat attachments, never from disk, so the developer consciously picks which story is in play
- **No filename backticks in developer-facing messages** — Copilot Chat auto-links them into broken `vscode-file://` URLs
- **Template + skill separation** for long-form outputs
- **Lowercase-hyphenated folders, `SKILL.md` uppercase**
- **One question at a time** when interrogating the developer
- **One step / one file / one fix at a time** for loop skills — never plow through multiple actions per invocation
- **Self-check step** before producing output

---

## Versioning

This toolkit uses semantic-ish git tags:

- **Major** (`v2.0.0`) — breaking changes to instruction shape, skill APIs, or output formats
- **Minor** (`v1.1.0`) — new instructions, new skills, non-breaking enhancements
- **Patch** (`v1.0.1`) — typos, clarifications, bug fixes

Check the latest tag: `git tag --sort=-v:refname | head -1`

Developers should note which version their project is synced from. When updating, scan the changelog (see below) for breaking changes.

---

## Compatibility

- **GitHub Copilot Business** or **Enterprise**
- **VS Code** with the Copilot Chat extension (primary, fully supported)
- **IntelliJ IDEA** with the Copilot plugin — path-specific `applyTo` glob support varies by plugin version; verify before mobile team relies on it
- Tested against Copilot Chat as of **[update this date with each release]**

---

## Changelog

> Manual changelog until release tagging is automated. Most recent first.

### Unreleased
- _Add new entries here as work progresses_

### 2026-05-30
- **Added** 3 new skills completing the AC-to-tested-code loop:
  - `execute-prompt-steps` — walks the build-prompt-steps plan one step at a time, with explicit approve / reject-with-comment / skip checkpoints between every step. Recognises and handles all eight step types from the plan (inventory, design decision, implementation, test, frontend wiring, validation, convention drift, non-functional)
  - `generate-unit-tests` — writes unit tests for the production code one test file at a time, with an explicit AC-to-test coverage map approved before any test code is written. Refuses to mock the class under test, refuses `@Disabled` / `xit`, refuses to modify production code (flags smells without fixing)
  - `run-and-fix-tests` — runs the test suite, parses results, and on failure proposes a concrete diff for developer approval before any code change. Configurable attempt limit (default 3, hard ceiling 5) to prevent unbounded fix loops. Refuses green-by-erosion (disabling tests, weakening assertions, flaky-labelling without evidence)
- **Updated** `copilot-output-naming.instructions.md` — added three new output paths (`story-execution-logs`, `story-test-generation-logs`, `story-test-run-logs`); updated pairing convention to show all five story-related files together; extended summary table
- **Updated** README — added "AC-to-tested-code loop" overview section; added loop-skill troubleshooting rows; added design principles for chat attachments and one-action-per-invocation discipline

### 2026-05-24
- **Added** 6 security guardrail instruction files (HIPAA, prompt-injection, OWASP-Java, OWASP-Angular, Angular-security, mobile-security)
- **Added** `security-review` skill + template
- **Added** 9 standard instruction files (shared logging contract, Java/Angular/mobile logging+error-handling, Java API design, Java/Angular/mobile performance)
- **Updated** `copilot-output-naming.instructions.md` — added service-analysis and security-reviews paths; corrected `SKILL.md` capitalisation; formalised `applyTo` frontmatter

### Earlier
- Initial skills: build-context, build-prompt-steps, build-copilot-instructions-* (×3), analyze-microservice, sequence-diagram-microservice
- Initial convention instructions: java-conventions, java-testing, angular-conventions, angular-testing, ionic-angular-conventions, ionic-angular-testing
- Initial output naming conventions

---

## Known limitations

- **Copilot adherence to instructions is probabilistic.** Even with hardened instructions, Copilot can drift — especially over long sessions. Critical guardrails (HIPAA, OWASP, prompt-injection) should be backed by code review and CI security scans, not relied on Copilot alone.
- **The loop skills depend on the developer attaching the right files each invocation.** If the wrong context file is attached, the wrong story gets executed. The skills check for plausibility (filename pairing, expected sections) but cannot prevent a deliberately wrong attachment.
- **IntelliJ path-specific instruction support** is plugin-version-dependent. Mobile team should verify before depending on path-scoped rules.
- **Convention files are first drafts.** `java-conventions`, `angular-conventions`, `ionic-angular-conventions`, and their testing counterparts encode reasonable defaults but are not yet ratified team standards. Each team should review and align them to their actual codebase.
- **Guardrail files are first drafts.** HIPAA, OWASP, security rules are sensible defaults — they should be reviewed by your security and compliance partners before treating them as authoritative.
- **Auto-sync from toolkit to project repos is not built yet.** Manual copy is the current workflow.
- **No automated CI test for skill quality.** Variance run-to-run is inherent; real-repo testing surfaces issues better than synthetic.
- **`sequence-diagram-microservice` output location** (inline vs file) is still undecided.

---

## Support

- **Owner:** [team name / Slack channel]
- **Issues / requests:** [filing process — Jira board, GitHub issues, Slack channel]
- **Office hours:** [if any]
- **Slack:** [channel for questions]

---

## FAQ

**Q: Why isn't Copilot picking up my skill?**
A: Most common: the skill folder name does not match the `name:` field in SKILL.md frontmatter. Both must be lowercase-hyphenated and identical. Also verify `SKILL.md` is uppercase.

**Q: Why does Copilot sometimes ignore an instruction?**
A: Copilot is probabilistic. Common causes: the `applyTo` glob doesn't match the file you're editing; the session has too much accumulated context — restart Copilot Chat; the instruction conflicts with another instruction Copilot weighted higher.

**Q: Why does the loop skill keep asking me for files instead of finding them on disk?**
A: By design. The loop skills (`build-prompt-steps`, `execute-prompt-steps`, `generate-unit-tests`, `run-and-fix-tests`) read from chat attachments only — never disk — so the developer consciously selects which story is in play. This prevents the wrong story's context from being applied to the current code change. Attach the required files each invocation.

**Q: My `run-and-fix-tests` run hit the attempt limit. What now?**
A: Working as designed. Beyond 3 fix attempts on the same failure, the issue is almost always structural (wrong design choice, wrong AC interpretation, missing piece of the implementation) rather than a simple test/code mismatch. The skill stops and offers four options: override the limit (up to 5 hard ceiling), pause to re-think, loop back to `execute-prompt-steps` to redo an earlier step, or loop back to `generate-unit-tests` if the test itself encodes a wrong expectation.

**Q: How do I report a false positive from a guardrail (Copilot refusing something it shouldn't)?**
A: [Slack channel / Jira board]. Include: the prompt, the refusal message, the guardrail file you think is responsible, and the case you think should be allowed.

**Q: Can I add my own instructions on top of the toolkit's?**
A: Yes, in your project repo's `.github/instructions/` add new files alongside the copied ones. Avoid editing copied toolkit files in place — those changes get overwritten on next sync. If your addition is generally useful, contribute it back via PR.

**Q: Does this work for non-Copilot AI assistants?**
A: Instructions and skills here use GitHub Copilot's specific conventions (`applyTo` frontmatter, `SKILL.md` discovery). Other assistants would need their own equivalent.

---

## Roadmap

- **Auto-sync mechanism** — VS Code `tasks.json` (`runOn: folderOpen`) + IntelliJ git hooks; bootstrap committed in project repos, full toolkit content stays in this repo
- **CI eval for skills** — automated check that core skills still produce well-formed output on a fixture repo
- **Loop skill refinement from pilot feedback** — once the Apexon-driven backend / frontend / full-stack pilot runs complete, refine the three new skills based on observed drift, failure modes, and reviewer feedback
- **Decision on `sequence-diagram-microservice`** — inline vs file output
- **Team-specific overrides** — pattern for teams to extend/override toolkit instructions without forking
