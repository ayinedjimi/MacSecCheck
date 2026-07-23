# Architecture — MacSecCheck

[🇫🇷 Version française](ARCHITECTURE.fr.md)

MacSecCheck follows the same pattern as the Windows edition (WinCheckSec) but decoupled from any
Windows code. The engine is cross‑platform C#; collectors call macOS system tools.

## Overview

```
┌──────────────────────────────────────────────────────────┐
│  Program.cs (CLI)                                         │
│  parse options → build collectors → scan                 │
└───────────────┬──────────────────────────────────────────┘
                │
        ┌───────▼────────┐        ┌───────────────────────────┐
        │  ScanEngine    │        │  MscpDataLoader           │
        │  parallel +    │◄───────┤  loads rules + baseline   │
        │  scoring       │        │  (embedded or --mscp)     │
        └───────┬────────┘        └───────────────────────────┘
                │
   ┌────────────┼─────────────────────────────┐
   ▼            ▼                              ▼
Native        MscpSectionCollector (1/section)   ...
collectors    runs each mSCP check via bash
   │            │
   └──────┬─────┘
          ▼
   JsonReportBuilder → JSON report + SHA-256 hash
```

## Components (`Core/`)

| File | Role |
|---|---|
| `IMacCollector` | Collector contract (`Name`, `Category`, `CollectAsync`). |
| `MacCollectorBase` | Shared base: measures duration, catches exceptions, short‑circuits off‑macOS (→ `NotApplicable`). |
| `ProcessRunner` | Safe tool execution **without a shell** (no injection), plus `RunShellAsync` (`/bin/bash -c`) for mSCP checks, plus `MacOsMajorAsync` (`sw_vers`). |
| `ScanEngine` | Runs all collectors in parallel and computes scores (Info/NA/Error excluded from the denominator). |
| `JsonReportBuilder` | Forensic JSON report + `Integrity` block (SHA‑256 of the body). |
| `Models` | `Severity`, `Finding`, `CollectorReport`. |

## Native collectors (`Collectors/`)

Each collector queries one or more macOS tools and produces `Finding`s:
FileVault (`fdesetup`), Gatekeeper (`spctl`), SIP (`csrutil`), firewall (`socketfilterfw`),
XProtect (plist), SSH/screen sharing (`systemsetup`, `launchctl`), updates (`defaults`).

## mSCP integration (`mscp/` + `MscpSectionCollector`)

- `MscpRule`: parses a rule YAML and **resolves `enforcement_info` by macOS version**:
  version‑specific block (`26.0`) → canonical macOS‑level block → fallback. Extracts `check.shell`,
  typed `result` (`integer`/`string`/`boolean`/`float`), `fix.shell`, CIS/NIST/DISA references + STIG severity.
- `MscpDataLoader`: indexes rules by `id`, resolves a baseline (`profile → sections → rules`).
  **Embedded** source (resources) or **external** (`--mscp <dir>`).
- `MscpSectionCollector`: one collector per section; runs each rule's `check` via `bash` and compares
  the output to the expected value → compliant (`Ok`) / gap (severity = DISA STIG severity, else `Medium`).
  A rule with no automatable check → `Info` ("manual verification").

## Adding a native collector

1. Create a class inheriting `MacCollectorBase` in `Collectors/`.
2. Implement `CollectCoreAsync`: call `Run("/usr/bin/tool", ct, "arg")`, interpret, add `Finding`s.
3. Register it in the `collectors` list in `Program.cs`.

## Portability

The project targets `net9.0` (not `-macos`) and compiles on Windows/Linux/macOS. Off macOS,
`MacCollectorBase` returns `NotApplicable` — handy to compile/validate logic in Windows CI.
Checks only produce real results on macOS.
