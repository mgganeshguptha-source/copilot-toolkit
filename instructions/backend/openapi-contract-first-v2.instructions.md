---
applyTo: "**/*.yml,**/openapi/**,**/pom.xml,src/main/java/**/controller/**/*.java"
---

# OpenAPI Contract-First — v2

## Approach
- Contract first: define the OpenAPI (3.0.x) yml before development. It produces Swagger docs, server controller interfaces, and client models consumed by other services.

## Maven module layout (microservices)
- Root `pom.xml` has two modules: `{mbp|mex}-{short-name}-service-application` and `{mbp|mex}-{short-name}-service-openapi-code`.
- `-service-openapi-code` holds the yml under `src/main/resources` and two submodules: `-service-openapi-client-code` and `-service-openapi-server-code`.
- Application module package: `com.bcbsm.{mbp|mex}.{short-name}`.

## yml content rules
- Start with service and functionality description.
- Each path defines: URL, HTTP method, `summary`, unique `operationId` (becomes the controller interface method), `tags` (group APIs into one controller), `parameters` (header/path), `requestBody`, `responses` per status code.
- Models in `components/schemas`; mark mandatory fields with `required` so the validator rejects incomplete payloads.
- Path naming follows `api-design.instructions.md`.

## Server code generation
- openapi-generator Maven plugin: `executions/execution/configuration/inputSpec` points to the yml; output goes to the server-code module.
- Add the server-code jar as a dependency of the application pom; the controller implements the generated interface (no `@RequestMapping` on the class).

## Client code generation
- Client-code module is published and added as a dependency by consuming services (e.g. `mbp-claims-service` client in `mex-claims-service`).
- Consumer injects the OpenAPI WebClient with the base path in a config bean (`AbstractClientConfig.openAPIWebClient`), obtains the Okta token, then calls the service. Two classes with the same simple name (client vs server model) must be imported explicitly.
