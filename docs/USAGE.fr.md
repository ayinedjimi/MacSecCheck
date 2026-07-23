# Guide d'utilisation — MacSecCheck

[🇬🇧 English version](USAGE.en.md)

## Installation rapide

```bash
# 1. Télécharger le binaire (arm64 pour Apple Silicon, x64 pour Intel)
#    depuis https://github.com/ayinedjimi/MacSecCheck/releases/latest
chmod +x macseccheck

# 2. Lever la quarantaine Gatekeeper (binaire non notarisé)
xattr -d com.apple.quarantine macseccheck 2>/dev/null || true

# 3. Lancer (root recommandé)
sudo ./macseccheck
```

## Options de ligne de commande

| Option | Description |
|---|---|
| *(aucune)* | Audit complet + tableau + rapport JSON sur le Bureau. |
| `--json <chemin>` | Écrit le rapport JSON à l'emplacement indiqué. |
| `--quiet` | N'affiche pas le tableau, écrit seulement le JSON. |
| `--baseline <nom>` | Baseline mSCP à évaluer (défaut : `cis_lvl1`). |
| `--list-baselines` | Liste les baselines disponibles puis quitte. |
| `--dump-rule <id>` | Affiche la résolution d'une règle (check / attendu / fix / réfs). |
| `--os-version <major>` | Force la version macOS majeure (ex. `14`, `26`). Auto‑détectée sinon. |
| `--mscp <dir>` | Utilise un checkout externe du dépôt mSCP au lieu des données embarquées. |

## Exemples

```bash
sudo ./macseccheck                                   # audit standard (cis_lvl1)
./macseccheck --baseline cis_lvl2                     # CIS niveau 2
./macseccheck --baseline disa_stig --json /tmp/r.json # STIG DISA, sortie ciblée
./macseccheck --list-baselines                        # voir toutes les baselines
./macseccheck --dump-rule os_gatekeeper_enable        # diagnostiquer une règle
```

## Privilèges

- Sans `sudo`, la plupart des checks fonctionnent, mais certains renverront des valeurs partielles
  (connexion SSH, journaux d'audit, état SIP complet).
- Pour un audit exhaustif : exécuter avec `sudo`. Certaines lectures (permissions TCC) exigeront
  aussi l'**Accès complet au disque** accordé au terminal (Réglages Système → Confidentialité et sécurité).

## Interprétation du score

- Chaque constat évaluable est pondéré par sa gravité ; `Info`, `NotApplicable` et `Error` sont exclus.
- Grade : A ≥ 90, B ≥ 75, C ≥ 60, D ≥ 40, F sinon.
- Code retour du processus : `0` si score ≥ 40, `2` sinon (utile en CI/CD).

## Rapport JSON

Sauvegardé par défaut sous `~/Desktop/MacSecCheck_<hôte>_<horodatage>.json`. Contient le contexte hôte,
les scores par catégorie, chaque constat horodaté (gravité, observé, attendu, remédiation, référence),
et un bloc `Integrity` avec l'empreinte **SHA‑256** re‑vérifiable du rapport.

Vérifier l'intégrité :
```bash
# recalculer le hash du corps (hors bloc Integrity) et comparer
shasum -a 256 MacSecCheck_*.json
```
