# Cartographies des actifs — Système ADS

> Représentations visuelles accompagnant l'analyse ([`01_analyse.md`](01_analyse.md)).
> Trois vues complémentaires, toutes en **Mermaid** (sources éditables dans `diagrammes/`).
>
> Voir aussi : [`00_README.md`](00_README.md) (vue d'ensemble et glossaire) et [`03_inventaire_actifs.md`](03_inventaire_actifs.md) (inventaire détaillé).

---

## Vue 1 — Vue logique par zones

**Objectif :** organisation des actifs par zone (ADS / INT / BG) et regroupement logique (infrastructure, SIEM, postes).
Source : [`diagrammes/vue_logique_zones.mmd`](diagrammes/vue_logique_zones.mmd)

```mermaid
%% Cartographie des ACTIFS uniquement (TP1) - les flux relevent du TP2.
flowchart TB

    subgraph ADS["Zone ADS - Systeme de defense (10.10.9.0/24) | CRITIQUE"]
        direction TB
        FWADS["fw-ads<br/>100.97.10.114<br/>(pare-feu)"]
        C4I["c4i-ads<br/>10.10.9.32<br/>Centre C4I - COEUR"]
        WPN["wpn-ads<br/>10.10.9.35<br/>Armement"]
        MIL["mil-ads<br/>10.10.9.31<br/>Supervision (RO)"]
        MAP["map-ads<br/>10.10.9.34<br/>Cartographie"]
        VPNADS["vpn-ads<br/>10.10.9.36<br/>Entree VPN"]
        USRADS["Utilisateurs ADS<br/>10.10.9.250-254"]
    end

    subgraph INT["Zone INT - SI interne (10.10.8.0/24) | DE CONFIANCE"]
        direction TB
        FWINT["fw-int<br/>100.97.10.115<br/>(pare-feu)"]
        subgraph INFRA["Infrastructure"]
            DC1["dc1 - AD"]
            DC2["dc2 - AD"]
            FILES["files"]
            MAIL["mail"]
            SQL["sql"]
            WSUS["wsus"]
            REPORTS["reports"]
            HQGSC["hq-gsc"]
        end
        subgraph SIEM["SIEM - Supervision securite"]
            SIEMD["siem-dashboard<br/>10.10.8.70"]
            SIEMI["siem-indexer<br/>10.10.8.71"]
            SIEMS["siem-server<br/>10.10.8.72"]
        end
        subgraph WS["Postes utilisateurs"]
            WS1["ws1<br/>10.10.8.31-35"]
            WS2["ws2<br/>10.10.8.36-40"]
            WS3["ws3<br/>10.10.8.41-45"]
        end
    end

    subgraph BG["Zone BG - Border Guard (10.10.12.0/24) | NON FIABLE"]
        direction TB
        FWBG["fw-bg<br/>100.97.10.242<br/>(pare-feu)"]
        RADAR["radar-ads<br/>10.10.12.33<br/>Radar detection"]
        BGGSC["bg-gsc<br/>10.10.12.64<br/>Controle local"]
        USRBG["Utilisateurs BG<br/>10.10.12.250-254"]
    end

    %% Maillage des pare-feux (interconnexion, sans detail de flux)
    FWADS -.maillage filtre.- FWINT
    FWINT -.maillage filtre.- FWBG
    FWADS -.maillage filtre.- FWBG

    classDef fw fill:#000,stroke:#f87171,color:#fff,stroke-width:2px;
    class FWADS,FWINT,FWBG fw;
```

## Vue 2 — Vue inventaire par catégorie d'actif

**Objectif :** classification transversale par type (serveurs, réseau, utilisateurs, services), indépendamment des zones.
Source : [`diagrammes/vue_inventaire_categories.mmd`](diagrammes/vue_inventaire_categories.mmd)

```mermaid
%% Classification transversale des actifs par TYPE (independamment des zones).
flowchart LR

    ROOT["SI - Air Defense System<br/>(ADS)"]

    ROOT --> SRV["SERVEURS"]
    ROOT --> NET["EQUIPEMENTS RESEAU"]
    ROOT --> USR["POSTES UTILISATEURS"]
    ROOT --> SVC["SERVICES CRITIQUES"]

    %% --- Serveurs ---
    SRV --> SRV_ADS["Zone ADS<br/>c4i-ads, mil-ads,<br/>map-ads, wpn-ads"]
    SRV --> SRV_INT["Zone INT<br/>dc1, dc2, files, mail, sql,<br/>wsus, reports, hq-gsc<br/>+ siem x3"]
    SRV --> SRV_BG["Zone BG<br/>radar-ads, bg-gsc"]

    %% --- Reseau ---
    NET --> FW["Pare-feux<br/>fw-ads, fw-int, fw-bg"]
    NET --> VPN["VPN<br/>vpn-ads (10.10.9.36)"]

    %% --- Utilisateurs ---
    USR --> U_ADS["Utilisateurs ADS<br/>10.10.9.250-254"]
    USR --> U_INT["ws1 / ws2 / ws3<br/>10.10.8.31-45"]
    USR --> U_BG["Utilisateurs BG<br/>10.10.12.250-254"]

    %% --- Services critiques ---
    SVC --> SVC1["Commandement C4I<br/>(c4i-ads)"]
    SVC --> SVC2["Armement<br/>(wpn-ads)"]
    SVC --> SVC3["Detection radar<br/>(radar-ads)"]
    SVC --> SVC4["Liaison VPN<br/>radar - armement<br/>(vpn-ads)"]
```

## Vue 3 — Vue physique / réseau

**Objectif :** rattachement de chaque actif à son réseau et à son pare-feu de zone (lecture « infrastructure »).
Source : [`diagrammes/vue_physique.mmd`](diagrammes/vue_physique.mmd)

```mermaid
%% Vue physique : rattachement des actifs a leur reseau et au pare-feu de zone.
%% Cartographie des ACTIFS uniquement (TP1) - les flux relevent du TP2.
%% Ordre des zones : BG -> INT -> ADS.
flowchart LR

    subgraph BG["Zone BG - 10.10.12.0/24 (NON FIABLE)"]
        direction TB
        FWBG{{"fw-bg<br/>100.97.10.242"}}
        SWBG["reseau 10.10.12.0/24"]
        FWBG --- SWBG
        SWBG --- RADAR["radar-ads<br/>10.10.12.33<br/>Radar de detection"]
        SWBG --- BGGSC["bg-gsc<br/>10.10.12.64<br/>Controle local"]
        SWBG --- UBG["Utilisateurs BG<br/>10.10.12.250-254"]
    end

    subgraph INT["Zone INT - 10.10.8.0/24 (DE CONFIANCE)"]
        direction TB
        FWINT{{"fw-int<br/>100.97.10.115"}}
        SWINT["reseau 10.10.8.0/24"]
        FWINT --- SWINT
        SWINT --- INFRA["Infrastructure<br/>dc1, dc2, files, mail,<br/>sql, wsus, reports, hq-gsc<br/>(IP non fournies)"]
        SWINT --- SIEM["SIEM<br/>siem-dashboard .70<br/>siem-indexer .71<br/>siem-server .72"]
        SWINT --- WS["Postes ws1/ws2/ws3<br/>10.10.8.31-45"]
    end

    subgraph ADS["Zone ADS - 10.10.9.0/24 (CRITIQUE)"]
        direction TB
        FWADS{{"fw-ads<br/>100.97.10.114"}}
        SWADS["reseau 10.10.9.0/24"]
        FWADS --- SWADS
        SWADS --- C4I["c4i-ads<br/>10.10.9.32<br/>Centre C4I (COEUR)"]
        SWADS --- WPN["wpn-ads<br/>10.10.9.35<br/>Armement"]
        SWADS --- MIL["mil-ads<br/>10.10.9.31<br/>Supervision (RO)"]
        SWADS --- MAP["map-ads<br/>10.10.9.34<br/>Cartographie"]
        SWADS --- VPN["vpn-ads<br/>10.10.9.36<br/>Entree VPN"]
        SWADS --- UADS["Utilisateurs ADS<br/>10.10.9.250-254"]
    end

    %% Maillage des pare-feux (chaine lineaire pour conserver l'alignement)
    FWBG -.maillage filtre.- FWINT
    FWINT -.maillage filtre.- FWADS

    classDef fw fill:#000,stroke:#f87171,color:#fff,stroke-width:2px;
    classDef net fill:#374151,stroke:#9ca3af,color:#fff;
    class FWBG,FWINT,FWADS fw;
    class SWBG,SWINT,SWADS net;
```

---

## Note de lecture

- **Code couleur** : noir = pare-feu · gris = segment réseau. Les autres actifs ne sont pas colorés.
- **Niveau de confiance des zones** : indiqué dans le titre de chaque zone (CRITIQUE / DE CONFIANCE / NON FIABLE), tel que désigné par le contexte.
- **Services critiques** (vue inventaire) : commandement C4I, armement, détection radar et liaison VPN radar–armement.
- **Hexagones** `{{ }}` (vue physique) = pare-feux ; **pointillés** = maillage entre pare-feux (élément structurel ; détail des flux en TP2).
- Aucune adresse IP n'a été inventée : les serveurs INT sans IP fournie sont indiqués comme tels.
