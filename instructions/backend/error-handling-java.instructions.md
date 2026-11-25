---
applyTo: "src/main/java/**/*.java"
---

# Error Handling — Java / Spring Boot

Stack-specific error-handling rules. Compounds with `owasp-java.instructions.md` (A05, A09) and `hipaa-guardrails.instructions.md` (no PHI in error responses).

## Exceptions

- Use domain-specific exceptions. Checked exceptions only where the caller must react; otherwise runtime.
- Microservices throw `GlobalException` or a subclass of it (from `mem-common-utils-lib`). Do not throw raw `RuntimeException` / `Exception`.
- Exception class names are PascalCase ending in `Exception` (`OrderNotFoundException`).
- Never swallow exceptions. Wrap lower-level exceptions in a meaningful service-level exception and keep the cause: `throw new MyException("...", e);`.
- DAO layer handles no-data / persistence exceptions and translates them; Service layer handles every route of logic and its exceptions.
- Never set the HTTP status manually in business code — the central mapper owns it. Pick (or create) the exception type that represents the failure.

## What never to do

- ❌ Catch `Exception` or `Throwable` broadly. Catch the narrowest type that makes sense.
- ❌ Catch-and-ignore: `catch (Exception e) { /* silent */ }` — always log or rethrow.
- ❌ Catch-and-rethrow as `RuntimeException(e.getMessage())` — loses the cause.
- ❌ Log-and-rethrow at every layer — log once, at the central mapper.
- ❌ Return `null` to signal failure when an exception is the right tool.
- ❌ Use exceptions for control flow (e.g., `try { parseInt } catch` in a hot loop) — performance and readability cost.

## Central mapping

- One central mapper only. Microservices use the shared `SharedControllerAdvice` from the common library (component-scanned via `com.bcbsm.*`). Do not add a local `@ControllerAdvice` unless handling a service-specific scenario, and then only for that scenario.
- MPA (Spring MVC monolith): a single `@ControllerAdvice` or `SimpleMappingExceptionResolver`.
- New controllers do not implement their own try/catch for cross-cutting cases.

The central mapper:
1. Maps exception type → HTTP status
2. Builds the `Response<T>` envelope (below)
3. Logs the exception (once)
4. Sanitises — never leaks stack trace, internal class names, SQL, or PHI to the client

## Functional code

- Avoid `try/catch` inside lambdas/streams. Use the `ThrowingFunction` helpers from the common library.
- For logging inside functional pipelines use `OptionalUtils.peek` (common lib); in Reactor use `doOnNext/doOnError` for logging/metrics only.

## Client-facing responses

- Never expose stack traces, internal messages, class names, SQL. Return sanitised messages with a **unique error code**.
- Log the full stack trace server-side with the trace/correlation id; return a readable message to the client.
- Standard envelope (`com.bcbsm.mem.common.utils.lib.model.response.impl.Response<T extends IResponse>`):

```json
{
  "messageId": "String",
  "status": "String",
  "timestamp": "String",
  "systemMessages": [ { "code": "String", "description": "String", "additionalInfo": "String" } ],
  "payLoad": <Response Payload>
}
```

- `SystemMessage` fields: `additionalInfo, description, code, errorCode, errorMessage, reason, source`.
- `stackTrace` in the `Response` class must be empty/null in every non-local environment.
- `messageId` is mandatory on every error response.
- `status` in the envelope must match the HTTP status. Never return `200` with a failure body.
- Never put PHI, stack traces, SQL, file paths, or internal class names in `description` or `additionalInfo`.
- Field-level validation results go in `systemMessages[]`, one entry per field.

## Status by failure kind

| Failure | HTTP | Notes |
|---|---|---|
| Validation (`@Valid`, Spring `MethodArgumentNotValidException`, `WebExchangeBindException`) | 400 | One `systemMessages[]` entry per field |
| Not authenticated | 401 | Never reveal whether user exists |
| Not permitted (scope check) | 403 |  |
| Not found | 404 |  |
| HTTP method mismatch | 405 |  |
| Conflict (optimistic lock, duplicate) | 409 |  |
| Unprocessable business rule | 422 |  |
| Rate limit | 429 | Include `Retry-After` |
| Downstream unavailable | 503 | Include `Retry-After` if known |
| Any uncaught exception | 500 | Generic message; log full trace server-side |

## Validation

- Use Bean Validation (`@Valid`, `@NotBlank`, `@Size`, etc.) on controller DTOs; sanitise strings.
- Custom validators extend `ConstraintValidator`. Never validate by writing imperative checks in the controller method — moves error handling out of the central mapper.
- Validation errors are 400 with `systemMessages[]` populated.

## Retry and circuit-breaker

- Use Spring Retry or Resilience4j; never write raw `for (i=0; i<3; i++) try {...}` loops.
- Retry only on technical/downstream failures, never on business errors — business errors don't get better by retrying.
- Circuit breaker/fallback for external SOAP calls (see `engineering-process-v2.instructions.md`). Do not let a failing downstream cascade into thread pool exhaustion.

## Async / reactive

- For `@Async` methods returning `CompletableFuture`, exceptions must be handled in `.exceptionally()` or `.handle()`. Never let exceptions escape an async boundary unhandled — they get swallowed.
- For Reactor (`Mono`/`Flux`), use `.onErrorMap()` to convert technical exceptions to `GlobalException` subclasses before they reach the controller.

## Refusal pattern

> ⚠️ Error-handling guardrail: <one-line rule>. Suggested alternative: <code>.
