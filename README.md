# Crown

A browser-based **Web MIDI** routing editor for the **Escapement** MIDI
router — the crown being the part of a watch you use to set the movement.
Sibling of [Pulse Check](https://github.com/plentyofnames/pulse-check) and
[Reflex Hammer](https://github.com/plentyofnames/reflex-hammer).

▶︎ **Live app:** https://plentyofnames.github.io/crown/

Escapement is a DIY ESP32-S3 hardware MIDI router/aggregator — 4× DIN in,
8× DIN out, USB-MIDI device, BLE-MIDI and rtpMIDI — whose entire routing
matrix is programmed over SysEx from this page. Edit the rules, hit **Set**
and the box reroutes live; **Save** persists to flash.

## Features

- **Ordered rule matrix** (up to 64 rules), evaluated top-down with
  **first-match-wins per destination** — overlapping rules are deterministic
  and can never double-deliver a note.
- Per rule: source port → any set of destinations, a **message-class
  filter** (notes, CC, program change, channel pressure, pitch bend, sysex,
  clock, transport, SPP, system common, active sensing, reset), a **16-bit
  channel mask** for the voice classes, a **sysex manufacturer-ID filter**
  (keep the DAW's controller-surface spray away from vintage gear), and an
  optional **force-channel** remap.
- **Set + Verify** — writes the config and reads it back byte-exact.
- **Set is live but volatile**: a config that breaks the studio is cured by
  power cycle or **Revert**; only **Save → NVS** persists. The editor tracks
  the RAM≠NVS state.
- **Built-in protocol test suite** — one click runs the device through the
  HELLO/round-trip/error-path battery and restores the pre-test state.
- **JSON import/export** of rule tables; the reference studio table ships
  in the page.
- No build step, no dependencies — one static page.

## Requirements

- A **Web MIDI–capable browser**: Chrome, Edge, or Opera (Safari and
  Firefox do not implement Web MIDI). Grant the SysEx permission when
  prompted.
- An Escapement running firmware ≥ 0.3, connected over USB
  (port name **"Escapement USB"**). Config over DIN works too — replies
  return on the paired DIN out.

## Protocol

Everything speaks manufacturer ID `7D` (non-commercial) plus the signature
`"ESC"`:

```
F0 7D 45 53 43 <cmd> <data…> F7
```

| cmd | action |
|-----|--------|
| `01` | HELLO → firmware / blob-format / max-rules info |
| `10` | GET the active (RAM) config |
| `11` | SET — validate, apply live, don't persist |
| `12` | SAVE — commit RAM config to flash |
| `13` | REVERT — reload the persisted config |

Config payloads are a packed little-endian rule blob (16 bytes/rule) with
CRC-16/CCITT-FALSE appended, transmitted as 7-bit-clean nibble pairs. Every
SET/SAVE/REVERT is answered by exactly one ACK or NACK with a specific
error code; the blob format already reserves address space for a planned
second, trunk-linked Escapement unit (8-in/16-out as one device).

The firmware lives in a separate (currently private) repository.

## License

MIT
