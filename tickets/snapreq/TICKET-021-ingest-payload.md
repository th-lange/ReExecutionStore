# TICKET-021 — SnapReq: Ingest Payload (Shared Contract)

**Project:** SnapReq (Go)

## Summary
Implement `internal/ingest/payload.go` — the `IngestPayload` struct that defines the JSON shape sent to EchoChamber's `POST /internal/ingest`. This is the **source of truth** for the shared contract on the SnapReq side.

## Acceptance Criteria
- [ ] `IngestPayload` struct defined with exact JSON field names matching the contract
- [ ] `Body` is `*string` (pointer) — `nil` when absent, never empty string
- [ ] `Headers` is `map[string]string` — single value per name
- [ ] Struct serialises to the canonical JSON shape (verified by test)
- [ ] No framework imports — stdlib only

## Dependencies
- **Blocked by:** TICKET-019 (project scaffold)

## Technical Notes

```go
// internal/ingest/payload.go
type IngestPayload struct {
    Method    string            `json:"method"`
    URI       string            `json:"uri"`
    Authority string            `json:"authority"`
    Headers   map[string]string `json:"headers"`
    Body      *string           `json:"body"`
}
```

- `capturedAt` is **not** a field — EchoChamber stamps it on receipt.
- Any future change to this struct requires a simultaneous PR to EchoChamber's ingest DTO (cross-project change protocol in CLAUDE.md).
- This struct must stay in sync with EchoChamber's `CapturedRequestDto`.

## Tests Required
- Serialise full payload → verify exact JSON output (field names, `body: null`)
- Deserialise JSON with `"body": null` → `Body` field is `nil`
- Deserialise JSON with body string → `Body` is non-nil pointer to that string
- Empty `Headers` map → serialises as `{}`, not `null`
