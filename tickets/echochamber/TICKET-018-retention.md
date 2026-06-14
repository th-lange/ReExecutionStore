# TICKET-018 — Retention (TTL)

## Summary

Implement a scheduled cleanup task that hard-deletes `captured_requests` rows older than
a configurable TTL. This is the **only** code path permitted to delete from
`captured_requests`.

## Acceptance Criteria

- [ ] `StorageAdapter` gains one new method: `suspend fun deleteRequestsOlderThan(cutoff: Instant): Long` (returns count deleted)
- [ ] `CapturedRequestJpaRepository` gains a `@Modifying @Query` method: `DELETE FROM captured_requests WHERE captured_at < :cutoff`; repository still exposes no other delete methods
- [ ] `JpaStorageAdapter` implements `deleteRequestsOlderThan` dispatched on `Dispatchers.IO`
- [ ] `RetentionService` in `application/` contains a single `@Scheduled` method that:
  - Calculates `cutoff = Instant.now().minus(ttlDays, ChronoUnit.DAYS)`
  - Calls `StorageAdapter.deleteRequestsOlderThan(cutoff)`
  - Logs deleted count at `info` level: `"Retention: deleted $count captured_requests older than $cutoff"`
- [ ] Retention is controlled by three config properties (read from env vars with defaults):
  - `echochamber.retention.ttl-days` / `RETENTION_TTL_DAYS` — default `14`
  - `echochamber.retention.cron` / `RETENTION_CRON` — default `0 3 * * *` (03:00 daily)
  - `echochamber.retention.enabled` / `RETENTION_ENABLED` — default `true`; when `false` the `@Scheduled` method exits immediately without querying the DB
- [ ] `@EnableScheduling` added to the application (or a `@Configuration` class)
- [ ] `execution_logs` rows are **not** deleted by this task — retention applies to `captured_requests` only
- [ ] `PortContractTest` updated to include `deleteRequestsOlderThan` in the `InMemoryStorageAdapter` stub

## Dependencies

- **Blocked by:** TICKET-004 (JPA Entities & Flyway Migrations — `captured_at` index already in V1)
- **No dependency on TICKET-017** — can be implemented in parallel

## Technical Notes

- `RetentionService` lives in `application/` — no Spring web or JPA imports; only `@Scheduled`, `@Service`, and `@Value`/`@ConfigurationProperties` are permitted
- The `@Modifying` query must run in a transaction; use `@Transactional` on the repository method (Spring Data handles this)
- `Dispatchers.IO` must be used in the JPA adapter — same pattern as all other blocking repository calls in `JpaStorageAdapter`
- `captured_at` index already exists from `V1__init.sql` — do not add a duplicate migration
- Do not use `pg_cron` or any DB-side scheduler — the Spring `@Scheduled` approach is sufficient and keeps the implementation self-contained

## Tests Required

- Unit test for `RetentionService`:
  * `enabled = true`: verifies `deleteRequestsOlderThan` called with correct cutoff (`now - ttlDays`)
  * `enabled = false`: verifies `deleteRequestsOlderThan` is never called
  * Deleted count logged at `info` level
- Integration test (`@SpringBootTest` + Testcontainers):
  * Insert rows at `now - 20 days`, `now - 10 days`, `now - 1 day`
  * Trigger `RetentionService` directly (call the method, do not wait for cron)
  * With default TTL (14 days): row at `now - 20 days` deleted; rows at `now - 10 days` and `now - 1 day` retained
  * Verify `execution_logs` rows are unaffected
