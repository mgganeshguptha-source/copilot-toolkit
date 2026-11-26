---
applyTo: "src/main/java/**/*.java"
---

# Caching — v2

## Migrating from the MPA monolith to a microservice

- The monolith caches with a **Caffeine L1 + Redis L2** stack (`CacheProxyManager` / `CacheObject`, `PerCacheTtlCaffeineCacheManager`, Redis pub/sub invalidation). Do not carry any of this into a microservice: no Caffeine, no Ehcache, no `CacheProxyManager`/`CacheObject`, no Mongo-backed cache, no pub/sub invalidation subscriber, no blocking `RedisTemplate`.
- Rebuild each monolith cache using the microservice pattern below (Redis only, reactive). Follow the reference implementation `dx_mbp-cms-service` (`ReactiveCmsCacheServiceImpl`, `CachingConstants`, `RedisCacheTtlConfig`, `CmsKeyGenerator`).
- Map each monolith cache one-to-one:
  - monolith `CacheNames` constant → microservice cache-name constant + `CacheProfile` entry;
  - monolith key parts (`EhcacheKeyGenUtil.buildCacheKey(...)` fields such as personId, methodName, contract/group number, dates) → the composite key fields of the key generator;
  - monolith per-cache TTL (`cache.properties`, `cacheTtlMap`) → the `CacheProfile` TTL;
  - monolith `services.cache.disabledCaches` switches → config-driven enable/disable, not code changes.
- Caches the monolith populated from Mongo (e.g. the membership cache stored on `ActiveUser`) are not ported as caches; the microservice caches the downstream response in Redis instead.
- Keep the Redis key layout identical to the monolith/other services so cross-service eviction keeps working: `<keyPrefix><cacheName>::<generatedKey>` (the monolith evicts microservice entries by `cacheName` + key through `evictMicroServicesRedisCacheByKeys`). Do not rename an existing cache name or change its key composition.
- If a monolith cache is session-scoped, UI-only, or has no equivalent downstream call, say so and do not cache by default.

## Reactive Redis (mandatory in microservices)

- Microservices are WebFlux/Reactor. All cache reads and writes use `ReactiveRedisTemplate` (directly in a cache service, or via `@ReactiveCacheable` on WebClient-impl methods where the common library provides it). No blocking `RedisTemplate`, no `.block()`, no `@Cacheable` on `Mono`/`Flux` methods.
- Use a small cache service with a **get-or-load** contract: look up the key; on miss run the loader `Supplier<Mono<T>>`, write the result, return it.
  ```java
  <T> Mono<T> getOrLoadValue(String cacheName, String key, Class<T> type, Supplier<Mono<T>> loader);
  ```
- Fail open: cache read/write errors are logged at WARN (`onErrorResume`) and treated as a miss; a Redis problem must never fail the request. A write failure still returns the loaded value.
- Wrap miss handling in `Mono.defer(...)` with `switchIfEmpty`, so the loader runs only on a miss.
- Coalesce concurrent misses for the same key (in-flight `Mono` map with `.cache()` and `doFinally` removal) so a cold or expired key triggers one downstream load, not a thundering herd.
- Never cache `Mono.empty()` or error signals. Do not write a value that cannot round-trip safely (for example payloads holding transient streams); skip the write and return the live response.
- Treat an undeserializable or structurally unusable cached entry as a miss and overwrite it.
- Build templates with an explicit `RedisSerializationContext`: `StringRedisSerializer` for keys. For values either (a) `SafeJdkSerializationRedisSerializer` restricted to trusted packages (`"com.bcbsm."`, `"java."`) for polymorphic `Response<IResponse>` payloads, or (b) raw `byte[]` plus Jackson with an explicit target type for typed values. Never use an unrestricted JDK deserializer or Jackson default typing.
- Copy the shared `ObjectMapper` (`objectMapper.copy()`) before changing cache-specific settings; never mutate the application-wide mapper.
- Log cache HIT / MISS / PUT at INFO or DEBUG with cache name and key (never the value, which may be PHI).

## Cache plan (required in the design/plan before any caching code)

