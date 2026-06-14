# TICKET-022 — SnapReq: Body Buffering

**Project:** SnapReq (Go)

## Summary
Implement `internal/capture/body.go` — helpers for reading and buffering the HTTP request body, including the truncation logic and the empty-body fast path.

## Acceptance Criteria
- [ ] `ReadBody(r *http.Request, maxBytes int64) ([]byte, bool, error)` reads up to `maxBytes` from the request body
- [ ] Returns the bytes read, a `truncated bool`, and any read error
- [ ] If truncated, logs a `warn` with the actual body size vs. limit
- [ ] **No `bytes.Buffer` allocated** when the body is absent (`r.Body == nil`, `r.Body == http.NoBody`, or `Content-Length: 0`) — fast path returns `nil, false, nil`
- [ ] In Mode A (forward + capture), the full body is buffered once and re-exposed for the forward leg via `io.NopCloser`; capture gets only the truncated copy
- [ ] `r.Body` is always closed after reading

## Dependencies
- **Blocked by:** TICKET-019 (project scaffold)

## Technical Notes
- For Mode A: read the full body into a buffer once; give forward the full buffer; give capture only up to `maxBytes` of that buffer (no second read)
- Use `io.ReadAll` combined with `io.LimitReader` for the capture copy
- `Body` in `IngestPayload` is `*string` — convert `[]byte` to `string` via a pointer only when non-nil

## Tests Required
- Empty body (`http.NoBody`) → `nil` returned, no allocation
- `Content-Length: 0` → `nil` returned, no allocation
- Body exactly at `maxBytes` → all bytes returned, `truncated = false`
- Body exceeds `maxBytes` → truncated bytes returned, `truncated = true`, warn logged
- Body shorter than `maxBytes` → all bytes returned, `truncated = false`
- Read error mid-stream → bytes read so far returned, error surfaced
