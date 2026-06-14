# TICKET-025 — SnapReq: Main Entrypoint & Integration Tests

**Project:** SnapReq (Go)

## Summary
Implement `cmd/snapreq/main.go` — the binary entrypoint that wires config, clients, handler, and HTTP server. Add the integration test suite that verifies SnapReq end-to-end in each deployment mode. This is the **Definition of Done gate** for SnapReq phase 1.

## Acceptance Criteria

### Main entrypoint
- [ ] `main.go` calls `config.Load()`, initialises shared HTTP clients, constructs the handler, and starts `http.ListenAndServe`
- [ ] Active mode logged clearly at `info` on startup: `mode=proxy forward_url=<url>` or `mode=mirror`
- [ ] All resolved config (except token) logged at `info` before server starts
- [ ] No logic in `main.go` beyond wiring and `log.Fatal` — no `if` branches except mode detection
- [ ] Graceful shutdown on `SIGTERM`/`SIGINT` via `http.Server.Shutdown` with a timeout context

### Integration tests
- [ ] **Mode B/D:** SnapReq started in-process against a `httptest.Server` mock EchoChamber; send a POST with body → mock receives correct `IngestPayload` shape; SnapReq returns `204`
- [ ] **Mode A:** SnapReq with mock upstream + mock EchoChamber; request in → upstream receives it → client gets upstream's response; EchoChamber mock receives payload concurrently
- [ ] **Payload shape verified:** `method`, `uri`, `authority`, `headers`, `body` correct in EchoChamber payload; `capturedAt` absent
- [ ] **Capture non-blocking in Mode A:** EchoChamber mock adds artificial 200ms delay; forward response still arrives before capture completes (verified by timing)
- [ ] `go test ./... -race` passes — race detector clean
- [ ] All unit tests from TICKET-020 through TICKET-024 continue to pass

## Dependencies
- **Blocked by:** TICKET-024 (forward client & capture handler — final wiring dep)

## Technical Notes
- Integration tests use `net/http/httptest` only — no Docker, no external processes
- SnapReq started in-process: construct `http.Server`, call `ListenAndServe` in a goroutine, shut down via `Shutdown` in test cleanup (`t.Cleanup`)
- All Agent.md §8 test requirements must be satisfied before this ticket closes

## Tests Required
- All integration scenarios listed in Acceptance Criteria
- `go vet ./...` zero warnings
- `staticcheck ./...` zero warnings
