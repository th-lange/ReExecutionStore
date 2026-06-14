# TICKET-017 — Drop Rules

## Summary

Implement the drop rule system that allows EchoChamber to silently discard captured
requests **before persistence**. Rules are stored in the DB, managed via Spring Data REST,
evaluated in-memory from a cache, and applied at the ingest endpoint after auth but before
any write.

## Acceptance Criteria

- [ ] `DropRule` domain model added to `domain/model/` (all `val`, `data class`)
- [ ] `DropAction` enum added to `domain/model/` with values `DROP` and `KEEP`
- [ ] `drop_rules` Flyway migration (`V2__add_drop_rules.sql`) creates the table with columns: `id`, `name`, `enabled`, `priority`, `action`, `method_pattern`, `path_pattern`, `authority_pattern`, `header_matches` (TEXT/JSON), `created_at`, `updated_at`
- [ ] `DropRuleEntity` JPA entity maps to `drop_rules` table
- [ ] `DropRuleJpaRepository` extends `JpaRepository` (full CRUD — rules are mutable)
- [ ] `StorageAdapter` gains `findAllDropRules(): List<DropRule>` method
- [ ] `JpaStorageAdapter` implements `findAllDropRules`
- [ ] `DropRuleEvaluator` in `application/` evaluates an ordered list of rules against a candidate request:
  - Rules sorted by `priority` ascending, `createdAt` ascending as tiebreaker
  - Only `enabled = true` rules evaluated
  - A rule matches if **all** non-null conditions match (AND logic)
  - First matching rule's action is applied: `DROP` discards, `KEEP` persists immediately and skips remaining rules
  - If no rule matches, request is persisted (default-allow)
  - `pathPattern` and `authorityPattern` evaluated as Java `Regex`; invalid patterns logged at `error` and skipped (never crash ingest)
  - `headerMatches` values are regex patterns; header key match is exact, case-insensitive
- [ ] Drop rule cache in `application/DropRuleCache`:
  - Loaded from DB on first use and on any write to `drop_rules`
  - 60-second TTL as a safety net
  - Cache invalidation triggered via Spring `@CacheEvict` or `@EventListener` on repository write
  - Evaluation is synchronous and in-memory — no IO on the ingest hot path
- [ ] `IngestionService` calls `DropRuleEvaluator` before `StorageAdapter.appendRequest`
- [ ] `/internal/ingest` always returns `202 Accepted` regardless of drop or store outcome
- [ ] `drop_rules` exposed via Spring Data REST at `/api/dropRules` (GET, POST, PUT, DELETE)

## Dependencies

- **Blocked by:** TICKET-006 (Ingestion Service & Endpoint)
- **Blocked by:** TICKET-004 (JPA Entities & Flyway Migrations — migration numbering)

## Technical Notes

- `DropRuleEvaluator` lives in `application/` — no Spring, JPA, or web imports
- Cache lives in `application/` — it depends on `StorageAdapter` port, not on `JpaStorageAdapter` directly
- Regex patterns must be compiled at cache-load time, not per-request — catch `PatternSyntaxException` at load
- `headerMatches` is stored as a JSON TEXT column; deserialised by the JPA mapper via Jackson (same pattern as `ExecutionConfig.headerOverrides`)
- Drop rules are never applied to replay executions — only to ingest
- Do not return different HTTP status codes from `/internal/ingest` based on drop/store outcome

## Tests Required

- Unit test for `DropRuleEvaluator`:
  * Matching `DROP` rule → request discarded
  * Matching `KEEP` rule → request persisted, remaining rules skipped
  * Disabled rule → ignored, default-allow applies
  * No matching rule → request persisted (default-allow)
  * Invalid regex in `pathPattern` → rule skipped, no exception propagated
  * AND logic: all conditions must match; partial match is not a match
  * Priority ordering: lower priority value evaluated first
- Unit test for `DropRuleCache`: cache invalidated on write, TTL fallback
- Integration test (`@SpringBootTest` + Testcontainers):
  * Ingest with matching DROP rule → 202, no row in `captured_requests`
  * Ingest with matching KEEP rule → 202, row persisted, lower-priority DROP rule not applied
  * Ingest with no rules → 202, row persisted
  * CRUD round-trip via Spring Data REST `/api/dropRules`
  * Cache invalidated after POST to `/api/dropRules`
