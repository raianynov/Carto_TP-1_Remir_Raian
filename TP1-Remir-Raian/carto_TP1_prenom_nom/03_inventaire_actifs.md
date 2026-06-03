# Inventaire structuré des actifs — Système ADS

> Recensement exhaustif des actifs du SI, organisé par zone de confiance.
> Source : document de contexte *Air Defense System (ADS)*.

---

## Vue d'ensemble des zones

| Zone | Libellé | Réseau (IPv4) | Pare-feu périmétrique | Niveau de confiance |
|------|---------|---------------|------------------------|---------------------|
| **ADS** | Système de défense (opérationnel, temps réel) | `10.10.9.0/24` | `fw-ads` — `100.97.10.114` | Critique |
| **INT** | Système d'information interne d'entreprise | `10.10.8.0/24` | `fw-int` — `100.97.10.115` | Interne / de confiance |
| **BG**  | Border Guard (terrain opérationnel) | `10.10.12.0/24` | `fw-bg` — `100.97.10.242` | Non fiable (externe) |

---

## Zone ADS — Système de défense (`10.10.9.0/24`)

| Actif | Adresse IP | Type | Service / Rôle | Criticité |
|-------|-----------|------|----------------|-----------|
| `c4i-ads` | `10.10.9.32` | Serveur | Centre de commandement C4I : agrégation radar, classification des menaces, décision d'interception | **Vitale** (cœur du système) |
| `mil-ads` | `10.10.9.31` | Serveur | Supervision militaire (lecture seule) | Haute |
| `map-ads` | `10.10.9.34` | Serveur | Serveur cartographique | Haute |
| `wpn-ads` | `10.10.9.35` | Serveur | Système d'armement anti-aérien | **Vitale** |
| `vpn-ads` | `10.10.9.36` | Équipement réseau | Point d'entrée VPN sécurisé | Haute |
| Utilisateurs ADS | `10.10.9.250 – 10.10.9.254` | Postes utilisateurs | Opérateurs de la zone de défense | Moyenne |

## Zone INT — Système d'information interne (`10.10.8.0/24`)

### Infrastructure (serveurs)

| Actif | Adresse IP | Type | Service / Rôle | Criticité |
|-------|-----------|------|----------------|-----------|
| `dc1` | *(non précisée)* | Serveur | Contrôleur de domaine (Active Directory) | **Vitale** |
| `dc2` | *(non précisée)* | Serveur | Contrôleur de domaine (redondance) | **Vitale** |
| `files` | *(non précisée)* | Serveur | Serveur de fichiers | Haute |
| `mail` | *(non précisée)* | Serveur | Messagerie | Haute |
| `sql` | *(non précisée)* | Serveur | Base de données | Haute |
| `wsus` | *(non précisée)* | Serveur | Gestion des mises à jour (WSUS) | Moyenne |
| `reports` | *(non précisée)* | Serveur | Reporting | Moyenne |
| `hq-gsc` | *(non précisée)* | Serveur | Service central (HQ) | Haute |

### SIEM (supervision sécurité)

| Actif | Adresse IP | Type | Service / Rôle | Criticité |
|-------|-----------|------|----------------|-----------|
| `siem-dashboard` | `10.10.8.70` | Serveur | Tableau de bord SIEM | Haute |
| `siem-indexer` | `10.10.8.71` | Serveur | Indexation des journaux | Haute |
| `siem-server` | `10.10.8.72` | Serveur | Collecte des logs, détection d'incidents | **Vitale** |

### Postes utilisateurs

| Groupe | Plage IP | Type |
|--------|----------|------|
| `ws1` | `10.10.8.31 – 10.10.8.35` | Postes utilisateurs |
| `ws2` | `10.10.8.36 – 10.10.8.40` | Postes utilisateurs |
| `ws3` | `10.10.8.41 – 10.10.8.45` | Postes utilisateurs |

## Zone BG — Border Guard (`10.10.12.0/24`)

| Actif | Adresse IP | Type | Service / Rôle | Criticité |
|-------|-----------|------|----------------|-----------|
| `radar-ads` | `10.10.12.33` | Serveur / capteur | Système radar de détection aérienne | **Vitale** |
| `bg-gsc` | `10.10.12.64` | Serveur | Centre de contrôle local | Haute |
| Utilisateurs BG | `10.10.12.250 – 10.10.12.254` | Postes utilisateurs | Opérateurs terrain | Moyenne |

---

## Synthèse quantitative

| Catégorie | Nombre d'actifs |
|-----------|-----------------|
| Pare-feux | 3 |
| Serveurs (zone ADS) | 4 |
| Serveurs (zone INT) | 11 (8 infra + 3 SIEM) |
| Serveurs / capteurs (zone BG) | 2 |
| Équipement VPN | 1 |
| Groupes de postes utilisateurs | 5 (ADS, ws1, ws2, ws3, BG) |

> **Note :** les adresses IP des serveurs de la zone INT (infrastructure) ne sont pas
> fournies dans le contexte ; seules celles du SIEM le sont. Ce manque est signalé
> tel quel plutôt que d'inventer des valeurs.
