---
applyTo: "src/main/java/**/*Controller.java,src/main/java/**/controller/**/*.java"
---

# API Design Conventions — REST (Backend)

Conventions for HTTP REST APIs this service exposes. Compounds with `error-handling-java.instructions.md` (response envelope), `openapi-contract-first-v2.instructions.md` (contract and generated interfaces) and `owasp-java.instructions.md` (authZ, injection).

## Contract first

- Follow the OpenAPI **contract-first** approach: write the OpenAPI yml first, generate the controller interface and models, then implement the interface. See `openapi-contract-first-v2.instructions.md`.
- Never write a raw controller with hand-written mapping annotations when a generated `*Api` interface exists.
- Keep all related functional operations in **one controller**; group controller methods by domain object (User, Claims, Copay).

## URL structure

- **Order:** `/<versionInfo>/<functionName>/<params>/.../<operation>`, e.g. `/v1/plandocuments/personId/{personId}/get` (operation = get / update / retrieve).
- **Version prefix** is the major version only (`/v1/...`). It is **not** prefixed with `/api` unless the service's OpenAPI contract says so.
- **Resource names:** **nouns** for resources, **plural** for collections (`/user-management/users`), singular instance via id (`/user-management/users/{id}`). The only verb allowed is the trailing operation segment above.
- **Separators:** use **hyphens**, never underscores.
- **Nesting:** at most one level deep. `/members/{memberId}/claims` is fine. `/members/{memberId}/claims/{claimId}/lines/{lineId}/notes` is too deep — flatten by exposing `/claim-lines/{lineId}/notes`.
- **Identifiers in path:** UUIDs or opaque tokens where the API mints the id. Never expose auto-increment integers (OWASP A01 enumeration risk).
- **Query params:** use query parameters to filter collections. camelCase (`?pageSize=20&sortBy=createdDate`). Filters use plain keys, not bracket syntax.

## HTTP methods — semantic use

| Method | Purpose | Idempotent | Request body | Response body |
|---|---|---|---|---|
| GET | Read | Yes | No | Yes |
| POST | Create / non-idempotent action | No | Yes | Yes (201 + Location header) |
| PUT | Full replace | Yes | Yes (complete resource) | Yes or 204 |
| PATCH | Partial update | No (technically) | Yes (changes only) | Yes |
| DELETE | Remove | Yes | No | 204 |

- Never use GET to mutate state. Never use POST for safe reads.
- POST that performs an action (`/claims/{id}/submit`) is acceptable — name the action explicitly.

## HTTP status codes

| Code | When |
|---|---|
| 200 OK | Successful GET, PUT, PATCH, action POST |
| 201 Created | Successful resource-creating POST. Include `Location` header. |
| 202 Accepted | Async work queued — include status URL |
| 204 No Content | Successful DELETE or PUT with no response body |
| 400 Bad Request | Validation failure, malformed body |
| 401 Unauthorized | Missing/invalid credentials |
| 403 Forbidden | Authenticated but not permitted |
| 404 Not Found | Resource doesn't exist OR caller can't see it (avoid 403→404 distinction for sensitive resources) |
| 409 Conflict | Optimistic lock failure, duplicate |
| 422 Unprocessable Entity | Body parses but violates business rule |
| 429 Too Many Requests | Rate limit — include `Retry-After` |
| 500 Internal Server Error | Unexpected — generic message |
| 503 Service Unavailable | Downstream down — include `Retry-After` |

- Never return `200` with a failure body. The `status` in the envelope must match the HTTP status.
- Never return 500 for validation errors.

## Response shape

- Success and error responses use the standard `Response<T>` envelope — see `error-handling-java.instructions.md`.
- Do not return a different envelope per endpoint.

## Request and response bodies

- **Content type:** `application/json`.
- **Field naming:** camelCase. No snake_case, no PascalCase.
- **Dates:** ISO-8601 strings (`2026-05-24T09:15:00Z` for instants, `2026-05-24` for dates). Never epoch ints.
- **Money:** decimal as string (`"123.45"`) with explicit `currency` field. Never float, never cents-as-int without documenting it.
- **IDs:** strings, even when numeric — protects against JS precision loss.
- **Booleans:** named `is*` / `has*` / `can*`. `enabled`, `active`, `paid` also fine. Never `disabled` (avoid negative names).
- **Nulls:** omit absent fields rather than emitting `"field": null` (configure Jackson). Exception: PATCH semantics where `null` means "set to null".

## Pagination

Cursor-based for large/active collections; offset-based acceptable for small static lists.

**Cursor (preferred):**
```
GET /v1/claims?pageSize=20&cursor=eyJpZCI6Ii4uIn0
→ {
  "items": [...],
  "nextCursor": "eyJpZCI6Ii4uIn0",
  "hasMore": true
}
```

**Offset (only when total count matters and dataset is small):**
```
GET /v1/claims?page=0&pageSize=20
→ {
  "items": [...],
  "page": 0,
  "pageSize": 20,
  "totalElements": 142,
  "totalPages": 8
}
```

The page object above is the `payLoad` of the `Response<T>` envelope.

- Default `pageSize`: 20. Max: 100. Reject larger.
- Never return unpaginated collections.

## Filtering, sorting, searching

- **Filter:** plain query params — `?status=ACTIVE&memberId=abc-123`.
- **Sort:** `?sortBy=createdDate&sortOrder=desc`. Allow multiple via repeat: `?sortBy=status&sortBy=createdDate`.
- **Search:** `?q=...` for full-text. Document searchable fields.
- Document the allowed filter/sort fields per endpoint — do not silently accept arbitrary fields (SQL injection adjacent + leaks schema).

## Versioning

- URL-path versioning: `/v1/...`, `/v2/...`. No header versioning, no query versioning.
- Only the major version is in the URL. Minor/patch are backward-compatible additions.
- A breaking change requires `v(N+1)`. Run both versions in parallel for ≥1 deprecation cycle.
- Deprecation: response includes `Deprecation: true` and `Sunset: <date>` headers.

## Required headers

- `messageId` — the upstream trace id; propagate it, never generate a new one downstream. See `logging-java.instructions.md`.
- `Content-Type` — set explicitly on requests with bodies
- `Accept` — clients should set; default to `application/json`
- `Authorization` — Bearer token (never API key in URL)

## Authentication and authorization

- Bearer tokens only (Okta, via `mem-security-lib-starter`). Never API key in query string or path.
- Every endpoint is protected with `@RequestReadScoped` (read) or `@RequestWriteScoped` (write) — see `config-security-libs-v2.instructions.md` and `owasp-java.instructions.md` A01.
- Endpoints that touch PHI must verify the caller is permitted for that specific record, not just the resource type.

## Idempotency

- Mutating POST endpoints that may be retried (payments, submissions) accept an `Idempotency-Key` header. Server stores key+result for ≥24h.
- PUT and DELETE are naturally idempotent.

## Async / long-running

- Operations >2s should return 202 with a status URL: `Location: /v1/jobs/{jobId}`.
- The status endpoint returns `{status, progress, resultUrl}`.

## What never to do

- ❌ Tunnel methods via POST + `_method` param
- ❌ Return HTML from a JSON API
- ❌ Use HTTP for anything other than what the method semantically means
- ❌ Put secrets in query strings (logged everywhere)
- ❌ Return different shapes for the same endpoint based on a flag — version it instead

## Refusal pattern

> ⚠️ API design guardrail: <one-line rule>. Suggested alternative: <code>.