- Caching is decided in the design or plan, never while coding. Any design or plan that adds, changes or removes a cache must contain a short cache plan, so the person who reviews the plan approves the keys before code exists.
- The cache plan contains, for each cache:

  | Cache name | Data cached (response/model, source call) | Key fields | Example generated key | TTL | Per-user data? | Eviction/reload trigger |
  |---|---|---|---|---|---|---|

  - Example key must show the full Redis form: `<keyPrefix><cacheName>::<generatedKey>`.
  - State what is deliberately **not** cached and why (empty/error results, PHI-heavy or user-specific data, session-scoped data).
  - For a migration, add a column mapping each monolith cache (`CacheNames` constant, `EhcacheKeyGenUtil` key parts, TTL) to its microservice equivalent, and flag any key or name that would change.
- When coding, implement only the caches, names, keys and TTLs the plan lists, exactly as listed. Do not add, rename or re-key a cache that is not in the plan.
- If caching looks useful but the plan has no cache plan for it, do not add it — note it in the summary as a suggestion for the plan.
- If the story or plan says not to cache a call, implement without caching.

## Cache names, TTLs and keys

- Define every cache in one place: a `CachingConstants` interface with name constants and a `CacheProfile` enum (`cacheName`, `Duration ttl`, `description`). Add or change a cache there only.
- Set an explicit TTL per cache; unspecified caches fall back to the global default from the `redis_cache` profile (`spring.cache.redis.*` in config). Pick TTLs by volatility: static/segment-derived content long (hours), person/membership data short (minutes).
- Apply TTLs through the profile enum, via `RedisCacheManagerBuilderCustomizer` and the reactive service's `resolveTtl(cacheName)`; do not hard-code durations in service code.
- Redis host/port, `spring.cache.type=redis` and `spring.cache.redis.key-prefix` come from the common config; read the prefix with `@Value("${spring.cache.redis.key-prefix:}")` and do not redefine connection settings in the service.
- Keys are **composite** and built by a key generator extending the common `CacheKeyGenerator` (`com.bcbsm.mem.common.utils.lib.redis.cachekey`), taking `ObjectMapper` and application name. Use the identifying request fields (e.g. operation + functionalArea + resolved personId; personId + memberId; contractNumber + startDate + endDate). For long request objects, generate the key from the request object rather than hand-built concatenation.
- Include the person/user identifier in the key for any user-dependent data. Never cache PHI-bearing responses under a key shared across users.

## Eviction

- Manual eviction goes through `RedisCacheService` (reactive), not inline template calls scattered in business code.
- When the request carries a reload/refresh flag, evict (or bypass) before the call so the fresh value repopulates the cache.
- Clear by cache name with the `<keyPrefix><cacheName>::*` pattern using SCAN, never `KEYS`; delete keys individually (Redis Cluster rejects multi-key `DEL` across slots).

## Non-reactive services only

- If a service is genuinely non-reactive, use Spring `@Cacheable` / `@CacheEvict` / `@Caching` against Redis with `unless` to skip empty/error results, `keyGenerator = "customCacheKeyGenerator"` for long request objects, and the same cache names/TTL profile.

## Code that stays in the monolith (MPA)

- Use `CacheProxyManager.getCacheObject(CacheNames.X)` with a `CacheNames` constant; do not create ad-hoc caches.
- L1 is a single Caffeine instance (bounded `maximumSize`, per-entry TTL from `caffeineL1TtlMap`); L2 is Redis (Lettuce, cluster-aware), both configured in `MpaCacheConfiguration` from properties/environment (`CAFFEINE_CACHE_MAX_SIZE`, `CAFFEINE_CACHE_TTL_SEC`, `REDIS_*`, `cache.properties`). Do not hard-code sizes or TTLs.
- Build keys with `EhcacheKeyGenUtil.buildCacheKey(...)` using stable, deterministic parts; keep `cacheName::key` layout.
- Invalidate through `CacheProxyManager` so L1 is cleared and the invalidation event is published to other nodes over the pub/sub channel (`CACHE_INVALIDATION_CHANNEL`).
- Caches can be switched off per name via `services.cache.disabledCaches`; code must work when a cache is disabled (get/put/remove become no-ops).
- Mongo is only used for the membership cache persisted on `ActiveUser` with an expiry date; do not use Mongo as a general cache.
