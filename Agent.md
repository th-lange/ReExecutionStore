# Agent Instructions — ReExecution Workspace

This file captures workspace-wide agent guidance, including the live status of the
GitHub project board. For per-project rules, read:

- [SnapReq/Agent.md](SnapReq/Agent.md)
- [EchoChamber/Agent.md](EchoChamber/Agent.md)

See [CLAUDE.md](CLAUDE.md) for the system overview and the SnapReq ⇄ EchoChamber contract.

---

## Project Board — "Request Store"

- **Board:** https://github.com/users/th-lange/projects/1 (project #1, owner `th-lange`)
- **Status columns:** Backlog → Ready → In progress → In review → Done
- **Other fields:** Priority (P0/P1/P2), Size (XS–XL), Estimate, Start date, Target date
- **Snapshot taken:** 2026-06-14

> Keep this section in sync when the board changes. Refresh with:
> `gh api graphql -f query='query{user(login:"th-lange"){projectV2(number:1){items(first:100){nodes{type content{... on Issue{number title url state}} fieldValues(first:30){nodes{... on ProjectV2ItemFieldSingleSelectValue{name field{... on ProjectV2FieldCommon{name}}}}}}}}}}'`
> (requires a `gh` token with the `read:project` scope)

## Ticket Specs — Layout

Ticket specs live under [`tickets/`](tickets/), one folder per project. Each ticket keeps its
globally-unique `TICKET-NNN` number (numbers are never reused across projects), and the filename
matches the issue title. EchoChamber ticket issues live in the `th-lange/EchoChamber` repo;
SnapReq ticket issues live in `th-lange/snapreq`.

```
tickets/
  echochamber/   TICKET-001 … TICKET-018   (Kotlin / Spring Boot)
  snapreq/       TICKET-019 … TICKET-025   (Go)
```

### Done — EchoChamber (5)

