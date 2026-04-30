# UniLog — Project Plan

## Overview

UniLog is a log ingestion and translation engine that writes events from arbitrary sources into the Windows Event Log as structurally native events. The long-term vision is a pluggable pipeline (sources → routes → transforms → destinations) with WEL as a first-class output target.

The project is being built in deliberate phases. This plan covers Phase 1 in detail; later phases are listed for context but will be planned in their own right when Phase 1 is complete.

---

## Goals

- Provide a high-fidelity translation path from non-Windows log sources into the Windows Event Log.
- Preserve original event data losslessly — timestamps, source hostnames, structured fields.
- Produce events that downstream Windows-native collectors consume identically to native events.
- Remain honest about provider identity. UniLog never impersonates Microsoft providers.

## Non-goals (Phase 1)

- General-purpose syslog handling beyond the targeted InfoBlox use case.
- A management UI.
- Output destinations other than the Windows Event Log.
- Pipeline transforms beyond what is required for InfoBlox field mapping.

---

## Phase 1 — InfoBlox DNS/DHCP → WEL

### Scope

Translate InfoBlox DNS and DHCP events delivered via Cribl into structured Windows Event Log entries that the Cortex XDR Collector can pull and forward into XSIAM.

### Pipeline

```
InfoBlox → Cribl (parse) → UniLog (translate) → Windows Event Log → Cortex XDR Collector → XSIAM
```

### Workstreams

1. **Translation engine.** Service that accepts structured events from Cribl and writes them to WEL.
2. **Event schema design.** Define the event ID space, field names, and channel layout for InfoBlox DNS and DHCP events.
3. **Cribl integration.** Configure the Cribl pipeline to parse InfoBlox syslog and forward structured events to UniLog.
4. **WEC / collector integration.** Validate that the Cortex XDR Collector reads UniLog events correctly and that XSIAM receives them with structured fields intact.
5. **Packaging.** Installer and service registration for deployment on a Windows host.
6. **Documentation.** Operator-facing setup docs, schema reference, and troubleshooting guide.

### Open questions

These are not yet decided and should be resolved early in Phase 1:

- Implementation language and runtime.
- Transport protocol between Cribl and UniLog.
- Event ID numbering scheme and channel naming.
- Deployment model (collocated with WEC vs. separate host).
- Test strategy and sample data sources.

### Definition of done

- UniLog runs as a Windows Service on a target host.
- InfoBlox DNS and DHCP events flowing through Cribl appear in the Windows Event Log under UniLog-owned channels.
- The Cortex XDR Collector successfully ingests those events and delivers them to XSIAM with structured fields preserved.
- A documented mapping exists between InfoBlox source fields, UniLog event schema, and XSIAM XDM fields.
- Setup, operation, and troubleshooting are documented well enough for a customer engineer to deploy without project-team involvement.

---

## Future phases (summary only)

- **Phase 2 — Generic syslog input.** Broaden the input side to handle arbitrary syslog sources with configurable parsing.
- **Phase 3 — Pipeline engine + management UI.** Introduce the full source/route/pipeline/destination model and a management surface.
- **Phase 4 — Source and destination expansion.** Add input sources beyond syslog and output destinations beyond WEL.

Each future phase will get its own detailed plan when Phase 1 is complete.

---

## Working agreements

- **Public repository.** No secrets, credentials, customer data, or sensitive configuration are ever committed. Sample data must be sanitized.
- **Commit author.** All commits are authored as `sr@rookdynamics.com`.
- **Phased discipline.** PRs that jump ahead of the current phase are deferred until earlier phases stabilize. Open an issue to discuss before doing speculative work.
- **Decisions are tracked.** Significant architectural decisions are recorded in the repo (e.g., `docs/decisions/`) so the rationale survives the conversation that produced it.
