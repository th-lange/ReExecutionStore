# TICKET-023 — SnapReq: Ingest Client (Fire-and-Forget)

**Project:** SnapReq (Go)

## Summary
Implement `internal/ingest/client.go` — the HTTP client that POSTs captured payloads to EchoChamber's `/internal/ingest` endpoint. Always called from a goroutine; never blocks the hot path.

## Acceptance Criteria
- [ ] `Client` struct wraps a single shared `*http.Client` (connection-pooled, explicit timeout)
- [ ] `NewClient(echochamberURL, token string, timeout time.Duration) *Client` — initialised once at startup, not per-request
- [ ] `Send(ctx context.Context, payload IngestPayload) error` serialises the payload and POSTs to `<echochamberURL>/internal/ingest`
- [ ] `Authorization: Bearer <token>` header set on every request
- [ ] `Content-Type: application/json` set
- [ ] Non-2xx response is treated as an error (caller logs at `warn`)
- [ ] Token is **never** logged
- [ ] The `ctx` passed in carries the `CAPTURE_TIMEOUT_MS` deadline (set by caller via `context.WithTimeout`)
- [ ] No retry logic — Agent.md §6 is explicit

## Dependencies
- **Blocked by:** TICKET-019 (project scaffold)
- **Blocked by:** TICKET-020 (config — for timeout value)
- **Blocked by:** TICKET-021 (ingest payload struct)

## Technical Notes
- The `http.Client` timeout is a safety net; the per-call `context.WithTimeout` is the primary timeout mechanism
- EchoChamber always returns `202 Accepted`; any other status should be returned as an error

## Tests Required
Using `net/http/httptest`:
- Valid payload → mock server receives correct JSON body, correct `Authorization: Bearer` header, correct path (`/internal/ingest`)
- Non-2xx from mock server → error returned
- Context cancelled before send → error returned promptly (no hang)
- Token does not appear in any logged output during the test