| Ticket | Title | Spec | Issue |
|---|---|---|---|
| TICKET-001 | Project Scaffold & Build Configuration | [spec](tickets/echochamber/TICKET-001-project-scaffold-build-configuration.md) | [#1](https://github.com/th-lange/EchoChamber/issues/1) |
| TICKET-002 | Domain Models | [spec](tickets/echochamber/TICKET-002-domain-models.md) | [#2](https://github.com/th-lange/EchoChamber/issues/2) |
| TICKET-003 | Port Interfaces | [spec](tickets/echochamber/TICKET-003-port-interfaces.md) | [#3](https://github.com/th-lange/EchoChamber/issues/3) |
| TICKET-004 | JPA Entities & Flyway Migrations | [spec](tickets/echochamber/TICKET-004-jpa-entities-flyway-migrations.md) | [#4](https://github.com/th-lange/EchoChamber/issues/4) |
| TICKET-005 | Internal Auth Filter | [spec](tickets/echochamber/TICKET-005-internal-auth-filter.md) | [#5](https://github.com/th-lange/EchoChamber/issues/5) |

### Ready — EchoChamber (13)

| Ticket | Title | Spec | Issue |
|---|---|---|---|
| TICKET-006 | Ingestion Service & Endpoint | [spec](tickets/echochamber/TICKET-006-ingestion-service-endpoint.md) | [#6](https://github.com/th-lange/EchoChamber/issues/6) |
| TICKET-007 | Spring Data REST & HAL Explorer | [spec](tickets/echochamber/TICKET-007-spring-data-rest-hal-explorer.md) | [#7](https://github.com/th-lange/EchoChamber/issues/7) |
| TICKET-008 | Mutation Engine Core | [spec](tickets/echochamber/TICKET-008-mutation-engine-core.md) | [#8](https://github.com/th-lange/EchoChamber/issues/8) |
| TICKET-009 | Built-in Mutation Handlers | [spec](tickets/echochamber/TICKET-009-built-in-mutation-handlers.md) | [#9](https://github.com/th-lange/EchoChamber/issues/9) |
| TICKET-010 | Script Mutation Handler (GraalVM) | [spec](tickets/echochamber/TICKET-010-script-mutation-handler-graalvm.md) | [#10](https://github.com/th-lange/EchoChamber/issues/10) |
| TICKET-011 | HTTP Executor | [spec](tickets/echochamber/TICKET-011-http-executor.md) | [#11](https://github.com/th-lange/EchoChamber/issues/11) |
| TICKET-012 | Replay Job Scheduler | [spec](tickets/echochamber/TICKET-012-replay-job-scheduler.md) | [#12](https://github.com/th-lange/EchoChamber/issues/12) |
| TICKET-013 | Replay Service (async, rate-limited) | [spec](tickets/echochamber/TICKET-013-replay-service-async-rate-limited.md) | [#13](https://github.com/th-lange/EchoChamber/issues/13) |
| TICKET-014 | Action Endpoints (Trigger & Cancel) | [spec](tickets/echochamber/TICKET-014-action-endpoints-trigger-cancel.md) | [#14](https://github.com/th-lange/EchoChamber/issues/14) |
| TICKET-015 | R2DBC Storage Adapter | [spec](tickets/echochamber/TICKET-015-r2dbc-storage-adapter.md) | [#15](https://github.com/th-lange/EchoChamber/issues/15) |
| TICKET-016 | Integration Tests (End-to-End) | [spec](tickets/echochamber/TICKET-016-integration-tests-end-to-end.md) | [#16](https://github.com/th-lange/EchoChamber/issues/16) |
| TICKET-017 | Drop Rules | [spec](tickets/echochamber/TICKET-017-drop-rules.md) | [#23](https://github.com/th-lange/EchoChamber/issues/23) |
| TICKET-018 | Retention (TTL) | [spec](tickets/echochamber/TICKET-018-retention.md) | [#24](https://github.com/th-lange/EchoChamber/issues/24) |

### Ready — SnapReq (7)

> **Note:** create the matching GitHub issues in `th-lange/snapreq` (enable Issues on that repo
> first if needed), then fill in the Issue column.

| Ticket | Title | Spec | Issue |
|---|---|---|---|
| TICKET-019 | Project Scaffold & Go Module | [spec](tickets/snapreq/TICKET-019-project-scaffold-go-module.md) | — |
| TICKET-020 | Config Package | [spec](tickets/snapreq/TICKET-020-config-package.md) | — |
| TICKET-021 | Ingest Payload (Shared Contract) | [spec](tickets/snapreq/TICKET-021-ingest-payload.md) | — |
| TICKET-022 | Body Buffering | [spec](tickets/snapreq/TICKET-022-body-buffering.md) | — |
| TICKET-023 | Ingest Client (Fire-and-Forget) | [spec](tickets/snapreq/TICKET-023-ingest-client.md) | — |
| TICKET-024 | Forward Client & Capture Handler | [spec](tickets/snapreq/TICKET-024-forward-client-capture-handler.md) | — |
| TICKET-025 | Main Entrypoint & Integration Tests | [spec](tickets/snapreq/TICKET-025-main-entrypoint-integration-tests.md) | — |

### In progress / In review / Backlog

None.

---

## Working Notes

- **EchoChamber:** the foundation (build, domain models, ports, persistence entities, auth filter)
  is **Done**. The remaining Ready work is the functional core: ingestion → mutation → replay
  execution, plus the REST surface, storage adapter, drop rules, retention, and end-to-end tests.
  Suggested build order follows the ticket numbering; ingestion (TICKET-006) and the REST surface
  (TICKET-007) unblock most downstream work. TICKET-017 (Drop Rules) is blocked by TICKET-006;
  TICKET-018 (Retention) can be parallelised after TICKET-004.
- **SnapReq:** currently a blank slate (Agent.md + CLAUDE.md only, no Go code). TICKET-019 through
  TICKET-025 define the full phase 1 implementation in dependency order:
  `019 → 020 + 021 + 022 (parallel) → 023 → 024 → 025`. All SnapReq tickets are independent of
  EchoChamber's open tickets and can be worked in parallel.
