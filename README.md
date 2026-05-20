# SHIFT

**S**erial **H**ost **I**nterface **F**lexibility **T**oolkit

SHIFT is a serial-console proxy toolkit for embedding automation, parsing, and telemetry into a transparent UART bridge.

The active runtime target is **CircuitPython** on microcontrollers with sufficient memory (e.g. *RP2350*, *ESP32-S3*).

## Goals

- Keep serial forwarding fast and asynchronous.
- Preserve transparent console behavior by default.
- Parse key terminal control families (`BEL`, `CSI`, `SGR`, `OSC`, `DCS`).
- Drive a state machine from declarative rules in `rules-states.json`.
- Capture structured fields from console logs with regex groups.
- Support a telemetry side channel over DCS-wrapped CBOR.

## Glossary

- HOST side: UART connected to the host system's console/UART port (host = telecommunication equipment or server).
- TERM side: USB CDC exposed to operator's workstation terminal/console (putty, MobaXterm, etc.).
- Proxy behavior: forwards traffic both ways, parses terminal control sequences, runs a config-driven state machine, and keeps rolling history.

---

## Runtime Architecture

`circuitpython/runtime.py` runs asynchronous tasks for:

1. Host UART -> terminal CDC forwarding and parsing.
2. Terminal CDC -> host UART forwarding and key-mode handling.
3. LED heartbeat and activity indications.
4. Terminal overlay/status rendering and probe handling.
5. Optional stats telemetry.

### Data Path (Pi -> USB)

1. `UART.readinto()` into preallocated buffer.
2. DCS-CBOR stream parser checks for configured telemetry magic.
3. Matching telemetry frames are decoded and sent to `on_telemetry(frame)` and not forwarded to USB.
4. Non-telemetry bytes are appended to circular history ring.
5. `term_parser.UL.feed()` parses `BEL/CSI/SGR/OSC/DCS`.
6. Chunk decoded to text and line-buffered for SHIFT rule evaluation.
7. Passthrough bytes forwarded to USB CDC.

---

## Licence

This project is licensed under the MIT Licence. See [LICENSE](LICENSE).

Copyright (c) 2026 Billie Badin, SIGORYX Engineering
