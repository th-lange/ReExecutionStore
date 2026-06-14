# ReExecutionStore

Umbrella repository for the projects that capture and re-execute HTTP requests.
It ties the individual services together and holds the shared documentation,
contract, and local-development setup.

Today it spans two repositories (more may follow):

| Project | Language | Role |
|---|---|---|
| [SnapReq](https://github.com/th-lange/SnapReq) | Go | Intercepts inbound HTTP requests and forwards them to EchoChamber (optionally proxying in-path). |
| [EchoChamber](https://github.com/th-lange/EchoChamber) | Kotlin / Spring Boot | Stores captured requests, filters them, and replays them against any target. |

## How it fits together

```
inbound HTTP ──► SnapReq ──► POST /internal/ingest ──► EchoChamber ──► store · filter · replay
```

SnapReq captures fire-and-forget; EchoChamber is the system of record and owns
the replay lifecycle. Their only coupling is the ingest payload contract.

## Where to go next

- **[CLAUDE.md](CLAUDE.md)** — full system overview: architecture, the shared ingest
  contract, cross-project change protocol, environment variables, local development,
  and key endpoints. Read this before working on either service.
- **Project board** — work is tracked on the
  ["Request Store" board](https://github.com/users/th-lange/projects/1).

## Quick start

```bash
cp .env.example .env
# set INTERNAL_INGEST_TOKEN to any non-empty string
docker compose up --build
```

See [CLAUDE.md](CLAUDE.md) for running each service on its own.
