# Analyse des flux et des zones de confiance — Système ADS (TP2)

> **TP2 — Cartographie des flux et des zones de confiance**
> Document d'analyse écrite. Les cartographies associées sont dans
> [`02_cartographies.md`](02_cartographies.md), sources Mermaid dans `diagrammes/`.
> Ce TP complète le TP1 (cartographie des actifs) en se concentrant sur les **interactions**
> entre composants et zones.

---

## 1. Explication des principaux flux

Les flux du système se répartissent en trois familles.

### 1.1 Flux opérationnels (vers le cœur `c4i-ads`)

Toute la chaîne opérationnelle converge vers le `c4i-ads`, qui agrège et exploite les
informations reçues.

| Flux | Protocole / Port | Rôle |
|------|------------------|------|
| `radar-ads → c4i-ads` | TCP 7000 | Transmission des pistes radar détectées |
| `map-ads → c4i-ads` | UDP 8080 | Fourniture du fond cartographique |
| `mil-ads → c4i-ads` | TCP 5000 / 7000 | Remontée de la supervision militaire |
| `vpn-ads → c4i-ads` | TCP 5000 | Acheminement des flux distants sécurisés |

### 1.2 Flux de commandement (depuis le cœur)

Une fois la décision prise, le `c4i-ads` émet vers l'aval :

| Flux | Rôle |
|------|------|
| `c4i-ads → wpn-ads` | Ordre d'engagement de l'armement |
| `c4i-ads → systèmes de décision` | Transmission de la décision tactique |

> Le contexte ne précise pas les ports de ces flux de commandement ; ils sont donc
> représentés sans numéro de port, par fidélité à la source.

### 1.3 Flux administratifs

Les accès de gestion sont transverses : **SSH (port 22)** pour l'administration en ligne de
commande et **VNC (ports 5900–6011)** pour l'accès graphique distant. Le contexte liste ces
protocoles sans détailler les couples source/destination ; la vue correspondante propose une
organisation plausible (administration depuis la zone INT), à considérer comme une
hypothèse de représentation.

---

## 2. Identification des flux critiques

Les flux critiques sont ceux dont dépend directement la mission de défense :

| Flux critique | Pourquoi |
|---------------|----------|
| `radar-ads → c4i-ads` (TCP 7000) | Sans les pistes radar, plus aucune détection n'alimente la décision. De plus, ce flux traverse une frontière sensible (BG non fiable → ADS critique). |
| `c4i-ads → wpn-ads` | Commande l'armement : un flux altéré a des conséquences physiques directes. |
| `vpn-ads → c4i-ads` (TCP 5000) | Point d'entrée des flux distants ; canal sensible à protéger. |

Le flux `radar-ads → c4i-ads` est le plus sensible des trois : il est à la fois vital pour
la mission **et** inter-zone (depuis la zone la moins fiable vers la plus critique).

---

## 3. Analyse des relations entre zones

Le système repose sur **trois zones de confiance** (reprises du TP1) reliées par un
**maillage de pare-feux** :

- **ADS** (`10.10.9.0/24`, critique) derrière `fw-ads` ;
- **INT** (`10.10.8.0/24`, de confiance) derrière `fw-int` ;
- **BG** (`10.10.12.0/24`, non fiable) derrière `fw-bg`.

Le principe directeur est que **tout flux inter-zone transite obligatoirement par les
pare-feux concernés**, qui filtrent, autorisent ou bloquent selon des règles explicites.
Aucune communication directe de zone à zone n'est possible sans passer par cette barrière.

Deux relations méritent attention :

- **BG ↔ ADS** : c'est la relation la plus sensible, car elle relie la zone non fiable
  (terrain, radar) à la zone critique (cœur opérationnel). Le radar doit pourtant y
  transmettre ses données : ce flux est donc autorisé mais étroitement filtré.
- **VPN** : `vpn-ads` constitue un point d'entrée sécurisé vers le cœur. Le VPN ne
  transporte que les flux strictement nécessaires, ce qui limite la surface exposée.

---

## 4. Justification des choix de représentation

Trois vues distinctes ont été produites, conformément à la séparation demandée entre flux
et zones :

- **Vue des flux entre composants** : montre le sens, les protocoles et les ports des flux
  opérationnels et de commandement. C'est la vue centrale du TP2.
- **Vue des flux administratifs** : isolée car les accès SSH/VNC sont de nature transverse
  et ne participent pas à la chaîne opérationnelle ; les mélanger nuirait à la lisibilité.
- **Vue des zones de confiance** : représente les frontières, les pare-feux, les
  interconnexions et le VPN, sans détailler chaque flux applicatif — elle répond au second
  attendu du TP.

Choix transverses : les flux conservent le **sens** indiqué par le contexte (flèches
orientées) ; les **ports** ne sont indiqués que lorsqu'ils sont connus ; et la cohérence
avec le TP1 est respectée (mêmes noms d'actifs, mêmes zones, mêmes adresses).

---

## 5. Cohérence avec le TP1

Ce TP2 prolonge directement le TP1 : les actifs et zones sont identiques, seules les
**interactions** sont ajoutées. Concrètement :

- Les **composants** reliés par les flux (`c4i-ads`, `radar-ads`, `wpn-ads`…) sont ceux
  recensés dans l'[**inventaire des actifs**](../carto_TP1_Raian_Remir/03_inventaire_actifs.md)
  du TP1, avec les mêmes adresses IP.
- La **vue des flux entre composants** précise, avec protocoles et ports, la **chaîne
  opérationnelle** déjà décrite dans la [**vue fonctionnelle**](../carto_TP1_Raian_Remir/04_vues_metier_fonctionnelle.md)
  du TP1 : détection → agrégation → décision → engagement.
  Le TP1 disait *quoi* (les fonctions) ; le TP2 dit *comment* (les flux concrets).
- La **vue des zones de confiance** reprend la **segmentation en trois zones** de la
  [vue logique](../carto_TP1_Raian_Remir/02_cartographies.md) du TP1, en y ajoutant les
  frontières, les points d'interconnexion et le VPN.
- Le **point d'attention** identifié au TP1 — le radar (actif vital) placé en zone BG non
  fiable — trouve ici sa traduction en flux : `radar-ads → c4i-ads` est le flux le plus
  sensible, car vital **et** inter-zone (de la zone la moins fiable vers la plus critique).

Ainsi, les deux TP forment un ensemble cohérent : le TP1 cartographie *ce qui existe et où*,
le TP2 cartographie *comment cela communique*.
