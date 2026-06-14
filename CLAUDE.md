# SnapReq + EchoChamber — System Overview

This workspace contains two projects that work together as a single system for HTTP request capture and replay.

| Project | Language | Role | Agent rules |
|---|---|---|---|
| `snapreq/` | Go | Intercept inbound HTTP requests; forward to EchoChamber | [snapreq/Agent.md](snapreq/Agent.md) |
| `echochamber/` | Kotlin / Spring Boot | Store, filter, and replay captured requests | [echochamber/Agent.md](echochamber/Agent.md) |

**Before working on either project, read its Agent.md in full.**  
If a task spans both projects (e.g. changing the ingest contract), read both Agent.md files before writing any code.

---

## System Architecture

```
                    ┌─────────────────────────────────────────┐
                    │              SnapReq                     │
  Inbound           │                                          │
  HTTP ────────────►│  capture (goroutine, fire-and-forget)   │──── POST /internal/ingest ──►┐
  request           │                                          │                              │
                    │  [if FORWARD_URL set]                    │                              │
                    │  forward ──────────────────────────────► upstream                       │
                    └─────────────────────────────────────────┘                              │
                                                                                              ▼
                                                                                   ┌─────────────────────┐
                                                                                   │    EchoChamber       │
                                                                                   │                      │
                                                                                   │  [drop rule eval]    │
                                                                                   │        ↓             │
                                                                                   │  [persist or drop]   │
                                                                                   │        ↓             │
                                                                                   │  [TTL retention]     │
                                                                                   │        ↓             │
                                                                                   │  [replay jobs]       │
                                                                                   └─────────────────────┘
```

---

## The Shared Contract

The **only coupling** between SnapReq and EchoChamber is the ingest payload JSON shape sent to `POST /internal/ingest`.

```json
{
  "method":    "GET",
  "uri":       "https://example.com/api/resource?q=1",
  "authority": "example.com",
  "headers":   { "accept": "application/json" },
  "body":      null
}
```

**Rules that apply to both sides simultaneously:**
- `capturedAt` is **not** in the payload — EchoChamber stamps it on receipt.
- `body` is `null` when absent, never omitted, never an empty string.
- `headers` is a flat map — one value per header name.
- Any change to this schema requires a PR touching **both** `snapreq/` and `echochamber/` in the same commit or back-to-back PRs with a clear dependency note.
- EchoChamber always returns `202 Accepted`. SnapReq does not inspect the response body.

---

## Cross-Project Change Protocol

When a change affects the ingest contract or the auth token mechanism:

1. Open a GitHub issue labelled `cross-project` describing the change.
2. Implement the EchoChamber side first (it is the authority on the contract).
3. Implement the SnapReq side second, referencing the EchoChamber PR.
4. Both PRs must be merged before either is considered done.

---

## Environment Variables Reference

### SnapReq

| Variable | Required | Default | Description |
|---|---|---|---|
| `ECHOCHAMBER_URL` | Yes | — | Base URL of EchoChamber |
| `ECHOCHAMBER_TOKEN` | Yes | — | Must match EchoChamber's `INTERNAL_INGEST_TOKEN` |
| `LISTEN_ADDR` | No | `:8080` | SnapReq listen address |
| `FORWARD_URL` | No | — | If set, activates in-path proxy mode |
| `FORWARD_TIMEOUT_MS` | No | `5000` | Forward leg timeout |
| `CAPTURE_TIMEOUT_MS` | No | `2000` | EchoChamber POST timeout |
| `MAX_BODY_BYTES` | No | `1048576` | Body capture limit (1 MiB) |
| `LOG_LEVEL` | No | `info` | Log verbosity |
| `CAPTURE_AUTH_HEADERS` | No | `false` | If `true`, include `Authorization` header in captured payload |

### EchoChamber

| Variable | Required | Default | Description |
|---|---|---|---|
| `INTERNAL_INGEST_TOKEN` | Yes | — | Bearer token SnapReq uses to authenticate |
| `DB_HOST` | No | `localhost` | PostgreSQL host |
| `DB_PORT` | No | `5432` | PostgreSQL port |
| `DB_NAME` | No | `echochamber` | Database name |
| `DB_USER` | No | `echochamber` | Database user |
| `DB_PASSWORD` | No | `echochamber` | Database password |
| `RETENTION_TTL_DAYS` | No | `14` | Days to keep captured requests |
| `RETENTION_CRON` | No | `0 3 * * *` | Cron for retention cleanup |
| `RETENTION_ENABLED` | No | `true` | Enable/disable retention |

---

## Local Development

### Full stack (both services)

```bash
cp .env.example .env
# Set INTERNAL_INGEST_TOKEN to any non-empty string
docker compose up --build
```

### EchoChamber only

```bash
docker compose up -d postgres
cd echochamber && ./gradlew bootRun
```

### SnapReq only (mirror mode, pointing at a running EchoChamber)

```bash
ECHOCHAMBER_URL=http://localhost:8080 \
ECHOCHAMBER_TOKEN=change-me \
LISTEN_ADDR=:9090 \
go run ./cmd/snapreq
```

---

## Key Endpoints

| Service | URL | Description |
|---|---|---|
| EchoChamber | `http://localhost:8080/swagger-ui.html` | API docs |
| EchoChamber | `http://localhost:8080/api/explorer` | HAL Explorer admin UI |
| EchoChamber | `http://localhost:8080/api/capturedRequests` | Browse captured requests |
| EchoChamber | `http://localhost:8080/api/dropRules` | Manage drop rules |
| EchoChamber | `http://localhost:8080/api/executionConfigs` | Manage replay configs |
| EchoChamber | `http://localhost:8080/api/replayJobs` | Browse replay jobs |
| EchoChamber | `http://localhost:8080/internal/ingest` | Ingest endpoint (Bearer auth) |
| SnapReq | `http://localhost:9090/` | Capture endpoint |

---

## What This System Is Not

- **Not a load balancer.** SnapReq does not do health checking or weighted routing.
- **Not a full API gateway.** SnapReq does no auth, rate limiting, or routing logic.
- **Not a streaming/real-time system.** Replay is batch-oriented and asynchronous.
- **Not an event bus.** There is no pub/sub, no webhooks, no streaming from EchoChamber to consumers.

These constraints are intentional. Scope creep in either direction makes the system harder to operate and reason about.
