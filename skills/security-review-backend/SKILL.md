---
name: security-review-backend
description: >
  Security review for Java / Spring Boot (WebFlux, reactive) backend services
  that handle member health data (PHI) under HIPAA. Use while writing backend
  code (as a checklist) and when reviewing a backend change: the harness coding
  and code_review phases, or a developer asking for a "security review",
  "OWASP check", "HIPAA check" or "is this endpoint safe" on .java files,
  application*.yml or pom.xml.

  Backend only. Angular and Ionic code use security-review-angular.
---

# Security Review — Java / Spring Boot backend

Reviews a backend change for security and PHI handling. It applies the
standing rules in `hipaa-guardrails.instructions.md` and
`owasp-java.instructions.md` to the code in front of it, and adds the checks
that matter most in reactive, PHI-handling microservices.

**Review the change, not the whole service.** Read the diff and the changed
files in full; open other files only to confirm a finding (e.g. the security
config that protects a new endpoint). A finding needs a file, a line, and the
reason it is unsafe — not a general worry.

## Severity

| Severity | Means | Examples |
|---|---|---|
| CRITICAL | Exploitable now, or PHI leaves its boundary | Missing authorisation on a member endpoint; PHI in logs; SQL/command injection; secret committed |
| HIGH | Exploitable with a plausible extra step | IDOR (path id not checked against the caller); SSRF via a user-controlled URL; stack trace returned to the client |
| MEDIUM | Weakens a control | Missing input validation; over-broad response (more PHI than the contract needs); CORS too wide |
| LOW | Hygiene | Missing correlation id in a log; TODO left on a security-relevant branch |

## Checklist

### 1. Access control (A01)
- Every new or changed endpoint is covered by the service's security
  configuration — check the `SecurityWebFilterChain` / path matchers or
  method-level `@PreAuthorize`, not just the controller.
- **IDOR:** an id from the path, query or body (`personId`, `memberId`,
  `contractNumber`, `groupNumber`) is checked against the authenticated
  principal or the token claims before data is fetched or returned. A
  controller that passes `{personId}` straight to a downstream call with no
  ownership check is HIGH.
- No `permitAll()`, disabled filters, or security config commented out.

### 2. PHI in logs, errors and telemetry (HIPAA)
- No PHI in log statements — including inside reactive operators:
  `doOnNext(r -> log.info("{}", r))`, `doOnError(e -> log.error("...", request))`,
  `.log()` on a `Mono`/`Flux` carrying a DTO. Log ids and status only.
- No DTO `toString()` in logs (Lombok `@Data`/`@ToString` on a PHI-bearing
  class makes every log of it a leak).
- Error responses carry a sanitised message and code (`SystemMessage`,
  `ApiErrorResponse`), never the exception message, stack trace or the
  downstream response body.
- Request/response bodies of PHI endpoints are not logged by filters or
  `WebClient` logging (`ExchangeFilterFunction`, wiretap, DEBUG on
  `reactor.netty.http`).

### 3. Minimum necessary
- The response returns only the fields the contract needs. A new field that
  exposes more member data than the story requires is MEDIUM.
- Mapping from a downstream response does not pass the whole object through
  (`BeanUtils.copyProperties`, returning the client model as the API model).

### 4. Input validation and injection (A03)
- `@RequestBody` DTOs are `@Valid`, with constraints on the fields used.
- Path/query/header values used in a downstream URL are encoded with
  `UriBuilder` variables — never string concatenation into a URL path.
- No string-built SQL/JPQL, LDAP, SpEL or shell commands with request data.
- Headers written to responses are free of CR/LF from input.

### 5. Outbound calls (SSRF, A10) — reactive WebClient
- Base URLs come from configuration, never from request data.
- Calls go through the service's established client
  (`AbstractCommonService` / the common-layer client), not a hand-rolled
  `WebClient` that skips its auth and timeout settings.
- Timeouts are set; a downstream error is mapped (`onErrorResume`/`onErrorMap`)
  to a sanitised response, not passed through with its body.
- TLS verification is never disabled (`InsecureTrustManagerFactory`,
  trust-all `SslContext`).

### 6. Reactive safety that becomes a security problem
- No `.block()` / `.toFuture().get()` on a request thread (thread starvation
  is a denial of service).
- Security context and MDC/correlation id survive operator boundaries
  (Reactor `Context`, not `ThreadLocal`), so audit logs name the right caller.

### 7. Secrets and configuration (A02, A05)
- No secrets, tokens, passwords or keys in code, `application*.yml`, tests or
  `pom.xml`. Values come from the environment / vault placeholders.
- Actuator endpoints are not newly exposed without authentication.
- CORS is an explicit allowlist; no `*` with credentials.

### 8. Deserialisation and dependencies (A06, A08)
- No polymorphic Jackson typing without an allowlist; no `ObjectInputStream`.
- New or changed dependencies: no version known to be vulnerable; when unsure,
  say so rather than guessing.

### 9. Test data
- Test fixtures and JSON test resources use clearly invented values
  (`MEM-0001`, `Jane Sample`, `2026-01-01`). Values that look like real member
  data (real-looking names, member ids, dates of birth, phone numbers) are
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
# Security review — <service> — <change or PR>

## Summary
<one paragraph: what was reviewed, counts by severity, overall verdict>

## Findings
| # | Severity | Category | File:line | Finding | Fix |
|---|----------|----------|-----------|---------|-----|

## Checked, no finding
<the checklist sections that were checked and passed>
```

Never quote PHI or a secret in a finding — point to the file and line.
