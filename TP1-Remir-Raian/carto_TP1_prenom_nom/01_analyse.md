# Analyse des actifs — Système d'Information *Air Defense System (ADS)*

> **TP1 — Cartographie des actifs du SI**
> Document d'analyse écrite. Les cartographies associées sont fournies dans
> `02_cartographies.md`, avec les sources Mermaid dans `diagrammes/`.
> Conformément au périmètre du TP1, ce document porte **uniquement sur les actifs** :
> les flux, règles de filtrage et communications inter-zones relèvent du TP2.

---

## 1. Démarche

La cartographie a été construite en trois temps :

1. **Recensement** exhaustif des composants décrits dans le contexte, zone par zone.
2. **Classification** des actifs par nature (serveurs, réseau, utilisateurs, services).
3. **Représentation** selon plusieurs vues complémentaires, chacune répondant à un
   besoin de lecture différent.

L'inventaire détaillé (nom, IP, rôle, criticité) figure dans `03_inventaire_actifs.md`.

---

## 2. Structuration globale des actifs

Le SI ADS est organisé en **trois zones de confiance** distinctes, chacune correspondant
à un réseau IPv4 dédié et protégée par un pare-feu périmétrique :

| Zone | Réseau | Pare-feu | Nature |
|------|--------|----------|--------|
| **ADS** | `10.10.9.0/24` | `fw-ads` (`100.97.10.114`) | Zone opérationnelle critique, temps réel |
| **INT** | `10.10.8.0/24` | `fw-int` (`100.97.10.115`) | SI d'entreprise interne |
| **BG**  | `10.10.12.0/24` | `fw-bg` (`100.97.10.242`) | Zone externe / terrain, **non fiable** |

Cette segmentation en zones constitue le **principe structurant majeur** du SI : elle
isole les environnements selon leur niveau de criticité et de confiance. Les trois
pare-feux ne sont pas isolés ; ils forment un maillage de filtrage entre zones (le détail
de ce filtrage est l'objet du TP2).

On note que les pare-feux portent des adresses dans un plan d'adressage distinct
(`100.97.10.0/24`), cohérent avec leur rôle d'interconnexion entre les réseaux internes.

---

## 3. Classification des actifs

Les actifs ont été regroupés en **quatre catégories**, conformément aux attendus du TP.

### 3.1 Serveurs et machines

- **Zone ADS** : `c4i-ads`, `mil-ads`, `map-ads`, `wpn-ads` — serveurs opérationnels du
  système de défense.
- **Zone INT** : infrastructure d'entreprise (`dc1`, `dc2`, `files`, `mail`, `sql`,
  `wsus`, `reports`, `hq-gsc`) et briques SIEM (`siem-dashboard`, `siem-indexer`,
  `siem-server`).
- **Zone BG** : `radar-ads` (capteur radar) et `bg-gsc` (contrôle local).

### 3.2 Postes utilisateurs

Trois ensembles de postes, identifiés par plages d'adresses :

- Utilisateurs **ADS** : `10.10.9.250 – 254`
- Utilisateurs **INT** : `ws1` (`.31–35`), `ws2` (`.36–40`), `ws3` (`.41–45`) sur `10.10.8.0/24`
- Utilisateurs **BG** : `10.10.12.250 – 254`

### 3.3 Équipements réseau

- **Pare-feux** : `fw-ads`, `fw-int`, `fw-bg` (périmétriques, un par zone).
- **VPN** : `vpn-ads` (`10.10.9.36`), point d'entrée sécurisé d'interconnexion.

### 3.4 Services identifiés

Sans décrire les communications, les services portés par les actifs sont notamment :
commandement C4I, armement, détection radar, cartographie, supervision militaire,
annuaire de domaine (AD), messagerie, fichiers, base de données, gestion des mises à
jour, reporting, et supervision de sécurité (SIEM).

---

## 4. Identification des actifs critiques

La criticité a été évaluée à partir du rôle de chaque actif dans la mission de défense
et de l'impact d'une compromission ou indisponibilité.

| Actif | Zone | Justification de la criticité |
|-------|------|-------------------------------|
| **`c4i-ads`** | ADS | **Cœur du système** : agrège le radar, classe les menaces et décide des interceptions. Sa perte paralyse toute la chaîne décisionnelle. |
| **`wpn-ads`** | ADS | Système d'armement : sa compromission a des conséquences physiques directes. |
| **`radar-ads`** | BG | Source unique de détection aérienne ; sans lui, plus d'alerte amont. Sa présence en zone *non fiable* accroît le risque. |
| **`dc1` / `dc2`** | INT | Contrôleurs de domaine : pivot d'authentification de tout le SI interne. |
| **`siem-server`** | INT | Cœur de la supervision sécurité ; sa perte aveugle la détection d'incidents. |
| **`vpn-ads`** | ADS | Point d'entrée des interconnexions : porte d'accès sensible. |

Le contexte qualifie explicitement la zone **ADS** de critique et temps réel, et la zone
**BG** de non fiable. Le fait que le radar (`radar-ads`), actif vital, soit situé en
zone BG non fiable constitue un point d'attention majeur de la cartographie.

---

## 5. Justification des choix de représentation

Trois vues complémentaires ont été produites, car aucune représentation unique ne couvre
l'ensemble des attendus de manière lisible.

### 5.1 Vue logique par zones (Mermaid)
**But :** montrer l'organisation des actifs par zones de confiance et leur regroupement
logique (infrastructure, SIEM, postes). C'est la vue principale, car la segmentation est
le principe structurant du SI. Le code couleur traduit le niveau de confiance (critique /
interne / non fiable). Choix de Mermaid : intégrable au Markdown, versionnable, lisible.

### 5.2 Vue inventaire par catégorie (Mermaid)
**But :** offrir une lecture transversale par **type d'actif** (serveurs, réseau,
utilisateurs, services), indépendamment des zones. Elle répond directement à l'attendu de
classification et complète la vue logique qui, elle, raisonne par zone.

### 5.3 Vue physique / réseau (Mermaid)
**But :** représentation orientée « infrastructure » : chaque actif est rattaché à son
segment réseau, lui-même relié au pare-feu de la zone. Elle met en évidence le découpage
physique et le rôle de point de passage des pare-feux.

### 5.4 Choix transverses
- **Les flux ne sont pas tracés** dans les diagrammes : le TP1 porte sur les actifs. Seul
  le *maillage* des pare-feux est représenté en pointillés, comme élément structurel, sans
  détailler les règles.
- **Aucune adresse IP n'a été inventée** : les serveurs INT dont l'IP n'est pas fournie
  sont représentés sans adresse, et ce manque est explicitement signalé. Cela préserve la
  fidélité de la cartographie au contexte.

---

## 6. Synthèse

Le SI ADS est un environnement multi-zones dont la structuration repose sur trois zones de
confiance segmentées par pare-feux. Les actifs se répartissent en serveurs, postes
utilisateurs, équipements réseau et services. Le système est organisé autour d'un cœur,
`c4i-ads`, qui concentre la criticité opérationnelle, tandis que des actifs vitaux comme le
radar se trouvent en zone non fiable. Les vues logique, inventaire et physique produites
offrent des angles de lecture complémentaires couvrant l'ensemble des actifs recensés.
