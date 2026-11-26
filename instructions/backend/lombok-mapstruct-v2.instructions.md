---
applyTo: "src/main/java/**/*.java"
---

# Lombok and MapStruct — v2

## Lombok

- Use selective annotations (`@Getter`, `@Setter`, `@Builder`, `@ToString`) as needed. `@Data` only when genuinely required; never on JPA entities.
- Immutable classes: `@Value` or a Java `record`.
- Prefer `@Builder` over setters, especially in reactive pipelines.
- No `@Autowired` field injection — use `@RequiredArgsConstructor`.
- Avoid `@AllArgsConstructor` in services.
- Logging: `@Slf4j` rather than hand-written loggers.
- Exclude PHI/sensitive fields with `@ToString.Exclude`.

## MapStruct (microservices)

- Use MapStruct for entity ↔ DTO/POJO mapping; mappers live in the `mappers` package, one responsibility per mapper.
- Dependency `org.mapstruct:mapstruct:1.5.5.Final` (property-managed version).
- `maven-compiler-plugin` (source/target 17) `annotationProcessorPaths` in this order-compatible set: `lombok` (`${lombok.version}`), `mapstruct-processor` 1.5.5.Final, `lombok-mapstruct-binding` 0.2.0.
- Compiler arg `-Amapstruct.defaultComponentModel=spring` so mappers are Spring beans.
- Run `mvn clean install` to generate implementations. Never hand-edit generated mapper classes.
- Map each field in one place.
