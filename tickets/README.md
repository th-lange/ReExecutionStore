# Tickets

Implementation tickets for the ReExecution system, one folder per project.

```
tickets/
  echochamber/   TICKET-001 … TICKET-018   (Kotlin / Spring Boot)
  snapreq/       TICKET-019 … TICKET-025   (Go)
```

## Conventions

- **Numbering is global.** A `TICKET-NNN` number is unique across the whole system and never
  reused, so any ticket can be referenced unambiguously regardless of project. New tickets take
  the next free number.
- **One file per ticket.** Filename: `TICKET-NNN-kebab-title.md`. The folder names the project,
  so the filename does not repeat it.
- **One spec, one issue.** Each ticket file is the source-of-truth spec. The matching GitHub issue
  lives in that project's repo (`th-lange/EchoChamber` or `th-lange/snapreq`). The live board status
  and issue links are tracked in [`../Agent.md`](../Agent.md).
- **Spec shape:** Summary → Acceptance Criteria → Dependencies → Technical Notes → Tests Required.
  Dependencies reference other tickets by number (`Blocked by: TICKET-006`).

## Adding a project

Create `tickets/<project>/`, continue the global numbering, and add a section + status table to
[`../Agent.md`](../Agent.md). No changes to existing folders are needed.
