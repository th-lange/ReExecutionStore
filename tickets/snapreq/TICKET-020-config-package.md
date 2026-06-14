# TICKET-020 — SnapReq: Config Package

**Project:** SnapReq (Go)

## Summary
Implement `internal/config/config.go` — all environment variable parsing and validation in one place, called once at startup. Fatal exit on missing required config.

## Acceptance Criteria
- [ ] `Config` struct holds all resolved configuration values with correct Go types
- [ ] `ECHOCHAMBER_URL` and `ECHOCHAMBER_TOKEN` are required — `log.Fatal` if either is absent or empty
- [ ] `ECHOCHAMBER_TOKEN` is **never** logged at any level
- [ ] All other resolved values are logged at `info` level on startup
- [ ] `CAPTURE_AUTH_HEADERS` defaults to `false`; parses `"true"` / `"1"` as `true`
- [ ] `MAX_BODY_BYTES` defaults to `1048576` (1 MiB), parsed as `int64`
- [ ] Integer/duration env vars (`FORWARD_TIMEOUT_MS`, `CAPTURE_TIMEOUT_MS`, `MAX_BODY_BYTES`) call `log.Fatal` on non-numeric value
- [ ] A single `Load() Config` function (no `init()` functions — Agent.md §10)
- [ ] No global mutable state — return a value, let `main.go` own it

## Dependencies
- **Blocked by:** TICKET-019 (project scaffold)

## Technical Notes

| Variable | Required | Default | Go type |
|---|---|---|---|
| `ECHOCHAMBER_URL` | Yes | — | `string` |
| `ECHOCHAMBER_TOKEN` | Yes | — | `string` |
| `LISTEN_ADDR` | No | `:8080` | `string` |
| `FORWARD_URL` | No | — | `string` (empty = Mode B/D) |
| `FORWARD_TIMEOUT_MS` | No | `5000` | `time.Duration` (ms × `time.Millisecond`) |
| `CAPTURE_TIMEOUT_MS` | No | `2000` | `time.Duration` |
| `MAX_BODY_BYTES` | No | `1048576` | `int64` |
| `LOG_LEVEL` | No | `info` | `string` |
| `CAPTURE_AUTH_HEADERS` | No | `false` | `bool` |

## Tests Required
- Missing `ECHOCHAMBER_URL` → process exits (use `exec.Command` sub-process test pattern)
- Missing `ECHOCHAMBER_TOKEN` → process exits
- All defaults applied when optional vars absent
- `CAPTURE_AUTH_HEADERS=true` → `true`; absent → `false`
- Invalid integer for `MAX_BODY_BYTES` → process exits
