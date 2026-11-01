---
name: security-review-angular
description: >
  Security review for Angular web apps, Angular micro-frontends and Ionic /
  Angular mobile apps that display member health data (PHI) under HIPAA. Use
  while writing frontend code (as a checklist) and when reviewing a frontend
  change: the harness coding and code_review phases, or a developer asking for a
  "security review", "OWASP check", "XSS check" or "HIPAA check" on .ts / .html
  files, environment files or package.json.

  Angular / Ionic only. Java / Spring backend code uses security-review-backend.
  Code quality (RxJS, NgRx patterns, control flow) is review-angular-code, not
  this skill.
---

# Security Review — Angular / Ionic frontend

Reviews a frontend change for security and PHI handling. It applies the
standing rules in `hipaa-guardrails.instructions.md`,
`angular-security.instructions.md`, `owasp-angular.instructions.md` and (for
Ionic) `mobile-security.instructions.md` to the code in front of it, and adds
the checks that matter most in PHI-displaying portals and micro-frontends.

**Review the change, not the whole app.** Read the diff and the changed files in
full; open other files only to confirm a finding (e.g. the interceptor that adds
the token, the route that loads a component). A finding needs a file, a line,
and the reason it is unsafe — not a general worry.

## Severity

| Severity | Means | Examples |
|---|---|---|
| CRITICAL | Exploitable now, or PHI leaves its boundary | `bypassSecurityTrustHtml` on server or user data; PHI written to `localStorage`; token or secret in source |
| HIGH | Exploitable with a plausible extra step | Open redirect from a query param; `postMessage` without an origin check; NgRx DevTools enabled in production |
| MEDIUM | Weakens a control | Raw server error shown to the user; PHI in the URL; missing route guard on a member page |
| LOW | Hygiene | `console.log` of non-PHI debug data left in; outdated dependency without a known CVE |

## Checklist

### 1. XSS and DOM safety (A03)
- No `bypassSecurityTrustHtml/Url/ResourceUrl/Script/Style` on data that came
  from the server, the store, a route or user input. Each use must be on a
  constant the code controls, with a comment saying why.
- No `[innerHTML]` with unsanitised data; no `ElementRef.nativeElement.innerHTML`,
  `document.write`, `insertAdjacentHTML`, `eval`, `new Function`.
- No template built from strings (dynamic component templates from data).

### 2. PHI in the browser (HIPAA)
- No PHI in `localStorage`, `sessionStorage`, IndexedDB, cookies set by the app,
  or the service worker cache. For Ionic: no PHI in `Preferences`/plain storage —
  only the secure storage plugin the app already uses.
- No PHI in `console.*` (including `console.log(response)` of a whole DTO), in
  analytics events, or in error-tracking breadcrumbs.
- No PHI in the URL: route params, query strings or fragments (they reach
  browser history, logs and the `Referer` header). Use ids that are not PHI.
- NgRx: no `StoreDevtoolsModule` / `provideStoreDevtools` without
  `logOnly: environment.production` or a production exclusion — DevTools shows
  every state value, including PHI. No state persisted to storage
  (`ngrx-store-localstorage` or similar) for PHI slices.

### 3. Access and navigation (A01)
- A new or changed route that shows member data has the app's auth guard
  (`canActivate` / `canMatch`), like its siblings.
- Redirects after login or from a link use a whitelisted internal path, never a
  raw `returnUrl` / query param (open redirect).
- Data shown is the data the logged-in member may see: the frontend does not
  pick a member id from the URL and fetch it without the backend checking.

### 4. HTTP and tokens (A02, A07)
- Tokens are added by the existing HTTP interceptor, not by hand in components;
  they are never put in URLs, logs, or storage the app does not already use for
  them.
- Calls go to the configured API base URL from `environment.*`, over HTTPS; no
  base URL built from user or route data.
- `withCredentials` and custom headers follow the existing interceptor pattern.

### 5. Micro-frontend / host bridge
- `window.postMessage` receivers check `event.origin` against an allowlist and
  validate the message shape before use. Senders name a target origin — never
  `'*'` when the payload holds PHI or tokens.
- Data received from the host bridge or the shared store is treated as
  untrusted input for rendering (no `bypassSecurityTrust*` on it).
- No PHI on `window` / global objects for other frontends to read.

### 6. Errors shown to the user
- Server error bodies, stack traces and HTTP status texts are not shown raw;
  the user sees the app's standard message. Error details are not logged with
  PHI.

### 7. Configuration and secrets (A05)
- No API keys, client secrets, tokens or passwords in `environment*.ts`,
  `assets/`, or source. (Anything shipped to the browser is public.)
- No new `unsafe-inline` / `unsafe-eval` in the CSP, no disabled Angular
  sanitisation.

### 8. Dependencies (A06, A08)
- New or upgraded packages: no version known to be vulnerable; when unsure, say
  so rather than guessing. No scripts loaded from a CDN without SRI.

### 9. Ionic / mobile only
- Deep links and custom URL schemes validate their parameters before
  navigation.
- No PHI in push notification payloads, app screenshots allowed on sensitive
  screens where the app already blocks them, or plain device storage.

### 10. Test data
- Specs, mocks and fixtures use clearly invented values (`MEM-0001`,
  `Jane Sample`, `2026-01-01`). Values that look like real member data are
  MEDIUM — they may have come from an environment.

## Output

**In a harness phase,** follow the phase's own instructions for the output file
and format. In `code_review`, report each security problem as a blocking line
in the harness format, with the class `SECURITY`:

```
[ISSUE]: (SECURITY) <file>:<line> — <what is unsafe and why> — <the fix>
```

LOW items are `[NOTE]` lines. While **coding**, apply the checklist to the code
you write; do not produce a report.

**Interactive use** (a developer asks for a security review): report in this
order, CRITICAL first.

```
# Security review — <app> — <change or PR>

## Summary
<one paragraph: what was reviewed, counts by severity, overall verdict>

## Findings
| # | Severity | Category | File:line | Finding | Fix |
|---|----------|----------|-----------|---------|-----|

## Checked, no finding
<the checklist sections that were checked and passed>
```

Never quote PHI or a secret in a finding — point to the file and line.
