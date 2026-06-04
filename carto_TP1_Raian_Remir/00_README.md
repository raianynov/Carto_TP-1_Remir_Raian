# Cartographie des actifs du SI — Air Defense System (ADS)

## Bien démarrer (à lire en premier)

Ce document décrit l'organisation du système d'information **ADS** (Air Defense System),
un SI lié à un dispositif de défense anti-aérienne. Il est destiné à toute personne qui
découvre ce SI et doit comprendre **comment il est organisé** et **où se trouve chaque
actif** (serveur, poste, équipement réseau).

**En une phrase :** le SI est découpé en **trois zones** isolées, chacune sur son propre
réseau et derrière son propre pare-feu —

- **ADS** (`10.10.9.0/24`) : le cœur opérationnel, qui détecte, décide et agit.
- **INT** (`10.10.8.0/24`) : le système d'information interne d'entreprise (utilisateurs,
  messagerie, fichiers, supervision de sécurité).
- **BG** (`10.10.12.0/24`) : la zone de terrain (radar de détection), considérée comme non
  fiable.

**Pour localiser un actif :** chaque actif appartient à une seule zone, et son adresse IP
indique immédiatement sa zone (préfixe `10.10.9` = ADS, `10.10.8` = INT, `10.10.12` = BG).
La liste complète avec les adresses se trouve dans [`03_inventaire_actifs.md`](03_inventaire_actifs.md).

> Périmètre : ce livrable porte sur les **actifs uniquement**. Les flux réseau et les
> règles de sécurité ne sont pas traités ici (ils relèvent du TP2).

## Glossaire des sigles

| Sigle | Signification |
|-------|---------------|
| **ADS** | Air Defense System — le système de défense, et le nom de la zone opérationnelle |
| **INT** | Zone du système d'information interne (entreprise) |
| **BG** | Border Guard — zone de terrain / frontière |
| **C4I** | Command, Control, Communications, Computers and Intelligence — le centre de commandement |
| **GSC** | Désignation d'un centre de service (`hq-gsc` au siège, `bg-gsc` en zone BG) |
| **SIEM** | Security Information and Event Management — supervision et détection d'incidents de sécurité |
| **AD** | Active Directory — annuaire de domaine (contrôleurs `dc1`, `dc2`) |
| **WSUS** | Windows Server Update Services — gestion centralisée des mises à jour |
| **VPN** | Virtual Private Network — interconnexion sécurisée |
| **ws** | Workstation — poste de travail utilisateur (`ws1`, `ws2`, `ws3`) |
| **fw** | Firewall — pare-feu (`fw-ads`, `fw-int`, `fw-bg`) |
| **dc** | Domain Controller — contrôleur de domaine (`dc1`, `dc2`) |

## Contenu du dossier

| Fichier | Contenu |
|---------|---------|
| [`01_analyse.md`](01_analyse.md) | Analyse écrite : organisation du SI, définition de chaque zone, localisation des actifs, classification |
| [`02_cartographies.md`](02_cartographies.md) | Les trois vues, avec diagrammes Mermaid intégrés |
| [`03_inventaire_actifs.md`](03_inventaire_actifs.md) | Inventaire structuré (nom, IP, type, service) par zone |
| [`04_vues_metier_fonctionnelle.md`](04_vues_metier_fonctionnelle.md) | Vue métier (mission, acteurs) et vue fonctionnelle (capacités), avec correspondance fonction → actif |
| [`diagrammes/vue_logique_zones.mmd`](diagrammes/vue_logique_zones.mmd) | Vue logique par zones — source Mermaid éditable |
| [`diagrammes/vue_inventaire_categories.mmd`](diagrammes/vue_inventaire_categories.mmd) | Vue inventaire par catégorie — source Mermaid éditable |
| [`diagrammes/vue_physique.mmd`](diagrammes/vue_physique.mmd) | Vue physique/réseau — source Mermaid éditable |

## Ordre de lecture conseillé

1. **Ce README** — vue d'ensemble et glossaire.
2. [`01_analyse.md`](01_analyse.md) — comment le SI est organisé, zone par zone.
3. [`02_cartographies.md`](02_cartographies.md) — les schémas pour visualiser l'ensemble.
4. [`03_inventaire_actifs.md`](03_inventaire_actifs.md) — le détail pour retrouver un actif précis.
5. [`04_vues_metier_fonctionnelle.md`](04_vues_metier_fonctionnelle.md) — pour comprendre le SI côté mission et capacités.

## Comment visualiser les diagrammes

Tous les documents sont visibles sur Github.
