# context.md Template — All Story Types & Development Layers

---

## Universal Sections (All Story Types, All Layers)

### Section 1 — What Are We Trying to Achieve
**Required:** Always
**Purpose:** One paragraph — the goal in plain language

| Story Type | Focus |
|---|---|
| Bug Fix | What is broken and why fixing it matters |
| New Dev | The user goal and business value |
| Enhancement | What limitation is being addressed |

**New terms introduced by this story** go here too — a new UI control, a new
state, a new concept that does not yet exist in the codebase. Define each in one
line so the ACs can use the name without ambiguity.

Do **not** restate domain terms that already exist in the codebase. Those live in
a governed instruction file (`applyTo: **`), defined once for every story.
Repeating them per story is how five stories end up with five slightly different
definitions of the same noun.

---

### Section 2 — Current Behaviour
**Required:** Bug Fix and Enhancement only. Skip for New Dev if nothing exists.

**Backend:** `GET /api/v1/owners?lastName=Smith returns all owners regardless of parameter`
**Frontend:** `Appointment form submits without validating the date field — past dates accepted`
**Full Stack:** `Search screen shows all owners on load. API ignores query parameter.`

---

### Section 3 — Expected Behaviour
**Required:** Always — always verify, most commonly vague

**Backend:** `Returns only owners whose lastName contains "Smith" (case-insensitive, partial match). Paginated, default 20.`
**Frontend:** `Past date entry shows inline error below field. Form does not submit. Field highlighted red.`
**Full Stack:** `User types in search box → API filters results → list updates → loading spinner during call → empty state if no results`

---

### Section 4 — Acceptance Criteria
**Required:** Always — minimum 2, ideally 3+. Each must be independently testable.

Three rules apply to every AC. The full reasoning is in the skill under
*"Acceptance criteria — syntax, numbering, and provenance"*; this is the shape.

**1. EARS phrasing.** Pick the simplest pattern that fits. One sentence, at most
three preconditions.

| Pattern | Shape |
|---|---|
| Ubiquitous | THE \<system\> SHALL \<response\> |
| Event-driven | WHEN \<trigger\>, THE \<system\> SHALL \<response\> |
| State-driven | WHILE \<state\>, THE \<system\> SHALL \<response\> |
| Unwanted behaviour | IF \<condition\>, THEN THE \<system\> SHALL \<response\> |
| Optional | WHERE \<feature is present\>, THE \<system\> SHALL \<response\> |

The system name is the component in the reader's language — *the owner search
endpoint*, *the booking form* — never a class or file name.

**2. Stable identifiers.** `AC-1`, `AC-2` … flat; `AC-2.1`, `AC-2.2` when the
story is split into several user stories. On regeneration, never renumber:
append new ones, mark removed ones `AC-4: [WITHDRAWN]`.

**3. Provenance.** Mark any criterion the story never asked for:

| Marker | When |
|---|---|
| *(none)* | Traceable to the story, the developer's answers, or an instruction file |
| `[ASSUMED]` | The model added it; the story never mentioned the topic |
| `[NEEDS CLARIFICATION]` | A requested behaviour is under-specified — goes in Section 8, blocks the harness |

A missing *dimension* of a requested behaviour is `[NEEDS CLARIFICATION]`, never
`[ASSUMED]`. Every `[ASSUMED]` criterion also appears in Section 9.

Include negative criteria (`SHALL NOT`) wherever a wrong implementation is
plausible — an unstated exclusion gets implemented anyway.

---

### Section 5 — Edge Cases
**Required:** Always

**Backend:** `null param, empty string, special chars (O'Brien), very long input, injection attempts`
**Frontend:** `Keyboard submit, screen reader announcement, slow network double-click, mobile keyboard overlap`
**Full Stack:** `Network failure, API 500 error, search during previous search in progress, session expiry`

---

### Section 6 — Constraints
**Required:** Always — auto-filled from copilot-instructions.md

**Backend (auto):** `Constructor injection | Standard error format | Jakarta Validation | JUnit 5 + Mockito | Pagination`
**Frontend (auto):** `Existing component library | Keyboard navigation | ARIA labels | Responsive 375px-1440px | Component tests`
**Full Stack:** Both sets above plus `API contract must not break existing consumers`

