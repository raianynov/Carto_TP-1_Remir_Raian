# Cartographie des actifs du SI — TP1 (Air Defense System)

Livrable du **TP1 — Cartographie des actifs du SI**. Périmètre : **actifs uniquement**
(les flux et règles de sécurité relèvent du TP2).

## Contenu du dossier

| Fichier | Contenu |
|---------|---------|
| `01_analyse.md` | Analyse écrite : démarche, structuration, classification, actifs critiques, justification des choix |
| `02_cartographies.md` | Les trois vues, avec diagrammes Mermaid intégrés |
| `03_inventaire_actifs.md` | Inventaire structuré (nom, IP, type, service, criticité) par zone |
| `diagrammes/vue_logique_zones.mmd` | Vue logique par zones — source Mermaid éditable |
| `diagrammes/vue_inventaire_categories.mmd` | Vue inventaire par catégorie — source Mermaid éditable |
| `diagrammes/vue_physique.mmd` | Vue physique/réseau — source Mermaid éditable |

## Ordre de lecture conseillé

1. `01_analyse.md` (le raisonnement)
2. `02_cartographies.md` (les vues)
3. `03_inventaire_actifs.md` (le détail)

## Comment visualiser les diagrammes

- **Mermaid (`.mmd`)** : rendu automatique dans `02_cartographies.md` sur tout lecteur
  Markdown compatible (GitHub, VS Code + extension Mermaid, Obsidian…).

Toutes les vues sont en Mermaid ; aucune dépendance externe (pas de fichier draw.io).
