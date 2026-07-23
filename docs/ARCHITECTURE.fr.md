# Architecture — MacSecCheck

[🇬🇧 English version](ARCHITECTURE.en.md)

MacSecCheck reprend le patron de la version Windows (WinCheckSec) mais découplé de tout code Windows.
Le moteur est du C# multiplateforme ; les collecteurs appellent les outils système macOS.

## Vue d'ensemble

```
┌──────────────────────────────────────────────────────────┐
│  Program.cs (CLI)                                         │
│  parse les options → construit les collecteurs → scanne  │
└───────────────┬──────────────────────────────────────────┘
                │
        ┌───────▼────────┐        ┌───────────────────────────┐
        │  ScanEngine    │        │  MscpDataLoader           │
        │  parallèle +   │◄───────┤  charge règles + baseline │
        │  scoring       │        │  (embarqué ou --mscp)     │
        └───────┬────────┘        └───────────────────────────┘
                │
   ┌────────────┼─────────────────────────────┐
   ▼            ▼                              ▼
Collecteurs   MscpSectionCollector (1/section)   ...
natifs        exécute chaque check mSCP via bash
   │            │
   └──────┬─────┘
          ▼
   JsonReportBuilder → rapport JSON + empreinte SHA-256
```

## Composants (`Core/`)

| Fichier | Rôle |
|---|---|
| `IMacCollector` | Contrat d'un collecteur (`Name`, `Category`, `CollectAsync`). |
| `MacCollectorBase` | Base commune : mesure la durée, capture les exceptions, court-circuite hors‑macOS (→ `NotApplicable`). |
| `ProcessRunner` | Exécution sûre d'outils système **sans shell** (pas d'injection), + `RunShellAsync` (`/bin/bash -c`) pour les checks mSCP, + `MacOsMajorAsync` (`sw_vers`). |
| `ScanEngine` | Exécute tous les collecteurs en parallèle et calcule les scores (Info/NA/Error exclus du dénominateur). |
| `JsonReportBuilder` | Rapport JSON forensique + bloc `Integrity` (SHA‑256 du corps). |
| `Models` | `Severity`, `Finding`, `CollectorReport`. |

## Collecteurs natifs (`Collectors/`)

Chaque collecteur interroge un ou plusieurs outils macOS et produit des `Finding` :
FileVault (`fdesetup`), Gatekeeper (`spctl`), SIP (`csrutil`), pare‑feu (`socketfilterfw`),
XProtect (plist), partages SSH/écran (`systemsetup`, `launchctl`), mises à jour (`defaults`).

## Intégration mSCP (`mscp/` + `MscpSectionCollector`)

- `MscpRule` : parse un YAML de règle et **résout `enforcement_info` selon la version macOS** :
  bloc spécifique (`26.0`) → bloc canonique de niveau macOS → repli. Extrait `check.shell`,
  `result` typé (`integer`/`string`/`boolean`/`float`), `fix.shell`, références CIS/NIST/DISA + sévérité STIG.
- `MscpDataLoader` : indexe les règles par `id`, résout une baseline (`profile → sections → rules`).
  Source **embarquée** (ressources) ou **externe** (`--mscp <dir>`).
- `MscpSectionCollector` : un collecteur par section ; exécute le `check` de chaque règle via `bash`
  et compare la sortie à la valeur attendue → conforme (`Ok`) / écart (gravité = sévérité DISA STIG,
  sinon `Medium`). Une règle sans check automatisable → `Info` (« vérification manuelle »).

## Ajouter un collecteur natif

1. Créer une classe qui hérite de `MacCollectorBase` dans `Collectors/`.
2. Implémenter `CollectCoreAsync` : appeler `Run("/usr/bin/outil", ct, "arg")`, interpréter, ajouter des `Finding`.
3. L'enregistrer dans la liste `collectors` de `Program.cs`.

## Portabilité

Le projet cible `net9.0` (pas `-macos`) et compile sur Windows/Linux/macOS. Hors macOS,
`MacCollectorBase` renvoie `NotApplicable` — utile pour compiler/valider la logique en CI Windows.
Les checks ne produisent des résultats réels que sur macOS.
