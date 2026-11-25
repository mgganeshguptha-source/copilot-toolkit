---
applyTo: "src/main/java/**/*.java"
---

# Reactive / Functional Conventions — v2

## Reactor rules
- Use `doOnNext`, `doOnError`, `doFinally` only for logging/metrics — never for business logic (setting fields, initialising variables).
- Blocking calls: `Mono.fromCallable(...).subscribeOn(Schedulers.boundedElastic())`, not `Mono.just(blockingCall())`.
- Use `flatMap` (not `map`) when the mapper returns a `Mono`/`Flux`.
- Do not call `subscribe()` unless you intend to own/dispose the stream.
- Compose independent calls in parallel with `Mono.zip(a(), b(), c()).map(t -> combine(t.getT1(), t.getT2(), t.getT3()))` instead of nested `flatMap` chains.
- Do not sprinkle `publishOn/subscribeOn`; know which thread you land on.
- Use `switchIfEmpty` for fallbacks instead of null checks.
- Avoid `try/catch` in functional code (`ThrowingFunction`); logging inside pipelines via `OptionalUtils.peek`.

## Resilience, validation and logging
- Timeouts and resilience are configured centrally (`client.mappings.<service>.timeout`, infrastructure), not hand-written per call.
- Validate with `@Valid` + Bean Validation on generated models; use a filter returning an empty `Mono` only for rules the contract cannot express.
- Logging uses `@Slf4j` (see `logging-java.instructions.md`).
