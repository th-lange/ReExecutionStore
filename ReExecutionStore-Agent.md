# Agent Instructions — ReExecution Workspace

This file captures workspace-wide agent guidance, including the live status of the
GitHub project board. For per-project rules, read:

- [SnapReq Agent.md](https://github.com/th-lange/SnapReq/blob/main/Agent.md)
- [EchoChamber Agent.md](https://github.com/th-lange/EchoChamber/blob/main/Agent.md)

See [CLAUDE.md](https://github.com/th-lange/ReExecutionStore/blob/main/CLAUDE.md) for the system overview and the SnapReq ⇄ EchoChamber contract.

---

## Project Board — "Request Store"

- **Board:** https://github.com/users/th-lange/projects/1 (project #1, owner `th-lange`)
- **Status columns:** Backlog → Ready → In progress → In review → Done
- **Other fields:** Priority (P0/P1/P2), Size (XS–XL), Estimate, Start date, Target date
- **Snapshot taken:** 2026-06-14
- All current items live in the `th-lange/EchoChamber` repo. No SnapReq tickets are on the board yet.

> Keep this section in sync when the board changes. Refresh with:
> `gh api graphql -f query='query{user(login:"th-lange"){projectV2(number:1){items(first:100){nodes{type content{... on Issue{number title url state}} fieldValues(first:30){nodes{... on ProjectV2ItemFieldSingleSelectValue{name field{... on ProjectV2FieldCommon{name}}}}}}}}}}'`
> (requires a `gh` token with the `read:project` scope)

### Done (5)

| Ticket | Title | Issue |
|---|---|---|
| TICKET-001 | Project Scaffold & Build Configuration | [#1](https://github.com/th-lange/EchoChamber/issues/1) |
| TICKET-002 | Domain Models | [#2](https://github.com/th-lange/EchoChamber/issues/2) |
| TICKET-003 | Port Interfaces | [#3](https://github.com/th-lange/EchoChamber/issues/3) |
| TICKET-004 | JPA Entities & Flyway Migrations | [#4](https://github.com/th-lange/EchoChamber/issues/4) |
| TICKET-005 | Internal Auth Filter | [#5](https://github.com/th-lange/EchoChamber/issues/5) |

### Ready (13)

| Ticket | Title | Issue |
|---|---|---|
| TICKET-006 | Ingestion Service & Endpoint | [#6](https://github.com/th-lange/EchoChamber/issues/6) |
| TICKET-007 | Spring Data REST & HAL Explorer | [#7](https://github.com/th-lange/EchoChamber/issues/7) |
| TICKET-008 | Mutation Engine Core | [#8](https://github.com/th-lange/EchoChamber/issues/8) |
| TICKET-009 | Built-in Mutation Handlers | [#9](https://github.com/th-lange/EchoChamber/issues/9) |
| TICKET-010 | Script Mutation Handler (GraalVM) | [#10](https://github.com/th-lange/EchoChamber/issues/10) |
| TICKET-011 | HTTP Executor | [#11](https://github.com/th-lange/EchoChamber/issues/11) |
| TICKET-012 | Replay Job Scheduler | [#12](https://github.com/th-lange/EchoChamber/issues/12) |
| TICKET-013 | Replay Service (async, rate-limited) | [#13](https://github.com/th-lange/EchoChamber/issues/13) |
| TICKET-014 | Action Endpoints (Trigger & Cancel) | [#14](https://github.com/th-lange/EchoChamber/issues/14) |
| TICKET-015 | R2DBC Storage Adapter | [#15](https://github.com/th-lange/EchoChamber/issues/15) |
| TICKET-016 | Integration Tests (End-to-End) | [#16](https://github.com/th-lange/EchoChamber/issues/16) |
| TICKET-017 | Drop Rules | [#23](https://github.com/th-lange/EchoChamber/issues/23) |
| TICKET-018 | Retention (TTL) | [#24](https://github.com/th-lange/EchoChamber/issues/24) |

### In progress / In review / Backlog

None.

---

## Working Notes

- The foundation (build, domain models, ports, persistence entities, auth filter) is **Done**.
  The remaining Ready work is the functional core: ingestion → mutation → replay execution,
  plus the REST surface, storage adapter, and end-to-end tests.
- Suggested build order follows the ticket numbering; ingestion (TICKET-006) and the REST
  surface (TICKET-007) unblock most downstream work.
- TICKET-017 (Drop Rules) is blocked by TICKET-006. It must be implemented before TICKET-016
  (Integration Tests) to ensure end-to-end tests cover drop rule behaviour.
- TICKET-018 (Retention) has no dependency on TICKET-017 and can run in parallel with it.
