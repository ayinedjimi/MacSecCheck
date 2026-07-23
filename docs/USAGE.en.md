# Usage guide — MacSecCheck

[🇫🇷 Version française](USAGE.fr.md)

## Quick start

```bash
# 1. Download the binary (arm64 for Apple Silicon, x64 for Intel)
#    from https://github.com/ayinedjimi/MacSecCheck/releases/latest
chmod +x macseccheck

# 2. Clear the Gatekeeper quarantine (unsigned binary)
xattr -d com.apple.quarantine macseccheck 2>/dev/null || true

# 3. Run (root recommended)
sudo ./macseccheck
```

## Command-line options

| Option | Description |
|---|---|
| *(none)* | Full audit + table + JSON report on the Desktop. |
| `--json <path>` | Write the JSON report to the given path. |
| `--quiet` | Do not print the table, only write JSON. |
| `--baseline <name>` | mSCP baseline to evaluate (default: `cis_lvl1`). |
| `--list-baselines` | List available baselines then exit. |
| `--dump-rule <id>` | Print a rule's resolution (check / expected / fix / refs). |
| `--os-version <major>` | Force the macOS major version (e.g. `14`, `26`). Auto‑detected otherwise. |
| `--mscp <dir>` | Use an external mSCP checkout instead of the embedded data. |

## Examples

```bash
sudo ./macseccheck                                    # standard audit (cis_lvl1)
./macseccheck --baseline cis_lvl2                      # CIS level 2
./macseccheck --baseline disa_stig --json /tmp/r.json  # DISA STIG, targeted output
./macseccheck --list-baselines                         # see all baselines
./macseccheck --dump-rule os_gatekeeper_enable         # diagnose a rule
```

## Privileges

- Without `sudo`, most checks work, but some will return partial values
  (SSH login, audit logs, full SIP state).
- For an exhaustive audit: run with `sudo`. Some reads (TCC permissions) also require
  **Full Disk Access** granted to the terminal (System Settings → Privacy & Security).

## Reading the score

- Each evaluable finding is weighted by severity; `Info`, `NotApplicable` and `Error` are excluded.
- Grade: A ≥ 90, B ≥ 75, C ≥ 60, D ≥ 40, F otherwise.
- Process exit code: `0` if score ≥ 40, `2` otherwise (useful in CI/CD).

## JSON report

Saved by default to `~/Desktop/MacSecCheck_<host>_<timestamp>.json`. Contains the host context,
per‑category scores, each timestamped finding (severity, observed, expected, remediation, reference),
and an `Integrity` block with the re‑verifiable **SHA‑256** hash of the report.

Verify integrity:
```bash
shasum -a 256 MacSecCheck_*.json
```
