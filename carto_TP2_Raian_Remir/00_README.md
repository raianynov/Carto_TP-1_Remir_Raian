# Cartographie des flux et des zones de confiance — ADS (TP2)

## Bien démarrer (à lire en premier)

Ce dossier est le **TP2**. Il complète le TP1 (cartographie des actifs) en représentant les
**interactions** du système d'information ADS : les flux réseau entre composants, les
échanges entre zones de confiance, et les mécanismes de filtrage et d'interconnexion.

**En une phrase :** tous les flux opérationnels convergent vers le cœur `c4i-ads`, et tout
échange entre zones (ADS, INT, BG) passe obligatoirement par les pare-feux qui filtrent les
communications.

> Cohérence avec le TP1 : mêmes actifs, mêmes zones, mêmes adresses. Le TP2 ajoute
> uniquement les flux et les frontières.

## Lien avec le TP1

Ce TP2 s'appuie directement sur la cartographie des actifs du TP1 (dossier voisin
[`carto_TP1_Raian_Remir`](../carto_TP1_Raian_Remir/00_README.md) dans le même dépôt). Les
correspondances :

| Élément du TP2 | Repris du TP1 |
|----------------|---------------|
| Les composants des flux (`c4i-ads`, `radar-ads`…) | [Inventaire des actifs](../carto_TP1_Raian_Remir/03_inventaire_actifs.md) |
| Les trois zones et leurs pare-feux | [Vue logique par zones](../carto_TP1_Raian_Remir/02_cartographies.md) |
| La chaîne de flux radar → c4i → wpn | [Chaîne opérationnelle (vue fonctionnelle)](../carto_TP1_Raian_Remir/04_vues_metier_fonctionnelle.md) |

## Contenu du dossier

| Fichier | Contenu |
|---------|---------|
| [`01_analyse_flux_zones.md`](01_analyse_flux_zones.md) | Analyse : flux principaux, flux critiques, relations entre zones, justification |
| [`02_cartographies.md`](02_cartographies.md) | Les trois vues, avec diagrammes Mermaid intégrés |
| [`diagrammes/vue_flux_composants.mmd`](diagrammes/vue_flux_composants.mmd) | Flux entre composants — source Mermaid |
| [`diagrammes/vue_flux_admin.mmd`](diagrammes/vue_flux_admin.mmd) | Flux administratifs — source Mermaid |
| [`diagrammes/vue_zones_confiance.mmd`](diagrammes/vue_zones_confiance.mmd) | Zones de confiance — source Mermaid |

## Rappel des flux (source : contexte + énoncé TP2)

| Flux | Protocole / Port |
|------|------------------|
| `radar-ads → c4i-ads` | TCP 7000 |
| `map-ads → c4i-ads` | UDP 8080 |
| `mil-ads → c4i-ads` | TCP 5000 / 7000 |
| `vpn-ads → c4i-ads` | TCP 5000 |
| Accès administratifs | SSH 22, VNC 5900–6011 |

## Glossaire technique

| Terme | Signification |
|-------|---------------|
| **Protocole** | Ensemble de règles définissant comment deux machines échangent des données |
| **Port** | Numéro identifiant un service précis sur une machine (ex. : 7000). Une machine peut exposer plusieurs services, chacun sur son port |
| **TCP** (Transmission Control Protocol) | Mode de transport **fiable** : établit une connexion, garantit que les données arrivent complètes et dans l'ordre (paquets perdus renvoyés). Adapté quand l'exactitude prime (ex. : pistes radar) |
| **UDP** (User Datagram Protocol) | Mode de transport **rapide sans garantie** : envoie les données sans connexion ni accusé de réception. Plus léger, adapté aux flux continus où une perte ponctuelle est tolérable (ex. : fond cartographique) |
| **SSH** (Secure Shell, port 22) | Connexion à distance **chiffrée** en ligne de commande, utilisée pour administrer un serveur |
| **VNC** (Virtual Network Computing, ports 5900–6011) | Prise de contrôle à distance de l'**interface graphique** d'une machine |
| **VPN** (Virtual Private Network) | **Tunnel chiffré** sécurisant les communications entre deux points à travers un réseau (ici `vpn-ads` vers le cœur) |

## Ordre de lecture conseillé

1. **Ce README** — vue d'ensemble.
2. [`01_analyse_flux_zones.md`](01_analyse_flux_zones.md) — l'explication des flux et des zones.
3. [`02_cartographies.md`](02_cartographies.md) — les schémas.

## Comment visualiser les diagrammes

Les diagrammes sont en **Mermaid** et s'affichent automatiquement dans
[`02_cartographies.md`](02_cartographies.md) sur tout lecteur Markdown compatible (GitHub,
VS Code avec l'extension Mermaid, Obsidian…). Aucune dépendance externe.
