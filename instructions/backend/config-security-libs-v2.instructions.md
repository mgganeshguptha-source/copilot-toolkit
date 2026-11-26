---
applyTo: "src/main/java/**/*.java,**/application*.yml,**/application*.properties"
---

# Configuration, Secrets and Shared Libraries — v2

## Externalising properties
- No hard-coding of values that change per environment or frequently. Common values go in the primary file (`application.yml` microservices / `application.properties` MPA); environment-specific values in profiles.
- **Secrets are encrypted** with an encryption library; never plain text in config files, and never copied into instructions, samples or code. Use a secrets manager (Vault / AWS Secrets Manager) in production.
- Property names in camelCase.
- Bind with `@ConfigurationProperties` POJOs; avoid `@Value` (if used, provide a default where applicable).

## Config locations
- Microservices: config repos served by config servers — MBP: `dx_mbp-config-repo` / `dx_mbp-config-server`; MEX: `dx_mex-config-repo` / `dx_mex-config-server`. Cross-service config in `application.yml`; service-specific config segregated by service-name package.
- MPA monolith: custom property files per environment and module under `mpa-config` (e.g. `bcbsm_dev1`), deployed externally, **not in the build**. A property change must be applied to every environment file.

## Common utils library (`dx_mem-common-utils-lib`)
- Transitive dependency of `mem-security-lib-starter`. Main class adds `@ComponentScan(basePackages = {"com.bcbsm.*"})`.
- WebClients: external via `serviceWebClient`, internal via `openAPIWebClient` in `AbstractClientConfig`.
- Shared `ObjectMapper` bean — do not define another. Client properties under `client.mappings.<service-name>` (`basePath`, `maxInMemorySize`, `timeout`).
- Exceptions via `GlobalException` subclasses; `ThrowingFunction`; `OptionalUtils.peek`; request-trace `HandlerInterceptor` puts the trace id into MDC.

## Security library (`mem-security-lib-starter`)
- Provides the Okta WebClient bean, security filter chain, whitelisted APIs, scope-based permission evaluator and JTI validation. Do not re-implement them in services.
- Token config stays in the service, following the config-repo pattern: `services.read.scopes`, `services.write.scopes`, `services.okta.<name>.{host,path,clientId,clientSecret,clientName,grantType,timeout,tokenName,cron,scope}`.
- `clientSecret` must be encrypted/injected — **the source document shows a plaintext value; never reproduce it**.
- Scope pattern: `urn:<service-name>-v1.0.0:<http_method>:any` (e.g. `urn:mbp-membership-service-v1.0.0:get:any`); in `scopes` properties prefixed `SCOPE_`.
- Endpoint protection uses `@RequestReadScoped` / `@RequestWriteScoped` (scope-based).
