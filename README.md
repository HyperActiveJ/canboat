# CANboat — proprietary-PGN decode branch

This is a fork of **[canboat/canboat](https://github.com/canboat/canboat)**.

Branch `Decode-In-Progress-HyperActiveJ` carries structural decodes for manufacturer
proprietary PGNs that upstream does not define yet. Branch `master` mirrors
`canboat/canboat` byte for byte and holds nothing of its own.

Everything else about CANboat — what it is, how to build it, how to use `analyzer`
and the gateway drivers, the PGN documentation — lives upstream:

- Source and build instructions: <https://github.com/canboat/canboat>
- PGN documentation: <https://canboat.github.io/canboat>
- Wiki: <https://github.com/canboat/canboat/wiki>

## What this branch adds

**16 proprietary PGN variant definitions, 185 fields**, on top of upstream, plus the
lookups and generated artifacts that go with them.

| Manufacturer | PGNs | Variants | Fields |
| --- | --- | --- | --- |
| Garmin | 126720 | 11 | 122 |
| Mercury Marine | 130816, 130817, 130821, 130830 | 4 | 49 |
| Navico | 130822 | 1 | 14 |

**Garmin, PGN 126720.** Eleven sub-protocols under the fast-packet proprietary
range are given their own variants and message-id lookup values: GHC self-status,
node-status and config-table beacons, GHC config key/value entries, a device-presence
master advertisement, a display version/identity handshake, a button-interface idle
beacon, GLM lighting control, GPSMAP 86xx device-health telemetry, and two GPSMAP
data-sync record layouts.

**Mercury Marine.** `130816 Alarm` (a fixed header with a fault-record count, followed
by that many 12-byte fault records), `130817 Vessel Configuration`, `130821 Pop-Up
Notification`, and `130830 Single Lever Mode` with a `MERCURY_SINGLE_LEVER_MODE`
lookup covering single-lever, throttle-only, docking and the three SmartCraft
no-comms states.

**Navico.** `130822 Alert`, the sub-type-2 alert record, alongside upstream's existing
130822 UDB variants.

**Field-level corrections and lookup extensions**

- `VICTRON_VREG` gains registers `0x0100` Product Id (24-bit) and `0x0102` Firmware
  Version (32-bit).
- `127750 Converter Status` gets its 1500 ms transmission interval and is marked
  complete.
- `126720 Garmin: Autopilot Heading to Steer` is an absolute heading on the full
  circle (0 .. 2 pi), not a signed offset; the range minimum is corrected accordingly.

**Regression test.** `analyzer/tests` gains `test26`, which runs one representative
frame per proprietary variant this fork works with — 21 frames across Mercury, Navico,
Fusion and Victron — through `analyzer` and diffs the decode against a checked-in
expectation.

**Generated output is regenerated on the branch**, so the definitions are live for
every consumer: `docs/canboat.json`, `docs/canboat.xml`, `docs/canboat.html`,
`analyzer/pgn-generated-data.h`, `analyzer/lookup-generated-data.h` and the
`canboat-core` Rust schema.

### What that looks like

A Garmin GPSMAP device-health frame on PGN 126720. Upstream reports the fallback
variant and leaves the payload as bytes:

```
126720 0x1EF00: Manufacturer Proprietary fast-packet addressed:
  Manufacturer Code = Garmin; Industry Code = Marine Industry;
  Data = 17 00 04 04 BF A0 1B 41 5E 14 7F 41 4C 67 95 41 ...
```

On this branch the same frame decodes structurally:

```
126720 Garmin: GPSMAP 86xx Device Health Telemetry:
  Manufacturer Code = Garmin; Industry Code = Marine Industry;
  Sub-protocol ID = GPSMAP device-health transport; Wrapper Byte 1 = 4;
  Wrapper Byte 2 = 4; Scalar 1 = 9.72674; Scalar 2 = 15.9425; ...
```

Where a payload's field layout is known but a value enumeration is not, the field is
decoded and left unlabelled rather than guessed at.

## Relation to upstream

Changes meant for `canboat/canboat` are offered there as focused pull requests, one
topic at a time, against upstream's own branch.

**This README describes the fork and is never part of such a pull request.**

## License

CANboat is licensed under the Apache License, Version 2.0. See `LICENSE`.
