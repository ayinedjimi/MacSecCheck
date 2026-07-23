<div align="center">

# 🍎 MacSecCheck

### macOS security posture auditor — fast, offline, driven by NIST baselines

**478 mSCP rules · 17 baselines (CIS, DISA STIG, NIST 800-53, CMMC…) · SHA‑256 signed reports**

[![macOS](https://img.shields.io/badge/macOS-Sonoma_14_|_Tahoe_26-000000?logo=apple&logoColor=white)](https://www.apple.com/macos/)
[![.NET 9](https://img.shields.io/badge/.NET-9.0-512BD4?logo=dotnet&logoColor=white)](https://dotnet.microsoft.com/)
[![Apple Silicon](https://img.shields.io/badge/Apple_Silicon_+_Intel-universal-0071e3)](#-installation)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![By Ayi NEDJIMI Consultants](https://img.shields.io/badge/By-Ayi%20NEDJIMI%20Consultants-C50F1F)](https://ayinedjimi-consultants.fr)

[🇫🇷 Français](README.md) · 🇬🇧 **English** · 🪟 Windows edition: [**WinCheckSec**](https://github.com/ayinedjimi/WinCheckSec)

</div>

---

## 🎯 Overview

**MacSecCheck** performs a deep audit of a Mac's security configuration (**macOS Sonoma 14** / **Tahoe 26**) and compares it against the official **NIST macOS Security Compliance Project (mSCP)** baselines — CIS, DISA STIG, NIST 800‑53, CMMC, CNSSI‑1253… It produces a score, prioritized findings and a **forensic JSON report signed with SHA‑256** for post‑incident review.

- 🔒 **100% local & offline** — no data ever leaves the machine.
- ⚡ **Fast** — collectors run in parallel.
- 📦 **Self‑contained** — a single native binary (embedded .NET runtime), nothing to install.
- 🧾 **Forensic report** — host context, per‑category scores, re‑verifiable **SHA‑256** hash.
- 🏛️ **NIST‑driven** — the 478 mSCP rules (check command + expected value + remediation + CIS/NIST/DISA mapping) are **embedded** in the binary.

```text
MacSecCheck — auditeur de securite macOS  v0.2.0
================================================
Baseline mSCP : cis_lvl1 (macOS 26) — 98 regles sur 5 sections, 478 regles indexees.

Hote     : macbook-pro (Darwin 25.0.0)
Elevation: root
Score    : 78/100

── Chiffrement / FileVault
   ✔ [Ok         ] Chiffrement FileVault du disque de demarrage : Actif
── mSCP · Operating System
   ✖ [High       ] Enable Gatekeeper : false   (expected string=true)
   ✔ [Ok         ] Ensure System Integrity Protection is Enabled : 1
```

---

## ✨ What is audited

| Area | Checks | Source |
|---|---|---|
| 🔐 **Encryption** | FileVault (status, in‑progress encryption) | `fdesetup` |
| 🛡️ **System integrity** | SIP (System Integrity Protection) | `csrutil` |
| 🚦 **App control** | Gatekeeper / notarization | `spctl` |
| 🧱 **Network** | Application firewall, stealth mode | `socketfilterfw` |
| 🦠 **Antimalware** | XProtect + XProtect Remediator | XProtect plist |
| 🌐 **Attack surface** | Remote login (SSH), screen sharing | `systemsetup`, `launchctl` |
| 🔄 **Maintenance** | Automatic updates & security patches | `defaults` |
| 🏛️ **mSCP compliance** | 478 rules / 17 baselines (CIS L1/L2, DISA STIG, 800‑53, CMMC…) | NIST mSCP |

Every finding carries a **severity**, an **observed vs expected** value, a **remediation** (`sudo …`) and a **reference** (CIS / NIST 800‑53 / DISA STIG + MITRE ATT&CK technique).

---

## 🏛️ mSCP baselines (NIST)

The YAML rules and baselines from the [**NIST macOS Security Compliance Project**](https://github.com/usnistgov/macos_security) (public domain) are **embedded** in the binary — no external tree required.

- **478 rules** indexed, **17 baselines** for macOS 26.
- `cis_lvl1` → 98 rules across 5 sections (Auditing, Operating System, Password Policy, System Settings, Supplemental).
- **One collector per section** runs each rule's `check` and compares the output to the expected value → compliant / gap (severity from **DISA STIG**).

```bash
./macseccheck --list-baselines            # list available baselines
./macseccheck --baseline cis_lvl2         # evaluate another baseline
./macseccheck --baseline disa_stig        # DISA STIG
./macseccheck --dump-rule os_sip_enable   # diagnostics: resolved check/expected/fix
./macseccheck --mscp /path/macos_security # use an external checkout (up-to-date data)
```

Available baselines: `cis_lvl1`, `cis_lvl2`, `disa_stig`, `800-53r5_high/moderate/low`, `cmmc_lvl1/2`, `cnssi-1253_high/moderate/low`, `cisv8`, `800-171`, `hicp_lp`, `nlmapgov_base/plus`, `all_rules`.

---

## 🚀 Installation

### Portable binary (recommended)
1. Download `macseccheck` (arm64 or x64) from the [**latest release**](https://github.com/ayinedjimi/MacSecCheck/releases/latest).
2. `chmod +x macseccheck`
3. `sudo ./macseccheck` — some checks require `root` (SSH, auditing, full SIP).

> Unsigned binary: on first launch **right‑click → Open**, or `xattr -d com.apple.quarantine macseccheck`.

### From source
```bash
git clone https://github.com/ayinedjimi/MacSecCheck.git
cd MacSecCheck
dotnet publish -c Release -r osx-arm64 --self-contained true -p:PublishSingleFile=true
# Intel: -r osx-x64
```

---

## 🧾 Forensic report

The JSON report (saved to the Desktop) contains: `SchemaVersion`, `Host` context, `Execution` block, `Summary` (score + grade + per‑severity counts), per‑category scores, timestamped `Modules` with each finding, and an **`Integrity`** block (re‑verifiable **SHA‑256** hash). Exit code: `0` if score ≥ 40, `2` otherwise (CI‑friendly).

---

## 🗺️ Roadmap

- [x] Cross‑platform engine + 7 native collectors
- [x] mSCP baseline integration (478 rules / 17 baselines)
- [ ] TCC collectors (privacy permissions), system/kext extensions, MDM profiles, Lockdown Mode, Secure Boot (`bputil`)
- [ ] PDF / HTML / SARIF exports
- [ ] **Avalonia** GUI (sharing the engine)
- [ ] **Signed + notarized** `.app` packaging, universal binary

Architecture details: [`docs/ARCHITECTURE.en.md`](docs/ARCHITECTURE.en.md) · Usage guide: [`docs/USAGE.en.md`](docs/USAGE.en.md).

---

## 🔒 Privacy & ethics

MacSecCheck is a **defensive** tool. It **reads** system state (read‑only commands) and **changes nothing**. No data is sent over the network. Use it only on systems you are authorized to audit.

---

## 👤 Author & services

Built by **Ayi NEDJIMI** — [**Ayi NEDJIMI Consultants**](https://ayinedjimi-consultants.fr), offensive security & AI expert.

📚 **Related resources**:
- [SME IT Security Audit: Complete Guide](https://ayinedjimi-consultants.fr/articles)
- [ISO 27001 Internal Audit: Method & Checklist](https://ayinedjimi-consultants.fr/iso-27001)
- [NIS 2 Compliance](https://ayinedjimi-consultants.fr/nis-2) · [Microsoft 365 Audit](https://ayinedjimi-consultants.fr/audit-microsoft-365)

💼 Need a professional security audit? [**Request a quote →**](https://ayinedjimi-consultants.fr/contact)

---

## 📄 License

Code under the **MIT** license — see [`LICENSE`](LICENSE).
mSCP data under the **NIST** license (public domain / U.S. Government work) — see [`mscp/LICENSE_mscp.md`](mscp/LICENSE_mscp.md).

<div align="center">
<sub>⭐ If MacSecCheck helps you, drop a star and join the <a href="https://github.com/ayinedjimi/MacSecCheck/discussions">Discussions</a>!</sub>
</div>
