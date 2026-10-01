# TP2 – Analyse de vulnérabilités (Audit des infrastructures et conformité)

> Module **BC01 – Audit des infrastructures et conformité** · ESATIC · Master ERIS (Expert Réseaux, Infrastructures et Sécurité)

**Auteurs** : DIABY Aboubacar Sidik · TEHI Stella · OSSOHOU Tresor

Simulation d'une analyse de vulnérabilités sur un laboratoire isolé : découverte réseau, scans non authentifié et authentifié, validation des résultats et priorisation des mesures correctives.

> ⚠️ Tous les scans ont été réalisés **uniquement dans le laboratoire** (sous-réseau privé `10.10.20.0/24`), sous autorisation. Ne reproduisez ces techniques que sur des systèmes dont vous êtes propriétaire ou pour lesquels vous avez une autorisation écrite.

## Environnement

| Rôle | Machine | Adresse |
|---|---|---|
| Machine d'audit | Kali Linux | 10.10.20.10 |
| Cible | srv-linux (Ubuntu 22.04.5 LTS) | 10.10.20.15 |
| Équipements réseau | R1, SW1 | 10.10.20.1 / 10.10.20.2 |

Schéma : [`preuves/captures/Architecture_du_Laboratoire.png`](preuves/captures/Architecture_du_Laboratoire.png)

## Méthode

1. **Découverte des hôtes** – Nmap (`-sn`) : 4 hôtes actifs
2. **Services** – Nmap (`-sS -sV -O -p-`, UDP top 50) : 22/tcp OpenSSH 8.9p1, 80/tcp Apache 2.4.52
3. **Détection par bannières** – Nmap `--script vuln` (vulners)
4. **Scan non authentifié** – OpenVAS 25.04, config *Full and fast*
5. **Scan authentifié** – OpenVAS avec un compte SSH d'audit en lecture seule
6. **Validation** – `apt changelog`, `debsecan`, catalogue CISA KEV

## Résultats : 3 vulnérabilités documentées

| Vulnérabilité | CVSS | Nature | Risque réel | Action |
|---|---|---|---|---|
| CVE-2024-6387 (OpenSSH, regreSSHion) | 8.1 | Faux positif (backport Ubuntu) | Nul | Consigner |
| CVE-2022-31813 (Apache mod_proxy) | 9.8 | Faux positif (backport Ubuntu) | Nul | Consigner |
| CVE-2025-32462 (sudo `--host`) | 8.8 | **Vrai positif** | Modéré (accès local requis) | Mettre à jour sudo |

### Enseignements

- Une **bannière de version** ou un **score CVSS élevé** ne suffit pas à conclure : Ubuntu rétroporte les correctifs sans changer le numéro de version.
- Le scan **authentifié** voit les versions réellement installées (il est le seul à remonter VMScape et TSA).
- Le vrai positif (sudo) n'est apparu ni dans Nmap ni dans les résultats OpenVAS retenus : seul `debsecan` l'a révélé. **Aucun outil ne suffit seul : il faut croiser les sources.**

## Contenu du dépôt

```
docs/
  Rapport_complet.pdf        Rapport complet (35 pages)
  Livrable_synthese.pdf      Livrable de synthèse (4 pages)
  Presentation.pptx          Support de présentation
preuves/
  captures/                  Captures d'écran (OpenVAS, SSH, architecture)
  scans-nmap/                Commandes, sorties et empreintes SHA-256
  rapports-openvas/          Rapports XML (scan authentifié / non authentifié)
```

## Outils

Nmap · OpenVAS (Greenbone) · Kali Linux · debsecan · CISA KEV · NVD

## Avertissement

Document réalisé dans un cadre pédagogique. Les adresses IP appartiennent à un laboratoire privé et isolé.
