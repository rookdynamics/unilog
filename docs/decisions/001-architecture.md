# ADR-001: Architecture & Technology Choices

**Status:** Accepted  
**Date:** 2026-04-30

## Context

UniLog Phase 1 translates InfoBlox DNS/DHCP syslog into Windows Event Log entries consumed by the Cortex XDR Collector. We need to decide on language, runtime, input/output mechanisms, project structure, and internal architecture.

## Decisions

### Language & Runtime

**C# on .NET 8 (LTS).** Windows Event Log APIs are native to the framework. Single-file self-contained publish gives a clean deployment story. `Microsoft.Extensions.Hosting.WindowsServices` provides built-in Windows Service hosting.

Minimum target: Windows Server 2019.

### Input

- **Primary:** TCP/UDP syslog listener. Ports configurable per ingest source. Multiple simultaneous sources supported.
- **Secondary:** HTTP/webhook endpoint for structured JSON delivery.
- Cribl forwards raw (unparsed) syslog — UniLog handles all parsing.

### Output

- **Primary:** Windows Event Collector (WEC) via WS-Management protocol. Colocated on the same host as the Cortex XDR Collector. Events must be 100% indistinguishable from native WEL entries.
- **Secondary:** Plain log file fallback.
- **Future (post-Phase 1):** Direct Windows Event Log API (ETW manifest-based provider).

### WEL Channel Layout

- **Default (single channel):** `UniLog-InfoBlox/Operational`
- **Optional (split mode):** `UniLog-InfoBlox/DNS` and `UniLog-InfoBlox/DHCP`
- Configurable via TOML. UniLog never impersonates Microsoft providers.

### Internal Architecture

Single-process pipeline using `System.Threading.Channels`:

```
[UDP Listener]──┐
[TCP Listener]──┼──▶ Channel<RawMessage> ──▶ Parser/Router ──▶ Channel<WelEvent> ──▶ Writer (batched)
[HTTP Listener]─┘    (bounded, backpressure)                    (bounded)              ├── WEC transport
                                                                                       └── File fallback
```

Two internal channels separate raw ingestion from parsed events:
- Adding new input formats (Phase 2) doesn't touch the writer.
- Swapping the writer (WEC → WEL) doesn't touch the parser.
- Bounded channels provide backpressure — if the writer can't keep up, the parser slows, which slows the listeners. Clean degradation without silent event loss.
- Writer batches events (count or timer threshold) for WEC performance.

### Project Structure

```
src/UniLog.Service/     — Windows Service host, WEC transport, file writer
src/UniLog.Core/        — Parser, translator, channel pipeline (platform-independent, testable)
tests/UniLog.Core.Tests/ — Unit tests for parsing and translation logic
```

Core is separated from Service so parsing/translation logic can be unit tested without Windows dependencies.

### Configuration

TOML format. Per-source port configuration, channel mode selection, output targets.

### Test Strategy

[LogSimv2](https://github.com/phreezone/LogSimv2) provides verified InfoBlox syslog simulation. Its `infoblox_dns.py` module generates production-accurate DNS, DHCP, audit, RPZ, and threat events. Will be used as the primary test data source if compatible with UniLog's listener.

## Consequences

- Single deployable binary (Windows Service) — simple operations story.
- Channel-based pipeline is easy to reason about and test in isolation.
- Core/Service split means most logic is testable on any platform (CI on Linux).
- WEC-first output defers the complexity of ETW manifest registration to a later phase.
