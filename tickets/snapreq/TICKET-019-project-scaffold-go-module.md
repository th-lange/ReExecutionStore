# TICKET-019 — SnapReq: Project Scaffold & Go Module

**Project:** SnapReq (Go)

## Summary
Initialise the Go module and project directory structure. No business logic — foundation only. Sets up the build, containerisation, and CI pipeline so every subsequent ticket has a working base to land on.

## Acceptance Criteria
- [ ] `go.mod` declares the module (`github.com/th-lange/snapreq`) and minimum Go version (1.22+)
- [ ] Directory tree matches the architecture spec: `cmd/snapreq/`, `internal/capture/`, `internal/forward/`, `internal/ingest/`, `internal/config/`
- [ ] Placeholder `package` declarations in each directory so `go build ./...` compiles cleanly
- [ ] Multi-stage `Dockerfile` produces a minimal scratch/distroless binary image
- [ ] `snapreq` service added to the workspace `docker-compose.yml` with required env vars stubbed
- [ ] GitHub Actions workflow runs `go build ./...`, `go vet ./...`, `staticcheck ./...`, `go test ./...` on every push/PR
- [ ] `README.md` updated with build and run instructions

## Dependencies
None — this is the root SnapReq ticket.

## Technical Notes
- Module path: `github.com/th-lange/snapreq`
- No external dependencies required at this stage (stdlib only per Agent.md §2)
- Dockerfile: build stage (`golang:1.22-alpine`), final stage (`scratch` or `gcr.io/distroless/static`)
- `staticcheck` installable via `go install honnef.co/go/tools/cmd/staticcheck@latest`

## Tests Required
- Smoke test: `go build ./...` and `go vet ./...` pass with zero warnings
