# Flux réseau et zones de confiance — Air Defense System (ADS)

## Présentation

Documentation des interactions du système ADS : flux réseau entre composants, échanges
entre zones de confiance, mécanismes de filtrage et d'interconnexion.

**En résumé :** les flux opérationnels convergent tous vers le cœur `c4i-ads`, et tout
échange entre zones (ADS, INT, BG) passe obligatoirement par les pare-feux, qui filtrent
les communications.

Cette documentation s'appuie sur l'architecture des actifs (dossier voisin
[`carto_TP1_Raian_Remir`](../carto_TP1_Raian_Remir/00_README.md)) : mêmes composants, mêmes
zones, mêmes adresses ; seules les interactions sont ajoutées ici.

| Élément documenté ici | Référence dans l'architecture des actifs |
|-----------------------|-------------------------------------------|
| Composants des flux (`c4i-ads`, `radar-ads`…) | [Inventaire des actifs](../carto_TP1_Raian_Remir/03_inventaire_actifs.md) |
| Zones et pare-feux | [Vue logique par zones](../carto_TP1_Raian_Remir/02_cartographies.md) |
| Chaîne radar → c4i → wpn | [Vue fonctionnelle](../carto_TP1_Raian_Remir/04_vues_metier_fonctionnelle.md) |

## Contenu du dossier

| Fichier | Contenu |
|---------|---------|
| [`01_analyse_flux_zones.md`](01_analyse_flux_zones.md) | Flux principaux, flux critiques, relations entre zones |
| [`02_cartographies.md`](02_cartographies.md) | Les trois vues, avec diagrammes Mermaid intégrés |
| [`diagrammes/vue_flux_composants.mmd`](diagrammes/vue_flux_composants.mmd) | Flux entre composants — source Mermaid |
| [`diagrammes/vue_flux_admin.mmd`](diagrammes/vue_flux_admin.mmd) | Flux administratifs — source Mermaid |
| [`diagrammes/vue_zones_confiance.mmd`](diagrammes/vue_zones_confiance.mmd) | Zones de confiance — source Mermaid |

## Référence des flux

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
| **TCP** (Transmission Control Protocol) | Transport **fiable** : établit une connexion, garantit l'arrivée complète et ordonnée des données (paquets perdus renvoyés). Adapté quand l'exactitude prime (ex. : pistes radar) |
| **UDP** (User Datagram Protocol) | Transport **rapide sans garantie** : envoie les données sans connexion ni accusé de réception. Plus léger, adapté aux flux continus tolérant une perte ponctuelle (ex. : fond cartographique) |
| **SSH** (Secure Shell, port 22) | Connexion distante **chiffrée** en ligne de commande, pour administrer un serveur |
| **VNC** (Virtual Network Computing, ports 5900–6011) | Prise de contrôle à distance de l'**interface graphique** d'une machine |
| **VPN** (Virtual Private Network) | **Tunnel chiffré** sécurisant les communications entre deux points (ici `vpn-ads` vers le cœur) |

## Ordre de lecture conseillé

1. **Ce README** — vue d'ensemble et référence des flux.
2. [`01_analyse_flux_zones.md`](01_analyse_flux_zones.md) — flux et zones en détail.
3. [`02_cartographies.md`](02_cartographies.md) — les schémas.

## Visualisation des diagrammes

Les diagrammes et les fichiers sont visibles depuis Github.



