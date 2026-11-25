---
applyTo: "src/main/java/**/*.java"
---

# Java Source Conventions (Spring Boot)

## Naming

- Classes `PascalCase` nouns (`OrderService`). Methods `camelCase` verbs (`createOrder`, `findOrdersByCustomerId`). Variables `camelCase`.
- Interfaces: **no `I` prefix**; descriptive (`OrderRepository`, `OrderDao`).
- Constants `UPPER_SNAKE_CASE` in `public static final`. Boolean variables read as `is*`/`has*`.
- Suffixes: `Controller`, `Service`, `Repository` / `Dao` by layer. DTO suffix `Request`/`Response` (`CreateOrderRequest`) — not `Dto`.
- Packages (lowercase): Spring Boot microservices `com.bcbsm.{mbp|mex}.{service-short-name}`; MPA monolith `com.bcbsm.mpa.<module>.<package>`. Layer packages: see `layering-architecture-v2.instructions.md`.

## Code style

- Use constructor injection (Lombok `@RequiredArgsConstructor`). Do not use field injection (`@Autowired` on fields).
- Mark injected dependencies `private final`.
- Apply `@Transactional` on service-layer methods where a transactional boundary is required, not on controllers.
- Lombok: selective annotations; `@Data` only when genuinely required, never on JPA entities — see `lombok-mapstruct-v2.instructions.md`.
- Log through SLF4J (`@Slf4j`) — see `logging-java.instructions.md`. Never log secrets, tokens, or PII.
- Keep controllers thin — no business logic. Business logic belongs in services.
- Never build SQL or JPQL via string concatenation of user input.

## Language rules

- Prefer explicit types in public APIs; `var` only for private/local scope.
- Favour immutability: records, Lombok `@Value`, `final` fields, `List.of()`. Use separate mapped domain objects when a framework needs setters.
- Prefer `Optional<T>` return types in non-persistence layers; avoid returning `null`.
- No public mutable static fields. Use try-with-resources for closeables. Program to interfaces.
- Prefer returning a value over mutating arguments.

## Methods

- One responsibility per method; small; avoid deep nesting; avoid many parameters; no duplicated logic (extract it).
- Add a Javadoc/comment for every method: functionality, **rationale** for non-obvious choices, bug-fix note when fixing a defect, JIRA/Confluence link where relevant.
- Do not comment out code — delete it.
- When adding parameters to a method that already has callers, **overload** instead of changing the signature.
- Do not suppress errors unless required.
- New-method checklist (all must be "yes"): single responsibility; readable in 30 seconds; clear errors with context; side effects isolated; right layer; scales to 10x data; testable in isolation.

## Reuse before you add

- Before writing a new method, helper, client call or utility, search the codebase (same class, its collaborators, shared/common modules and the common libraries) for one that already does the job. Reuse it, even if the call needs a small adaptation at the call site.
- Do not add methods to shared or common modules (common request/client classes, base classes, shared utils, common libraries) unless the plan names the addition and states why no existing method fits.
- When a class already has a pattern for a concern — how it calls downstream services, wraps or unwraps responses, null-checks results, maps errors — follow that pattern in new code. Do not introduce a second way of doing the same thing in the same class.
- If no existing method fits, say which ones were considered and why they do not fit (in the plan or the phase summary).

## Existing contracts and types

- Do not change the signature of an existing public method or interface method (return type, parameter types, method name) unless the plan names that change. Fit the new code to the existing contract, not the contract to the new code.
- Never widen a type to get past a compile error: no `Object` parameters, raw types, or wildcards (`Mono<?>`, `List<?>`, `Map<String, ?>`) in place of the concrete type the code actually handles. A type mismatch means the wrong class was picked — fix the choice, not the signature.
- When two classes share a simple name (e.g. a generated client model and a server model, both `VisitSummaryResponse`), use the one the plan names, with an explicit import or its fully qualified name. Never accept both.
- Do not add `instanceof` checks or type switches on a value whose type the signature already guarantees. A branch that can never run is dead code — do not write it.
- Map each field in one place. When a mapper/translator (e.g. MapStruct) already sets a field, do not set it again after the mapping call.

## Enums

- Use enums for any field with a fixed set of values; constants `UPPER_SNAKE_CASE` (`STYLE_BOLD`).

## Deprecation

- Avoid deprecated APIs; read their Javadoc for the replacement.
- When deprecating: `@Deprecated` + Javadoc `@deprecated` with reason and alternative. Review and remove deprecated methods periodically.

## Utility classes

- `final`, private constructor, all members `static`, no state, no injected dependencies, no domain business logic. Do not dump unrelated helpers in `utils`.

## Functional style

- Prefer streams/lambdas/`Optional` over imperative loops; use `Predicate`, `Function`, `Consumer`, records/pairs for temporary structures.
- Method references over lambdas (`Integer::sum`). `filter()` before `map()`. Use `sorted()` sparingly. No nested streams (use local variables). No parallel streams unless justified. Reactor rules: `reactive-conventions-v2.instructions.md`.

## Dependencies and versions

- Maven (microservices) / Gradle (MPA). Semantic versioning `major.minor.patch`. Versions in properties for shared dependencies.
- Upgrade libraries to latest **stable, vulnerability-free** versions; check CVEs when upgrading.

## Static analysis

- SonarLint aligned with project rules; fix reported issues; see `engineering-process-v2.instructions.md`.
