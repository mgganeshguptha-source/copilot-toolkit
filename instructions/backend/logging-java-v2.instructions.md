---
applyTo: "src/main/java/**/*.java,**/log4j2*.xml,**/logback*.xml"
---

# Logging — Java / Spring Boot

Stack-specific logging rules for Spring Boot. Follows the shared contract in `logging-standards.instructions.md`. Compounds with `hipaa-guardrails.instructions.md` (no PHI in logs, ever).

## Level semantics

- DEBUG: diagnose program state. INFO: normal operation events. ERROR: unexpected operation errors. WARN: expected business exceptions (validation, not-found) and fail-open fallbacks.
- Set levels per application/package in config properties.
- Be descriptive; include context (trace id in MDC). Never log passwords or sensitive data.

## Logger acquisition

- Use the SLF4J facade via Lombok `@Slf4j`. If Lombok is not on the classpath: `private static final Logger log = LoggerFactory.getLogger(MyClass.class);`
- Never use `System.out`, `System.err`, `printStackTrace()`, `java.util.logging`, or the Log4j API directly (`@Log4j2`, `LogManager.getLogger`). Log4j may be the backend behind SLF4J (MPA); code still logs through SLF4J.
- Do not log request payloads that may contain PHI/PII (HIPAA + secure-coding checklist "no PII in logs").

## Parameterised logging

- Always use `{}` placeholders, never string concatenation:
  - ✅ `log.info("order processed orderId={}", orderId);`
  - ❌ `log.info("order processed orderId=" + orderId);` — builds the string even when level is disabled
- For expensive payloads guarded by level: `if (log.isDebugEnabled()) { log.debug(...); }`
- Pass the exception as the **last** argument to capture the stack trace:
  - ✅ `log.error("payment failed orderId={}", orderId, ex);`
  - ❌ `log.error("payment failed orderId=" + orderId + " " + ex.getMessage());` — loses the trace

## Structured logging

- Add fields via MDC or structured arguments (`StructuredArguments.kv("key", value)` where the project uses `logstash-logback-encoder`) — never by interpolating into the message.
- Follow the logging config the project already has (`logback-spring.xml` or log4j2 XML); do not introduce a second logging backend.

## Trace identifiers

- Microservices: the request-trace `HandlerInterceptor` from the common library puts the trace id into MDC. Do not write another interceptor or filter for this.
- The standard header is the upstream `messageId`; propagate it and do not generate a new one downstream.
- Log both `messageId` and `correlationId` when they differ.
- Outbound: every `WebClient` / `RestTemplate` call carries the inbound `messageId`. Never let a request reach a downstream service without it.
- **Always clear MDC at the end of the request** if your code sets it — leaked MDC pollutes the next request on a thread pool.
- For async code (`@Async`, `CompletableFuture`, reactive), propagate MDC/trace context explicitly. Plain async drops it.

## What to log at request boundaries

- **Inbound, request entry:** INFO with `httpMethod`, `requestPath`, `messageId`, `appName`. Never the request body.
- **Inbound, request exit:** INFO with `httpStatus`, `durationMs`. Do not log the full response body.
- **Outbound call:** DEBUG with method + URL + status + duration. Never the full body for PHI-handling targets.

## MPA (monolith)

- Same rules plus: log4j XML config in `mpa-common`; asynchronous logging; separate logs for SOAP (CXF appended) and REST APIs; separate audit and application logs; dynamic levels via `LevelFilter`.

## CloudWatch (microservices / cloud monolith)

- Log group per application name; log streams per internal/external service call.
- Stream prefixes: internal calls `MBP_`; external/common services `CXF_`.
- A scheduler keeps streams active for one day (rotation).
- CloudWatch logic lives in the common library — do not re-implement per service.

## What never to log (Java-specific reminders)

- DTO `.toString()` for any DTO that may contain PHI — DTOs often auto-generate `toString()` via Lombok `@Data`. Use `@ToString.Exclude` on PHI fields or explicit field selection.
- JPA entities — their `toString()` may trigger lazy-load and dump the entire object graph.
- `Authentication` / `Principal` objects — they may contain credentials.
- `HttpServletRequest` / `HttpServletResponse` / `ServerWebExchange` directly — they may contain headers like `Authorization`.

## Exception logging

- Log the exception once at the boundary where it's handled. Do not log-and-rethrow at every layer — produces duplicate stack traces.
- The central mapper (`SharedControllerAdvice` / the MPA `@ControllerAdvice`) is the typical single point. Inner layers re-throw without logging.
- For expected business exceptions (validation, not-found), log at WARN. For unexpected, ERROR.

## Refusal pattern

> ⚠️ Java logging guardrail: <one-line rule>. Suggested alternative: <code>.
