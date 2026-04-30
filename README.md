# UniLog

**A log ingestion and translation engine for Windows environments.**

UniLog accepts logs from arbitrary sources and writes them into the Windows Event Log as native-looking events. Once in WEL, logs flow through standard Windows tooling and on to whatever SIEM or observability platform you already use.

The goal: stop building one-off shippers for every log source touching a Windows-centric pipeline. Normalize at the edge, write once, consume natively.

---

## Vision

Modern security and observability stacks on Windows are built around the Event Log. Getting non-Windows log sources into that pipeline today is a mess of brittle scripts, format mismatches, and lossy translations.

UniLog closes that gap. It takes structured input from arbitrary sources, emits properly-formed Windows events, and lets downstream collectors treat translated logs the same way they treat native ones — without impersonating Microsoft providers or forging identity.

The long-term direction is a pluggable pipeline supporting many input sources, configurable transforms, and multiple output destinations. UniLog is being built in deliberate phases toward that vision; each phase ships a working, useful tool rather than a half-built version of the next one.

---

## Phase 1 — InfoBlox DNS/DHCP → WEL *(current)*

A focused translator for a single high-value path: InfoBlox syslog, ingested via Cribl, translated by UniLog, written to the Windows Event Log, and pulled by the Cortex XDR Collector into XSIAM.

Phase 1 proves the core translation engine against a real customer pipeline before generalizing.

**Status:** 🚧 In development. Many implementation decisions are still open and details may change.

---

## Contributing

Contributions are welcome — please open an issue to discuss before starting work.

## License

MIT
