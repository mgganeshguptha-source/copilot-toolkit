---
applyTo: "src/main/java/**/*.java"
---

# Layering, Spring DI and Persistence — v2

## Layers

### Controller
- Handle only incoming HTTP/websocket requests. Map request-derived state (tokens, `HttpServletRequest`/`ServerWebExchange` attributes, logged-in user) to POJOs before calling the service.
- Thin: parse → validate (`@Valid`, sanitise strings) → call service → return DTO. No business logic, no data access.
- Implement the OpenAPI-generated interface; group methods by domain object.

### Service
- Business logic and orchestration; stateless; exposes an interface with only required methods.
- Calls DAO/Repository or Integration (API) classes. **Do not call cross-domain DAO/API directly** — `ClaimsService` needs user data → call `UserService`, not `userApi`.
- Do not leak entity models or API/vendor models into this layer. SOAP-generated stubs must never be used here.
- Owns the full flow (e.g. DB call first, then downstream API) and exception handling. `@Transactional` boundaries live here: minimal, explicit, `readOnly = true` for reads.

### DAO / Data access
- Encapsulate all persistence and query logic; prefer repositories over named queries.
- Return internal POJOs/records — never `Document`, `DBObject`, `Cursor` or entities.
- Separate packages for reactive and non-reactive DAOs; reactive DAOs return `Mono`/`Flux`.
- Handle persistence exceptions and no-data cases here.

### Integration / API
- One API class per external client (or group). Translate vendor/SOAP models immediately via dedicated translators/mappers; service never knows if it is SOAP, REST or file.
- Wrap blocking calls (SOAP) with `Mono.fromCallable(...).subscribeOn(Schedulers.boundedElastic())`.
- Cache responses where needed; never call APIs inside loops or parallel streams.

### Utils / mappers
- Utils: stateless, final, no dependencies, no domain logic. Mappers in `mapper` package, one responsibility each (see `lombok-mapstruct-v2`).

## Package structure

Microservice `com.bcbsm.{mbp|mex}.{short}`: `config`, `controller`, `service`, `dao`, `repository` (Spring Data), `utils`, `mappers`, `models` (POJOs), `entity` (DB objects), `translator` (vendor → app POJO), `api`/`webservice` (external calls).
MPA monolith `com.bcbsm.mpa.<module>.<package>` (e.g. `com.bcbsm.mpa.web.config`); modules: `mpa-app` (root), `mpa-api`, `mpa-nba`, `mpa-common`, `mpa-config`, `mpa-data`, `mpa-service`, `mpa-web`; Gradle with one `build.gradle` per module plus a root build.
Every module has `src/main/java` and `src/test`, even if empty. Logical layers: web, api (DTOs), service, client, persistence, cache, common/util.

## Spring DI

- Consistent Java-based config (`@Configuration`). Constructor injection for mandatory dependencies; field/setter injection only for optional ones.
- Stereotypes: `@Controller`/`@RestController`, `@Service`, `@Repository`, `@Component`. `@Repository` on data-access beans for exception translation.
- Singleton scope unless a documented reason exists.

## Persistence

### Oracle (mostly read)
- Connection pool: HikariCP (microservices), JNDI (MPA monolith). Java 17 compatible.
- `JdbcTemplate` (MVC) and/or JPA/Hibernate for read queries. Always parameterised/prepared statements. No `SELECT *`; select needed columns; proper indexes. Optimistic locking/snapshot where concurrency matters.
- If a service has no local DB, do not add one.

### MongoDB (application DB / L2 cache / documents)
- `MongoTemplate` or supported driver wrapper; clean repositories. No wide, unbounded documents; keep schema-version metadata. TTL where applicable and `updatedAt` for L2 invalidation. No sensitive data in plain text.

## Security basics (config & code)
- Least-privilege DB users, network segmentation, TLS with certificate validation for external connections, validate/sanitise all input before queries or CXF payloads.
