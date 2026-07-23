<div align="center">

# 🍎 MacSecCheck

### Auditeur de posture de sécurité macOS — rapide, hors-ligne, piloté par les baselines NIST

**478 règles mSCP · 17 baselines (CIS, DISA STIG, NIST 800-53, CMMC…) · rapports signés SHA‑256**

[![macOS](https://img.shields.io/badge/macOS-Sonoma_14_|_Tahoe_26-000000?logo=apple&logoColor=white)](https://www.apple.com/macos/)
[![.NET 9](https://img.shields.io/badge/.NET-9.0-512BD4?logo=dotnet&logoColor=white)](https://dotnet.microsoft.com/)
[![Apple Silicon](https://img.shields.io/badge/Apple_Silicon_+_Intel-universal-0071e3)](#-installation)
[![License](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![By Ayi NEDJIMI Consultants](https://img.shields.io/badge/By-Ayi%20NEDJIMI%20Consultants-C50F1F)](https://ayinedjimi-consultants.fr)

🇫🇷 **Français** · [🇬🇧 English](README.en.md) · 🪟 Version Windows : [**WinCheckSec**](https://github.com/ayinedjimi/WinCheckSec)

</div>

---

## 🎯 En bref

**MacSecCheck** audite en profondeur la configuration de sécurité d'un Mac (**macOS Sonoma 14** / **Tahoe 26**) et la compare aux référentiels officiels du **NIST macOS Security Compliance Project (mSCP)** — CIS, DISA STIG, NIST 800‑53, CMMC, CNSSI‑1253… Il produit un score, des constats priorisés et un **rapport JSON forensique signé SHA‑256** pour une étude a posteriori.

- 🔒 **100 % local & hors‑ligne** — aucune donnée ne quitte le poste.
- ⚡ **Rapide** — collecteurs exécutés en parallèle.
- 📦 **Autonome** — un seul binaire natif (runtime .NET embarqué), rien à installer.
- 🧾 **Rapport forensique** — contexte hôte, scores par catégorie, empreinte **SHA‑256** re‑vérifiable.
- 🏛️ **Piloté par le NIST** — les 478 règles mSCP (commande de vérification + valeur attendue + remédiation + mapping CIS/NIST/DISA) sont **embarquées** dans le binaire.

```text
MacSecCheck — auditeur de securite macOS  v0.2.0
================================================
Baseline mSCP : cis_lvl1 (macOS 26) — 98 regles sur 5 sections, 478 regles indexees.

Hote     : macbook-pro (Darwin 25.0.0)
Elevation: root
Score    : 78/100

── Chiffrement / FileVault
   ✔ [Ok         ] Chiffrement FileVault du disque de demarrage : Actif
── Integrite systeme / SIP
   ✔ [Ok         ] System Integrity Protection (SIP) : Active
── mSCP · Operating System
   ✖ [High       ] Enable Gatekeeper : false   (attendu string=true)
   ✔ [Ok         ] Ensure System Integrity Protection is Enabled : 1
```

---

## ✨ Ce qui est audité

| Domaine | Contrôles | Source |
|---|---|---|
| 🔐 **Chiffrement** | FileVault (état, chiffrement en cours) | `fdesetup` |
| 🛡️ **Intégrité système** | SIP (System Integrity Protection) | `csrutil` |
| 🚦 **Contrôle applicatif** | Gatekeeper / notarisation | `spctl` |
| 🧱 **Réseau** | Pare‑feu applicatif, mode furtif | `socketfilterfw` |
| 🦠 **Antimalware** | XProtect + XProtect Remediator | plist XProtect |
| 🌐 **Surface d'attaque** | Connexion distante (SSH), partage d'écran | `systemsetup`, `launchctl` |
| 🔄 **Maintenance** | Mises à jour automatiques & correctifs de sécurité | `defaults` |
| 🏛️ **Conformité mSCP** | 478 règles / 17 baselines (CIS L1/L2, DISA STIG, 800‑53, CMMC…) | NIST mSCP |

Chaque constat porte une **gravité**, une **valeur observée vs attendue**, une **remédiation** (`sudo …`) et une **référence** (CIS / NIST 800‑53 / DISA STIG + technique MITRE ATT&CK).

---

## 🏛️ Baselines mSCP (NIST)

Les règles et baselines YAML du [**NIST macOS Security Compliance Project**](https://github.com/usnistgov/macos_security) (domaine public) sont **embarquées** dans le binaire — aucune arborescence externe requise.

- **478 règles** indexées, **17 baselines** macOS 26.
- `cis_lvl1` → 98 règles / 5 sections (Auditing, Operating System, Password Policy, System Settings, Supplemental).
- Un **collecteur par section** exécute le `check` de chaque règle et compare la sortie à la valeur attendue → conforme / écart (gravité issue de la sévérité **DISA STIG**).

```bash
./macseccheck --list-baselines            # liste les baselines disponibles
./macseccheck --baseline cis_lvl2         # évalue une autre baseline
./macseccheck --baseline disa_stig        # STIG DISA
./macseccheck --dump-rule os_sip_enable   # diagnostic : check/attendu/fix résolus
./macseccheck --mscp /chemin/macos_security   # utilise un checkout externe (données à jour)
```

Baselines disponibles : `cis_lvl1`, `cis_lvl2`, `disa_stig`, `800-53r5_high/moderate/low`, `cmmc_lvl1/2`, `cnssi-1253_high/moderate/low`, `cisv8`, `800-171`, `hicp_lp`, `nlmapgov_base/plus`, `all_rules`.

---

## 🚀 Installation

### Binaire portable (recommandé)
1. Téléchargez `macseccheck` (arm64 ou x64) depuis la [**dernière release**](https://github.com/ayinedjimi/MacSecCheck/releases/latest).
2. `chmod +x macseccheck`
3. `sudo ./macseccheck` — certains contrôles exigent `root` (SSH, audit, SIP complet).

> Binaire non notarisé : au premier lancement, faites **clic‑droit → Ouvrir**, ou `xattr -d com.apple.quarantine macseccheck`.

### Depuis les sources
```bash
git clone https://github.com/ayinedjimi/MacSecCheck.git
cd MacSecCheck
dotnet publish -c Release -r osx-arm64 --self-contained true -p:PublishSingleFile=true
# Intel : -r osx-x64
```

---

## 🧾 Rapport forensique

Le rapport JSON (sauvegardé sur le Bureau) contient : `SchemaVersion`, contexte `Host`, bloc `Execution`, `Summary` (score + grade + décompte par gravité), scores par catégorie, `Modules` horodatés avec chaque constat, et un bloc **`Integrity`** (empreinte **SHA‑256** re‑vérifiable). Code retour : `0` si score ≥ 40, `2` sinon (exploitable en CI).

---

## 🗺️ Feuille de route

- [x] Moteur multiplateforme + 7 collecteurs natifs
- [x] Intégration des baselines mSCP (478 règles / 17 baselines)
- [ ] Collecteurs TCC (permissions vie privée), extensions système/kext, profils MDM, Lockdown Mode, Secure Boot (`bputil`)
- [ ] Exports PDF / HTML / SARIF
- [ ] UI **Avalonia** (interface graphique partageant le moteur)
- [ ] Packaging `.app` **signé + notarisé**, binaire universel

Détails d'architecture : [`docs/ARCHITECTURE.fr.md`](docs/ARCHITECTURE.fr.md) · Guide d'utilisation : [`docs/USAGE.fr.md`](docs/USAGE.fr.md).

---

## 🔒 Confidentialité & éthique

MacSecCheck est un outil **défensif**. Il **lit** l'état du système (commandes en lecture seule) et **ne modifie rien**. Aucune donnée n'est transmise sur le réseau. À utiliser uniquement sur des systèmes que vous êtes autorisé à auditer.

---

## 👤 Auteur & services

Développé par **Ayi NEDJIMI** — [**Ayi NEDJIMI Consultants**](https://ayinedjimi-consultants.fr), expert en cybersécurité offensive & IA.

📚 **Ressources en lien** :
- [Audit de Sécurité Informatique PME : Guide Complet](https://ayinedjimi-consultants.fr/articles)
- [Audit Interne ISO 27001 : Méthode & Checklist](https://ayinedjimi-consultants.fr/iso-27001)
- [Conformité NIS 2](https://ayinedjimi-consultants.fr/nis-2) · [Audit Microsoft 365](https://ayinedjimi-consultants.fr/audit-microsoft-365)

💼 Besoin d'un audit de sécurité professionnel ? [**Demandez un devis →**](https://ayinedjimi-consultants.fr/contact)

---

## 📄 Licence

Code sous licence **MIT** — voir [`LICENSE`](LICENSE).
Données mSCP sous licence **NIST** (domaine public / œuvre du gouvernement américain) — voir [`mscp/LICENSE_mscp.md`](mscp/LICENSE_mscp.md).

<div align="center">
<sub>⭐ Si MacSecCheck vous est utile, laissez une étoile et rejoignez les <a href="https://github.com/ayinedjimi/MacSecCheck/discussions">Discussions</a> !</sub>
</div>
