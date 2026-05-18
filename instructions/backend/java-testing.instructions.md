---
applyTo: "src/test/java/**/*.java"
---

# Java Testing Conventions (Spring Boot)

## Framework and Tooling

- JUnit 5 only. Do not use JUnit 4.
- Mockito for mocking: `@Mock`, `@InjectMocks`,
  `@ExtendWith(MockitoExtension.class)` for pure unit tests.
- Verify mock interactions only when the interaction is the thing under
  test — do not verify every call.

## Test Types

- Pure unit tests: no Spring context, no real database. Deterministic
  and fast.
- Repository tests: `@DataJpaTest`.
- Controller tests: `@WebMvcTest` with `MockMvc` — assert status codes,
  error response shape, and controller-to-service interactions.
- Full integration tests: `@SpringBootTest`. Mock external dependencies
  with `@MockBean`.
- Do not use a real database connection in unit tests.

## Test Structure and Placement

- Test class naming: `<ClassName>Test`, file suffix `*Test.java`.
- Place tests under `src/test/java` mirroring the production package
  structure exactly. Do not create tests outside this folder.
- Cover, at minimum: happy path, negative cases, edge cases, and
  null/empty inputs.
- Each test asserts one behaviour. Prefer several focused tests over one
  test with many assertions.