Non-functional targets (latency, bundle size, memory) go here — with load
context — never in Acceptance Criteria.

---

### Section 7 — Out of Scope
**Required:** Always — prevents Copilot over-engineering

**Backend:** `firstName search | sorting | performance optimisation | email notification`
**Frontend:** `Full redesign | time zones | cancellation flow`
**Full Stack:** `Advanced filters | export | search history | URL update on search`

---

### Section 8 — Clarifications Needed
**Required:** Only if genuine ambiguities exist

A requested behaviour whose dimension is undecided. **Blocks the harness** until
resolved. Name the missing dimension — never parrot the vague phrase.

**Backend:** `[NEEDS CLARIFICATION]: Match type — partial or exact? | Maximum page size?`
**Frontend:** `[NEEDS CLARIFICATION]: Validation trigger — on blur or on submit? | Error-state design available?`
**Full Stack:** `[NEEDS CLARIFICATION]: Search trigger — keypress or button? | Minimum characters before search fires?`

---

### Section 9 — Assumptions
**Required:** Whenever any AC carries `[ASSUMED]`

Every assumed criterion, restated with its basis, so a reviewer can confirm or
delete it without reading the whole AC list. These do **not** block the harness —
which is exactly why they need to be visible in one place.

```markdown
## Assumptions
- AC-6 — result sort order. Assumed lastName ascending.
  Basis: paginated responses need a deterministic sort (copilot-instructions.md);
  the story specifies none. Confirm or replace before building.
```

If more than a third of the ACs are `[ASSUMED]`, the story is a seed rather than
a specification — refine it before building.

---

## Complete Examples

### Backend Bug Fix
```markdown
## What Are We Trying to Achieve
Fix the owner search API which ignores the lastName parameter
and returns all owners, preventing clinic staff from finding
specific owners efficiently.

## Current Behaviour
GET /api/v1/owners?lastName=Smith returns all owners regardless
of the lastName parameter value.

## Expected Behaviour
Returns only owners whose lastName contains "Smith"
(case-insensitive, partial match). Results paginated, default 20.

## Acceptance Criteria
- AC-1: WHEN a lastName parameter is supplied, THE owner search endpoint SHALL
  return only owners whose lastName contains that value.
- AC-2: THE owner search endpoint SHALL match lastName case-insensitively.
- AC-3: WHEN the lastName parameter is absent or empty, THE owner search
  endpoint SHALL return all owners with status 200.
- AC-4: IF no owner matches the supplied lastName, THEN THE owner search
  endpoint SHALL return status 200 with an empty content array and
  totalElements 0 — not 404.
- AC-5: THE owner search endpoint SHALL return results paginated at 20 per page.
- AC-6: [ASSUMED] THE owner search endpoint SHALL sort results by lastName
  ascending. Basis: paginated responses need a deterministic sort order
  (copilot-instructions.md); the story specifies none.
- AC-7: THE owner search endpoint SHALL NOT match on firstName or any field
  other than lastName.

## Edge Cases
- null parameter: return all owners
- Special characters (O'Brien, García): handled correctly
- Very long input: return 400 validation error

## Constraints
- Constructor injection throughout
- Standard error response format
- Jakarta Validation on parameters
- JUnit 5 + Mockito tests
- Pagination for results

## Out of Scope
- firstName search not in this story
- Sorting configurable by the caller not in scope

## Clarifications Needed
- [NEEDS CLARIFICATION]: Match type — partial (contains) or exact only?
- [NEEDS CLARIFICATION]: Maximum accepted lastName length before 400 —
  the story requires a 400 on very long input but sets no limit.

## Assumptions
- AC-6 — result sort order. Assumed lastName ascending.
  Basis: pagination without a deterministic sort produces unstable pages;
  the story specifies no order. Confirm or replace before building.
```

---

