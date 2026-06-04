# Analyse des actifs — Système d'Information *Air Defense System (ADS)*

> **TP1 — Cartographie des actifs du SI**
> Document d'analyse écrite. Les cartographies associées sont fournies dans
> [`02_cartographies.md`](02_cartographies.md), avec les sources Mermaid dans `diagrammes/`.

> Voir aussi : [`00_README.md`](00_README.md) (vue d'ensemble et glossaire) et [`03_inventaire_actifs.md`](03_inventaire_actifs.md) (inventaire détaillé).

---

## 1. Démarche

L'analyse a consisté à recenser tous les composants décrits dans le contexte, puis à les
rattacher à leur zone d'appartenance et à les classer par nature (serveurs, équipements
réseau, postes utilisateurs, services). Le SI est d'abord présenté zone par zone — chaque
zone étant définie par son utilité, son contenu et sa place dans l'architecture — puis les
actifs sont localisés précisément. L'inventaire complet (nom, IP, type, rôle) figure dans
[`03_inventaire_actifs.md`](03_inventaire_actifs.md).

## 2. Structuration globale

Le système ADS est découpé en **trois zones** distinctes, chacune correspondant à un
réseau IPv4 dédié et protégée par un pare-feu périmétrique qui lui est propre. Aucune
machine n'appartient à deux zones à la fois : l'adresse IP d'un actif suffit à déterminer
la zone dans laquelle il se trouve (préfixe `10.10.9` pour ADS, `10.10.8` pour INT,
`10.10.12` pour BG).

| Zone | Réseau | Pare-feu |
|------|--------|----------|
| **ADS** | `10.10.9.0/24` | `fw-ads` (`100.97.10.114`) |
| **INT** | `10.10.8.0/24` | `fw-int` (`100.97.10.115`) |
| **BG**  | `10.10.12.0/24` | `fw-bg` (`100.97.10.242`) |

Les trois pare-feux portent des adresses dans un plan distinct (`100.97.10.0/24`),
cohérent avec leur rôle de frontière entre les réseaux. Ils sont reliés entre eux par un
maillage de liens contrôlés ; le détail de ce filtrage relève du TP2.

**Pourquoi ce découpage ?** Séparer le SI en zones permet d'isoler les environnements
selon leur rôle et le niveau de confiance qu'on leur accorde. Le cœur opérationnel (ADS)
n'est pas mélangé avec la bureautique d'entreprise (INT), ni avec la zone de terrain (BG)
que le contexte considère comme non fiable. Ainsi, un problème survenant dans une zone
(par exemple une compromission côté terrain) ne se propage pas directement aux autres : le
pare-feu de chaque zone agit comme une barrière. C'est ce principe de segmentation qui
structure tout le reste de la cartographie.

## 3. Définition des zones

### 3.1 Zone ADS — Système de défense

**Utilité.** C'est le cœur opérationnel du système : c'est ici que les données de
détection sont reçues, analysées, et que les décisions de défense anti-aérienne sont
prises et exécutées. Le contexte qualifie cette zone d'opérationnelle et temps réel.

**Situation dans l'architecture.** Zone centrale du dispositif, sur le réseau
`10.10.9.0/24`, derrière le pare-feu `fw-ads`. C'est vers elle que convergent les
informations produites par les autres zones (notamment le radar situé en BG).

**Contenu.** Elle regroupe les serveurs opérationnels de la chaîne de défense ainsi que
le point d'entrée VPN et les postes des opérateurs :

| Actif | Adresse IP | Rôle |
|-------|-----------|------|
| `c4i-ads` | `10.10.9.32` | Centre de commandement C4I (Command, Control, Communications, Computers and Intelligence) : agrège les données radar, classe les menaces, décide des interceptions. Composant central du système. |
| `wpn-ads` | `10.10.9.35` | Système d'armement anti-aérien, qui exécute les ordres. |
| `mil-ads` | `10.10.9.31` | Supervision militaire, en lecture seule. |
| `map-ads` | `10.10.9.34` | Serveur cartographique, support de visualisation. |
| `vpn-ads` | `10.10.9.36` | Point d'entrée VPN sécurisé de la zone. |
| Postes utilisateurs ADS | `10.10.9.250` à `10.10.9.254` | Opérateurs de la défense. |

### 3.2 Zone INT — Système d'information interne

**Utilité.** C'est le système d'information d'entreprise classique de l'organisation :
gestion des utilisateurs, bureautique, messagerie, stockage, et supervision de la sécurité
du SI. Elle assure le fonctionnement administratif et le pilotage de sécurité, en support
de la mission opérationnelle.

**Situation dans l'architecture.** Zone interne sur le réseau `10.10.8.0/24`, derrière le
pare-feu `fw-int`. Elle est de confiance, distincte du domaine opérationnel (ADS) comme du
terrain (BG).

**Contenu.** C'est la zone la plus fournie. Elle se décompose en trois ensembles :

