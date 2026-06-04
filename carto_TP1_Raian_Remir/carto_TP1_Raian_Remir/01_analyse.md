# Architecture des actifs — Air Defense System (ADS)

> Documentation de l'architecture du système ADS, côté actifs.
> Cartographies associées : [`02_cartographies.md`](02_cartographies.md) · inventaire complet : [`03_inventaire_actifs.md`](03_inventaire_actifs.md) · glossaire : [`00_README.md`](00_README.md).
> Les flux réseau et les règles de filtrage sont documentés séparément.

---

## Structuration générale

Le système ADS est segmenté en trois zones, chacune sur un réseau IPv4 dédié et protégée
par un pare-feu périmétrique. Le découpage est strict : un actif appartient à une seule
zone, et son adresse IP suffit à l'identifier (préfixe `10.10.9` pour ADS, `10.10.8` pour
INT, `10.10.12` pour BG).

| Zone | Réseau | Pare-feu |
|------|--------|----------|
| **ADS** | `10.10.9.0/24` | `fw-ads` (`100.97.10.114`) |
| **INT** | `10.10.8.0/24` | `fw-int` (`100.97.10.115`) |
| **BG**  | `10.10.12.0/24` | `fw-bg` (`100.97.10.242`) |

Les pare-feux sont adressés sur un plan distinct (`100.97.10.0/24`), conformément à leur
rôle de frontière entre réseaux, et sont reliés par un maillage de liens contrôlés.

La segmentation isole les environnements selon leur rôle et leur niveau de confiance : le
cœur opérationnel (ADS) reste séparé de la bureautique d'entreprise (INT) et de la zone de
terrain (BG), classée non fiable. Une compromission dans une zone ne se propage pas
directement aux autres, chaque pare-feu jouant le rôle de barrière.

---

## Zone ADS — Système de défense

Cœur opérationnel du système : réception et analyse des données de détection, prise de
décision et exécution de la défense anti-aérienne. Zone temps réel, située au centre du
dispositif sur `10.10.9.0/24` derrière `fw-ads`. Les informations produites par les autres
zones (en particulier le radar, hébergé en BG) convergent vers elle.

| Actif | Adresse IP | Rôle |
|-------|-----------|------|
| `c4i-ads` | `10.10.9.32` | Centre de commandement C4I (Command, Control, Communications, Computers and Intelligence) : agrège les données radar, classe les menaces, décide des interceptions. Composant central. |
| `wpn-ads` | `10.10.9.35` | Système d'armement anti-aérien ; exécute les ordres. |
| `mil-ads` | `10.10.9.31` | Supervision militaire, en lecture seule. |
| `map-ads` | `10.10.9.34` | Serveur cartographique ; support de visualisation. |
| `vpn-ads` | `10.10.9.36` | Point d'entrée VPN sécurisé de la zone. |
| Postes utilisateurs ADS | `10.10.9.250` – `10.10.9.254` | Opérateurs de la défense. |

---

## Zone INT — Système d'information interne

Système d'information d'entreprise : gestion des utilisateurs, bureautique, messagerie,
stockage et supervision de la sécurité du SI. En support de la mission opérationnelle.
Située sur `10.10.8.0/24` derrière `fw-int`, distincte du domaine opérationnel (ADS) et du
terrain (BG). C'est la zone la plus dense, organisée en trois ensembles.

| Ensemble | Actifs | Adresse IP | Rôle |
|----------|--------|-----------|------|
| Infrastructure | `dc1`, `dc2` | *(à documenter)* | Contrôleurs de domaine Active Directory |
| Infrastructure | `files` | *(à documenter)* | Serveur de fichiers |
| Infrastructure | `mail` | *(à documenter)* | Messagerie |
| Infrastructure | `sql` | *(à documenter)* | Base de données |
| Infrastructure | `wsus` | *(à documenter)* | Gestion des mises à jour |
| Infrastructure | `reports` | *(à documenter)* | Reporting |
| Infrastructure | `hq-gsc` | *(à documenter)* | Service central |
| SIEM | `siem-dashboard` | `10.10.8.70` | Tableau de bord de supervision |
| SIEM | `siem-indexer` | `10.10.8.71` | Indexation des journaux |
| SIEM | `siem-server` | `10.10.8.72` | Collecte des logs, détection d'incidents |
| Postes | `ws1` | `10.10.8.31` – `.35` | Postes utilisateurs |
| Postes | `ws2` | `10.10.8.36` – `.40` | Postes utilisateurs |
| Postes | `ws3` | `10.10.8.41` – `.45` | Postes utilisateurs |

