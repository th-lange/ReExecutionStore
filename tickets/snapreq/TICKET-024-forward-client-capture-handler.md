# TICKET-024 — SnapReq: Forward Client & Capture Handler

**Project:** SnapReq (Go)

## Summary
Implement `internal/forward/client.go` and `internal/capture/handler.go` — the two files that form SnapReq's hot path. The handler is the `http.Handler` registered for every inbound request; the forward client handles Mode A upstream proxying.

## Acceptance Criteria

### Forward client (`internal/forward/client.go`)
- [ ] Single shared `*http.Client` with explicit timeout (`FORWARD_TIMEOUT_MS`), separate instance from the ingest client
- [ ] `Forward(ctx context.Context, r *http.Request, targetURL string, body []byte) (*http.Response, error)` builds and sends the upstream request
- [ ] Copies original method, path, query, and headers to the upstream request (hop-by-hop headers stripped)
- [ ] Returns an error on upstream failure (caller responds `502`)

### Capture handler (`internal/capture/handler.go`)
- [ ] **Mode A** (`FORWARD_URL` set): fires capture goroutine **before** starting the forward dial; forwards upstream response to client; returns `502` on forward failure
- [ ] **Mode B/D** (`FORWARD_URL` not set): returns `204 No Content` immediately; capture goroutine fires asynchronously
- [ ] Capture goroutine: `context.WithTimeout(CAPTURE_TIMEOUT_MS)`, `recover()` from panics, logs errors at `warn`, never propagates to caller
- [ ] Header processing: collapse multi-value headers (last wins); strip hop-by-hop headers (`Connection`, `Transfer-Encoding`, `Keep-Alive`, `Upgrade`, `Proxy-*`)
- [ ] Strip `Authorization` from captured payload unless `CAPTURE_AUTH_HEADERS=true`
- [ ] `URI` construction: if request URI is absolute, use as-is; otherwise `<scheme>://<authority><path>?<query>` where scheme comes from `X-Forwarded-Proto` (fallback `http`)
- [ ] `Authority` derived from `Host` header, fallback `r.Host`

## Dependencies
- **Blocked by:** TICKET-020 (config)
- **Blocked by:** TICKET-021 (ingest payload)
- **Blocked by:** TICKET-022 (body buffering)
- **Blocked by:** TICKET-023 (ingest client)

## Technical Notes
- The capture goroutine must fire **before** the forward dial in Mode A — this is the Prime Directive (Agent.md §0)
- Body buffering in Mode A: read once into `[]byte`; full buffer to forward, truncated copy to capture
- Hop-by-hop headers to strip: `Connection`, `Keep-Alive`, `Proxy-Authenticate`, `Proxy-Authorization`, `Proxy-Connection`, `TE`, `Trailer`, `Transfer-Encoding`, `Upgrade`

## Tests Required
Using `net/http/httptest`:
- **Mode B/D:** request in → `204` returned; mock EchoChamber receives correct `IngestPayload`
- **Mode A:** request in → upstream mock receives it and responds; client gets upstream's response; EchoChamber mock receives payload
- **Mode A forward failure:** upstream errors → client gets `502`; capture goroutine still fires
- `Authorization` stripped when `CAPTURE_AUTH_HEADERS=false` (default)
- `Authorization` present in payload when `CAPTURE_AUTH_HEADERS=true`
- Hop-by-hop headers absent from captured payload
- `X-Forwarded-Proto: https` → URI scheme is `https`
- Multi-value header → single value (last-wins) in captured payload
- Capture goroutine panic → recovered, handler does not crash