| Ensemble | Actifs | Adresse IP | Rôle |
|----------|--------|-----------|------|
| Infrastructure | `dc1`, `dc2` | *(non fournies)* | Contrôleurs de domaine Active Directory |
| Infrastructure | `files` | *(non fournie)* | Serveur de fichiers |
| Infrastructure | `mail` | *(non fournie)* | Messagerie |
| Infrastructure | `sql` | *(non fournie)* | Base de données |
| Infrastructure | `wsus` | *(non fournie)* | Gestion des mises à jour |
| Infrastructure | `reports` | *(non fournie)* | Reporting |
| Infrastructure | `hq-gsc` | *(non fournie)* | Service central |
| SIEM | `siem-dashboard` | `10.10.8.70` | Tableau de bord de supervision |
| SIEM | `siem-indexer` | `10.10.8.71` | Indexation des journaux |
| SIEM | `siem-server` | `10.10.8.72` | Collecte des logs, détection d'incidents |
| Postes | `ws1` | `10.10.8.31` à `.35` | Postes utilisateurs |
| Postes | `ws2` | `10.10.8.36` à `.40` | Postes utilisateurs |
| Postes | `ws3` | `10.10.8.41` à `.45` | Postes utilisateurs |

Le SIEM (Security Information and Event Management — `siem-dashboard`, `siem-indexer`,
`siem-server`) forme un sous-ensemble dédié à la supervision de sécurité : collecte des
journaux, détection d'incidents. Les sigles employés dans ce document sont rassemblés dans
le glossaire du fichier [`00_README.md`](00_README.md).

### 3.3 Zone BG — Border Guard

**Utilité.** C'est la zone de terrain : elle héberge le moyen de détection aérienne et son
centre de contrôle local. Elle produit l'information de détection qui alimente la zone ADS.
Le contexte la considère comme non fiable.

**Situation dans l'architecture.** Zone externe sur le réseau `10.10.12.0/24`, derrière le
pare-feu `fw-bg`. Elle est en périphérie du dispositif, au plus près du terrain, et c'est
la zone à laquelle on accorde le moins de confiance.

**Contenu.**

| Actif | Adresse IP | Rôle |
|-------|-----------|------|
| `radar-ads` | `10.10.12.33` | Système radar de détection aérienne, source de l'information amont. Malgré son préfixe « ads », il est situé dans la zone BG, pas dans la zone ADS. |
| `bg-gsc` | `10.10.12.64` | Centre de contrôle local de la zone. |
| Postes utilisateurs BG | `10.10.12.250` à `10.10.12.254` | Opérateurs de terrain. |

## 4. Classification transversale des actifs

Indépendamment des zones, les actifs se répartissent en quatre catégories :

| Catégorie | Actifs |
|-----------|--------|
| Serveurs et machines | `c4i-ads`, `mil-ads`, `map-ads`, `wpn-ads` (ADS) ; infrastructure et SIEM (INT) ; `radar-ads`, `bg-gsc` (BG) |
| Équipements réseau | Pare-feux `fw-ads`, `fw-int`, `fw-bg` ; point VPN `vpn-ads` |
| Postes utilisateurs | Groupes `ws1` / `ws2` / `ws3` (INT) ; postes des zones ADS et BG |
| Services | Commandement C4I, armement, détection radar, cartographie, supervision militaire, annuaire de domaine, messagerie, fichiers, base de données, gestion des mises à jour, reporting, supervision de sécurité (SIEM) |

Parmi ces services, les **services critiques** — ceux dont dépend directement la mission de
défense — sont : le **commandement C4I** (`c4i-ads`), l'**armement** (`wpn-ads`), la
**détection radar** (`radar-ads`) et la **liaison VPN** qui relie le radar à l'armement
(`vpn-ads`). Les services de support comme l'annuaire de domaine (AD) ou la supervision de
sécurité (SIEM), bien qu'importants pour le fonctionnement du SI interne, ne sont pas
classés comme critiques pour la mission opérationnelle.

## 5. Justification des choix de représentation

Trois vues complémentaires ont été produites, car aucune représentation unique ne couvre
l'ensemble des attendus de manière lisible.

- **Vue logique par zones** : organise les actifs par zone et par regroupement logique
  (infrastructure, SIEM, postes). C'est la vue principale, puisque le découpage en zones
  est le principe structurant du SI.
- **Vue inventaire par catégorie** : lecture transversale par type d'actif, qui répond à
  l'attendu de classification et complète la vue par zones.
- **Vue physique / réseau** : rattache chaque actif à son segment réseau, lui-même relié
  au pare-feu de sa zone ; elle met en évidence le découpage réseau et le rôle de point de
  passage des pare-feux.

## 6. Synthèse

Le SI ADS se structure en trois zones aux rôles distincts : ADS, le cœur opérationnel qui
décide et agit ; INT, le système d'information interne qui administre et supervise ; et BG,
la zone de terrain qui détecte. Chaque actif appartient à une seule zone, identifiable par
son adresse IP, et chaque zone est isolée derrière son pare-feu. Le composant `c4i-ads`
occupe la position centrale, recevant l'information produite ailleurs pour piloter la
défense. Les trois vues produites couvrent l'ensemble des actifs recensés sous des angles
complémentaires.
