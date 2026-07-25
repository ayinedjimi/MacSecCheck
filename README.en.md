<div align="center">

# 🍎 MacSecCheck

### macOS security posture audit application — driven by NIST mSCP baselines

**Native GUI (Avalonia) · 478 mSCP rules · 17 baselines (CIS, DISA STIG, NIST 800‑53, CMMC…) · SHA‑256 signed reports**

[![Latest release](https://img.shields.io/github/v/release/ayinedjimi/MacSecCheck?label=latest%20release&color=0071e3)](https://github.com/ayinedjimi/MacSecCheck/releases/latest)
[![macOS](https://img.shields.io/badge/macOS-12%2B_Monterey_%E2%86%92_Tahoe_26-000000?logo=apple&logoColor=white)](https://www.apple.com/macos/)
[![.NET 9](https://img.shields.io/badge/.NET-9.0-512BD4?logo=dotnet&logoColor=white)](https://dotnet.microsoft.com/)
[![Apple Silicon](https://img.shields.io/badge/Apple_Silicon_+_Intel-universal-0071e3)](#-download--installation)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![By Ayi NEDJIMI Consultants](https://img.shields.io/badge/By-Ayi%20NEDJIMI%20Consultants-C50F1F)](https://ayinedjimi-consultants.fr)

[🇫🇷 Français](README.md) · 🇬🇧 **English** · 🪟 Windows edition: [**WinCheckSec**](https://github.com/ayinedjimi/WinCheckSec)

</div>

---

## 🎯 Overview

**MacSecCheck** is a **native macOS application** with a **graphical interface (Avalonia)** that performs a deep audit of a Mac's security configuration and compares it against the official **NIST macOS Security Compliance Project (mSCP)** baselines — CIS, DISA STIG, NIST 800‑53, CMMC, CNSSI‑1253… It produces a score, prioritized findings and a **forensic JSON report signed with SHA‑256** for post‑incident review. It is the natural macOS companion to [**WinCheckSec**](https://github.com/ayinedjimi/WinCheckSec), its Windows counterpart.

- 🖥️ **Graphical application** — no terminal required: everything is driven from a native macOS window.
- 📦 **Self‑contained** — .NET 9, Avalonia and the audit engine (478 mSCP rules) are all **embedded** in the `.app`; nothing to install.
- 🔒 **100% local & offline** — no data ever leaves the machine.
- ⚡ **Fast** — collectors run in parallel.
- 🧾 **Forensic report** — host context, per‑category scores, re‑verifiable **SHA‑256** hash, exportable to JSON.
- 🏛️ **NIST‑driven** — the 478 mSCP rules (check command + expected value + remediation + CIS/NIST/DISA mapping + MITRE ATT&CK technique) are **embedded**.
- 🖥️ **Universal** — macOS 12 (Monterey) and later, Intel & Apple Silicon.

---

## ⬇️ Download & installation

[![Download latest release](https://img.shields.io/badge/⬇️_Download-latest_release-0071e3?style=for-the-badge&logo=apple)](https://github.com/ayinedjimi/MacSecCheck/releases/latest)

1. **Download** the archive: [**MacSecCheck-macOS-app.zip**](https://github.com/ayinedjimi/MacSecCheck/releases/download/v0.3/MacSecCheck-macOS-app.zip) (≈ 74 MB) from the [**latest release**](https://github.com/ayinedjimi/MacSecCheck/releases/latest).
2. **Double-click** the zip to unpack it.
3. **Drag `MacSecCheck.app`** into your **Applications** folder.
4. On first launch, **right‑click → Open** (the app isn't Developer ID signed / notarized yet — ad‑hoc signature).
5. Click **"Analyze"** — that's it.

> 📦 **Self‑contained application**: the **.NET** runtime, the **Avalonia** framework and the audit engine (**478 mSCP rules**) are **all embedded** in `MacSecCheck.app`. No dependencies to install, no network access required. Compatible with **macOS 12 and later**, both **Intel and Apple Silicon**.

### From source
```bash
git clone https://github.com/ayinedjimi/MacSecCheck.git
cd MacSecCheck
dotnet publish -c Release -r osx-arm64 --self-contained true -p:PublishSingleFile=true
# Intel: -r osx-x64
```

---

## 🖼️ The application

<div align="center">
<img src="docs/screenshots/macseccheck.png" alt="MacSecCheck — native macOS application" width="840">
<br><em>MacSecCheck v0.3 — native macOS application (real screenshot)</em>
</div>

MacSecCheck presents itself as a true macOS application, organized around a navigation **sidebar**:

- **Overview** plus one entry per **category** of checks, each displaying a **colored severity badge** (Critical / High / Medium / Low / OK) reflecting the state of its findings.
- **Home screen** with the application's title and author.
- **Baseline selector**: **"All"** is selected by default (`all_rules`, the full **478 rules**), with `cis_lvl1`, `cis_lvl2`, `disa_stig`, `800-53r5_high` and the other mSCP baselines also available.
- **"Analyze"** button to launch the audit.
- **Overview view**: overall score **out of 100** with a colored grade (A → F), plus summary cards per severity (Critical / High / Medium / Low / OK).
- **Category view**: list of **expandable** findings, each with its detail, its **remediation**, its **reference** (CIS / NIST 800‑53 / DISA STIG) and the associated **MITRE ATT&CK technique**.
- **Export**: generates a **SHA‑256 signed forensic JSON report**, for archiving or further analysis.

### Real-world scan preview

```
Baseline : all_rules — 333 rules across 10 sections
Score    : 47/100  (grade D)
⛔ SIP .................. Disabled    (Critical)
✖  FileVault ........... Inactive    (High)
✔  Gatekeeper .......... Active      (OK)
✔  XProtect ............ v5352       (OK)
▲  Application firewall  Inactive    (Medium)
▲  SSH / Screen sharing  Enabled     (Medium)
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

The YAML rules and baselines from the [**NIST macOS Security Compliance Project**](https://github.com/usnistgov/macos_security) (data published under the **NIST** license, public domain / U.S. Government work) are **embedded** in the application — no external tree required.

- **478 rules** indexed, **17 baselines** for macOS 26.
- The **"All" (`all_rules`)** baseline is selected by default in the interface and covers all 478 rules.
- `cis_lvl1` → 98 rules across 5 sections (Auditing, Operating System, Password Policy, System Settings, Supplemental).
- **One collector per section** runs each rule's `check` and compares the output to the expected value → compliant / gap (severity from **DISA STIG**).

Available baselines: `cis_lvl1`, `cis_lvl2`, `disa_stig`, `800-53r5_high/moderate/low`, `cmmc_lvl1/2`, `cnssi-1253_high/moderate/low`, `cisv8`, `800-171`, `hicp_lp`, `nlmapgov_base/plus`, `all_rules`.

The CLI remains available for scripted / CI use:

```bash
./macseccheck --list-baselines            # list available baselines
./macseccheck --baseline cis_lvl2         # evaluate another baseline
./macseccheck --baseline disa_stig        # DISA STIG
./macseccheck --dump-rule os_sip_enable   # diagnostics: resolved check/expected/fix
./macseccheck --mscp /path/macos_security # use an external checkout (up-to-date data)
```

---

## 🧾 Forensic report

The JSON report exported from the application contains: `SchemaVersion`, `Host` context, `Execution` block, `Summary` (score + grade + per‑severity counts), per‑category scores, timestamped `Modules` with each finding, and an **`Integrity`** block (re‑verifiable **SHA‑256** hash). CLI exit code: `0` if score ≥ 40, `2` otherwise (CI‑friendly).

---

## 🗺️ Roadmap

- [x] Cross‑platform engine + 7 native collectors
- [x] mSCP baseline integration (478 rules / 17 baselines)
- [x] **Graphical interface** (Avalonia)
- [x] **Self‑contained `.app` packaging** (.NET + Avalonia + engine embedded)
- [ ] TCC collectors (privacy permissions), system/kext extensions, MDM profiles, Lockdown Mode, Secure Boot (`bputil`)
- [ ] PDF / HTML / SARIF exports
- [ ] **Developer ID signed + notarized** packaging, official universal binary

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
