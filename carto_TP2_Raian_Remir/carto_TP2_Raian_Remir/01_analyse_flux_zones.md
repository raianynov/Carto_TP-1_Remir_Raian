# Flux réseau et zones de confiance — Air Defense System (ADS)

> Documentation des interactions du système ADS : flux entre composants, échanges entre
> zones, filtrage et interconnexions.
> Cartographies associées : [`02_cartographies.md`](02_cartographies.md). Architecture des
> actifs (composants, zones, adresses) : [`carto_TP1_Raian_Remir`](../carto_TP1_Raian_Remir/00_README.md).

---

## Flux principaux

Les communications du système se répartissent en trois familles.

### Flux opérationnels (vers le cœur `c4i-ads`)

La chaîne opérationnelle converge vers le `c4i-ads`, qui agrège et exploite les
informations reçues.

| Flux | Protocole / Port | Rôle |
|------|------------------|------|
| `radar-ads → c4i-ads` | TCP 7000 | Transmission des pistes radar détectées |
| `map-ads → c4i-ads` | UDP 8080 | Fourniture du fond cartographique |
| `mil-ads → c4i-ads` | TCP 5000 / 7000 | Remontée de la supervision militaire |
| `vpn-ads → c4i-ads` | TCP 5000 | Acheminement des flux distants sécurisés |

### Flux de commandement (depuis le cœur)

Après décision, le `c4i-ads` émet vers l'aval.

| Flux | Rôle |
|------|------|
| `c4i-ads → wpn-ads` | Ordre d'engagement de l'armement |
| `c4i-ads → systèmes de décision` | Transmission de la décision tactique |

Les ports de ces flux de commandement ne sont pas connus à ce stade et restent à
documenter.

### Flux administratifs

Accès de gestion, transverses au système : **SSH (port 22)** pour l'administration en ligne
de commande, **VNC (ports 5900–6011)** pour l'accès graphique distant. Les couples
source/destination ne sont pas spécifiés ; la vue correspondante retient une administration
depuis la zone INT, à confirmer.

---

## Flux critiques

Les flux dont dépend directement la mission de défense.

| Flux critique | Justification |
|---------------|---------------|
| `radar-ads → c4i-ads` (TCP 7000) | Sans pistes radar, plus aucune détection n'alimente la décision. Le flux traverse de plus une frontière sensible (BG non fiable → ADS critique). |
| `c4i-ads → wpn-ads` | Commande l'armement ; une altération a des conséquences physiques directes. |
| `vpn-ads → c4i-ads` (TCP 5000) | Point d'entrée des flux distants, canal à protéger. |

`radar-ads → c4i-ads` est le plus sensible : à la fois vital et inter-zone, de la zone la
moins fiable vers la plus critique.

---

## Relations entre zones

Trois zones de confiance reliées par un maillage de pare-feux :

- **ADS** (`10.10.9.0/24`, critique) derrière `fw-ads` ;
- **INT** (`10.10.8.0/24`, de confiance) derrière `fw-int` ;
- **BG** (`10.10.12.0/24`, non fiable) derrière `fw-bg`.

Tout flux inter-zone transite obligatoirement par les pare-feux concernés, qui filtrent,
autorisent ou bloquent selon des règles explicites. Aucune communication directe de zone à
zone n'est possible sans franchir cette barrière.

Deux relations sont particulièrement sensibles :

- **BG ↔ ADS** — relie la zone non fiable (terrain, radar) à la zone critique (cœur). Le
  radar doit y transmettre ses données : flux autorisé mais étroitement filtré.
- **VPN** — `vpn-ads` est le point d'entrée sécurisé vers le cœur ; il ne transporte que
  les flux strictement nécessaires, ce qui limite la surface exposée.

---

## Choix de représentation

Trois vues séparent les flux des zones.

- **Flux entre composants** — sens, protocoles et ports des flux opérationnels et de
  commandement. Vue centrale.
- **Flux administratifs** — accès SSH/VNC isolés, car transverses et hors chaîne
  opérationnelle ; les fusionner nuirait à la lisibilité.
- **Zones de confiance** — frontières, pare-feux, interconnexions et VPN, sans détail des
  flux applicatifs.

Les flux conservent leur sens (flèches orientées) ; les ports ne figurent que lorsqu'ils
sont connus. Les actifs, zones et adresses sont identiques à ceux de l'architecture des
actifs.

---

## Articulation avec l'architecture des actifs

Cette documentation des flux prolonge celle des actifs ([`carto_TP1_Raian_Remir`](../carto_TP1_Raian_Remir/00_README.md)) :
mêmes composants, mêmes zones, seules les interactions sont ajoutées.

- Les composants reliés par les flux (`c4i-ads`, `radar-ads`, `wpn-ads`…) figurent dans
  l'[inventaire des actifs](../carto_TP1_Raian_Remir/03_inventaire_actifs.md), avec les
  mêmes adresses.
- La vue des flux entre composants précise, avec protocoles et ports, la
  [chaîne opérationnelle](../carto_TP1_Raian_Remir/04_vues_metier_fonctionnelle.md)
  (détection → agrégation → décision → engagement).
- La vue des zones reprend la [segmentation en trois zones](../carto_TP1_Raian_Remir/02_cartographies.md)
  en y ajoutant frontières, interconnexions et VPN.
- Le radar, actif sensible placé en zone BG non fiable, se traduit ici par le flux
  `radar-ads → c4i-ads`, le plus critique du système.
