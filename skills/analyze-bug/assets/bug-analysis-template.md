# Bug analysis — <story id>: <one-line title>

<!--
TEMPLATE RULES
- Keep every section, in this order, with these headings.
- Cite code as repo/relative/path.ext:LINE. Cite only files that were read.
- "None found" always names what was searched.
- Do not add a commit id or branch: the harness records the analysed source
  above section 1 when it publishes this document.
- Remove these comment blocks and all <placeholder> text from the output.
-->

## 1. Symptom

- **Expected:** <what should happen>
- **Actual:** <what happens; the exact error text>
- **Where:** <endpoint / screen / job>
- **Who is affected and how often:** <always / under which conditions>

## 2. Evidence supplied

| Evidence | What it shows |
|---|---|
| <stack trace / log extract / reproduction steps / screenshot / payload> | <one line> |

**Not supplied:** <evidence that is absent and would have helped, or "Nothing missing">

## 3. Execution path

From the entry point to the failure, one step per line.

1. `<path:line>` — <what happens here>
2. `<path:line>` — <what happens here>

## 4. Root cause

- **Defect:** <the specific wrong code and what is wrong with it>
- **Location:** `<path:line>`
- **Trigger:** <the data or sequence that makes the defect show>
- **Why it produces the symptom:** <symptom → path → defect, each link cited>

<If not established: write "Not established." then the candidate causes and
what evidence would settle it. Do not choose one.>

## 5. Alternatives ruled out

| Other possible cause | Ruled out because | Evidence |
|---|---|---|
| <cause> | <reason> | `<path:line>` or the log line |

<Causes the evidence cannot rule out are listed as "Still open" and raised in
section 11.>

## 6. Fix options

### Option A — <short name> (recommended)

- **Change:** <what changes, in words>
- **Symbols touched:** <class.method, field, endpoint, DTO, config key, template>
- **Trade-off:** <what this option costs or risks>

### Option B — <short name>

- **Change:** <what changes>
- **Symbols touched:** <...>
- **Trade-off:** <...>

**Recommendation:** <which option and why>

## 7. Impact analysis (recommended option)

One row per affected thing. "How found" is the search term or the reference
followed.

### 7.1 Callers and usages

| Symbol touched | Used by | Location | Why affected | How found |
|---|---|---|---|---|
| <symbol> | <caller> | `<path:line>` | <reason> | <search term> |

<Or: `<symbol>: none found (searched "<term>", "<term>")`>

### 7.2 Contracts and shared objects

API responses and requests, the OpenAPI file, DTOs, shared stores (NgRx),
events, configuration keys.

| Object | Change | Location | Consumers |
|---|---|---|---|
| <object> | <what changes, or "unchanged"> | `<path:line>` | <in this repo: list; other repos: "not verified from this repository"> |

### 7.3 Data

Entities, queries, stored data, migrations, existing records affected by the
defect or by the fix.

<items, or "None found (searched ...)">

### 7.4 User interface

Screens, templates, messages, routes.

<items, or "None found (searched ...)" / "Not applicable: backend-only repository">

### 7.5 Other repositories

<Repositories that call or consume the touched code, as far as this repository
shows (client names, base URLs, shared libraries). State clearly: "Not verified
from this repository."  Or: "No contract surface is touched.">

## 8. Regression scope

| # | What to verify | Covers impact item | Method | Detail |
|---|---|---|---|---|
| 1 | The bug itself: fails before the fix, passes after | Root cause | Automated test (to add) | <what it asserts> |
| 2 | <behaviour> | <7.x item> | Automated test (existing) / Automated test (to add) / Manual check / Cannot be verified today | <test class, steps, or why not> |

## 9. Evidence limitations

What could not be established, and why.

- <limit>

## 10. Files and symbols relied on

Every file and symbol the root cause and the impact items rest on.

| File | Symbols |
|---|---|
| `<path>` | <class, methods, fields> |

## 11. Open questions and assumptions

### Blocking questions

<One line per blocking question, in the marker form the analyze-bug skill
defines under "When the evidence is not enough". When nothing blocks, write the
single word `None.` and nothing else here.>

### Assumptions

- <assumption made to proceed, or "None.">

## 12. Lead approval

To be filled by the reviewer. Leave as is.

| Item | Decision | Notes |
|---|---|---|
| Root cause accepted | Yes / No | |
| Fix option chosen | A / B / other | |
| Impact scope accepted | Yes / No | |
| Regression scope accepted | Yes / No | |
| Outstanding risks or exceptions | | |

Reviewed by: <name>, <date>