Le SIEM (Security Information and Event Management) regroupe `siem-dashboard`,
`siem-indexer` et `siem-server` pour la supervision de sécurité : collecte des journaux et
détection d'incidents. Les adresses des serveurs d'infrastructure ne sont pas connues à ce
stade et restent à documenter. Sigles détaillés dans le glossaire ([`00_README.md`](00_README.md)).

---

## Zone BG — Border Guard

Zone de terrain : héberge le moyen de détection aérienne et son centre de contrôle local,
et produit l'information de détection qui alimente ADS. Située sur `10.10.12.0/24` derrière
`fw-bg`, en périphérie du dispositif et au niveau de confiance le plus bas.

| Actif | Adresse IP | Rôle |
|-------|-----------|------|
| `radar-ads` | `10.10.12.33` | Radar de détection aérienne, source de l'information amont. Malgré son préfixe « ads », il est rattaché à la zone BG. |
| `bg-gsc` | `10.10.12.64` | Centre de contrôle local de la zone. |
| Postes utilisateurs BG | `10.10.12.250` – `10.10.12.254` | Opérateurs de terrain. |

---

## Classification transversale

Par nature, indépendamment des zones, les actifs se répartissent en quatre catégories.

| Catégorie | Actifs |
|-----------|--------|
| Serveurs et machines | `c4i-ads`, `mil-ads`, `map-ads`, `wpn-ads` (ADS) ; infrastructure et SIEM (INT) ; `radar-ads`, `bg-gsc` (BG) |
| Équipements réseau | Pare-feux `fw-ads`, `fw-int`, `fw-bg` ; point VPN `vpn-ads` |
| Postes utilisateurs | Groupes `ws1` / `ws2` / `ws3` (INT) ; postes des zones ADS et BG |
| Services | Commandement C4I, armement, détection radar, cartographie, supervision militaire, annuaire de domaine, messagerie, fichiers, base de données, gestion des mises à jour, reporting, supervision de sécurité (SIEM) |

Les **services critiques**, dont dépend directement la mission de défense, sont le
commandement C4I (`c4i-ads`), l'armement (`wpn-ads`), la détection radar (`radar-ads`) et
la liaison VPN reliant le radar à l'armement (`vpn-ads`). L'annuaire de domaine et le SIEM,
nécessaires au fonctionnement du SI interne, ne sont pas critiques pour la mission
opérationnelle.

---

## Représentations

Trois vues complémentaires couvrent l'architecture sous des angles différents.

- **Vue logique par zones** — actifs organisés par zone et par regroupement logique
  (infrastructure, SIEM, postes). Vue principale, alignée sur le découpage en zones qui
  structure le système.
- **Vue inventaire par catégorie** — lecture transversale par type d'actif.
- **Vue physique / réseau** — rattachement de chaque actif à son segment réseau et au
  pare-feu de sa zone ; met en évidence les points de passage.

Les flux applicatifs ne sont pas tracés ici (seul le maillage des pare-feux figure, comme
élément de structure). Les adresses IP non connues sont signalées comme telles plutôt que
renseignées arbitrairement.

---

## Synthèse

ADS s'organise autour de trois zones aux rôles distincts : ADS décide et agit, INT
administre et supervise, BG détecte. Chaque actif appartient à une seule zone, identifiable
par son adresse, et chaque zone est isolée derrière son pare-feu. Le `c4i-ads` occupe la
position centrale : il reçoit l'information produite ailleurs pour piloter la défense.