### Frontend New Development
```markdown
## What Are We Trying to Achieve
Build a new appointment booking form so pet owners can
schedule visits directly from the portal without calling.

## Current Behaviour
No booking form exists — owners call the clinic to book.

## Expected Behaviour
Form with: pet selection dropdown, vet dropdown, future-only
date picker, reason text area. On submit: confirmation message.

## Acceptance Criteria
- AC-1: THE booking form SHALL require pet, vet, date, and reason before a
  booking can be submitted.
- AC-2: WHILE any required field is empty, THE booking form SHALL keep the
  Submit button disabled.
- AC-3: THE date picker SHALL offer only dates later than the current date.
- AC-4: THE pet dropdown SHALL list only pets belonging to the logged-in owner.
- AC-5: WHEN the API confirms the booking, THE booking form SHALL display a
  success message.
- AC-6: IF the submit request fails, THEN THE booking form SHALL display an
  error and preserve every value the user entered.
- AC-7: THE booking form SHALL remain usable at 375px and 1440px viewport
  widths.
- AC-8: THE booking form SHALL NOT submit more than one request when Submit is
  pressed repeatedly.

## Edge Cases
- No pets registered: show "Please add a pet first"
- No vets available: show availability message
- Network failure on submit: show error, keep form data
- Double submit: only one request sent

## Constraints
- Follow existing design system components
- Keyboard navigation for all fields
- ARIA labels on all inputs and errors
- Responsive layout
- Component unit tests

## Out of Scope
- Payment processing not in scope
- Appointment cancellation not in scope
- Email confirmation not in scope

## Clarifications Needed
- [NEEDS CLARIFICATION]: Post-submit state — does the user stay on the form,
  see it cleared, or navigate to a confirmation screen?
- [NEEDS CLARIFICATION]: Booking horizon — how far ahead can a date be chosen?
- [NEEDS CLARIFICATION]: Can one submission cover multiple pets?
```

*(No Assumptions section — every criterion here traces to the story or an
instruction file. Not every context needs one.)*

---

### Full Stack Enhancement
```markdown
## What Are We Trying to Achieve
Enhance owner search so results update live as the user types,
removing the need to click Search and speeding up busy workflows.

## Current Behaviour
Owner search requires typing lastName and clicking Search.
API correctly filters by lastName. Results replace full list on click.

## Expected Behaviour
After 3 characters typed, results update automatically with 300ms
debounce. Loading spinner shown. Empty state if no results.
Search button remains for accessibility.

## Acceptance Criteria
- AC-1: WHEN the user has typed 3 or more characters, THE owner search SHALL
  update the results without a Search button click.
- AC-2: THE owner search SHALL debounce input by 300ms so the API is not called
  on every keystroke.
- AC-3: WHILE a search request is in flight, THE owner list SHALL display a
  loading spinner.
- AC-4: IF no owner matches the search term, THEN THE owner list SHALL display
  the empty state "No owners found".
- AC-5: THE Search button SHALL remain functional for keyboard and
  screen-reader users.
- AC-6: WHEN the search input is cleared, THE owner list SHALL restore the full
  owner list.
- AC-7: [ASSUMED] WHILE fewer than 3 characters are present in the search input,
  THE owner list SHALL display the full unfiltered list. Basis: mirrors the
  cleared-input behaviour in AC-6; the story defines only the 3-character
  threshold for searching, not the state below it.
- AC-8: WHILE a new search is in flight, THE owner list SHALL continue showing
  the previous results rather than blanking.

## Edge Cases
- Slow network: previous results stay until new ones arrive
- User types faster than debounce: only last query fires
- Network failure: show "Something went wrong, try again"
- Screen reader: result count announced after update

## Constraints
- API contract unchanged — existing consumers not affected
- Constructor injection throughout backend
- Standard error response format
- 300ms debounce on frontend
- ARIA live region for result updates
- Unit tests for debounce behaviour

## Out of Scope
- Advanced filters not in scope
- Search history not in scope
- URL update on search not in scope

## Clarifications Needed
- [NEEDS CLARIFICATION]: Minimum characters — 2 or 3 before search fires?
- [NEEDS CLARIFICATION]: Mobile trigger — same live search, or button-click?

## Assumptions
- AC-7 — behaviour below the character threshold. Assumed the full list is
  shown. Basis: consistent with clearing the input (AC-6); the story is silent
  on the sub-threshold state. Confirm or replace before building.
```
