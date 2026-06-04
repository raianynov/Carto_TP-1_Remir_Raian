# Cartographies des flux et des zones — Air Defense System (ADS)

> Représentations visuelles accompagnant la documentation ([`01_analyse_flux_zones.md`](01_analyse_flux_zones.md)).
> Flux (vues 1 et 2) et zones de confiance (vue 3) sont présentés séparément.
> Sources éditables dans `diagrammes/`.

---

## Vue 1 — Flux réseau entre composants

**Objectif :** communications opérationnelles et de commandement, avec sens, protocoles et ports.
Source : [`diagrammes/vue_flux_composants.mmd`](diagrammes/vue_flux_composants.mmd)

```mermaid
%% Vue FLUX : communications entre composants, sens, protocoles et ports.
%% Source : flux operationnels et administratifs du systeme ADS.
flowchart LR

    RADAR["radar-ads<br/>(BG) 10.10.12.33"]
    MAP["map-ads<br/>(ADS) 10.10.9.34"]
    MIL["mil-ads<br/>(ADS) 10.10.9.31"]
    VPN["vpn-ads<br/>(ADS) 10.10.9.36"]
    C4I["c4i-ads<br/>(ADS) 10.10.9.32<br/>COEUR"]
    WPN["wpn-ads<br/>(ADS) 10.10.9.35"]
    DEC["Systemes de<br/>decision"]

    %% Flux operationnels vers le coeur
    RADAR -->|"TCP 7000"| C4I
    MAP -->|"UDP 8080"| C4I
    MIL -->|"TCP 5000 / 7000"| C4I
    VPN -->|"TCP 5000"| C4I

    %% Flux de commandement depuis le coeur
    C4I -->|"commande"| WPN
    C4I -->|"decision"| DEC

    classDef coeur fill:#7f1d1d,stroke:#fca5a5,color:#fff,stroke-width:2px;
    classDef src fill:#374151,stroke:#9ca3af,color:#fff;
    class C4I coeur;
    class RADAR,MAP,MIL,VPN,WPN,DEC src;
```

---

## Vue 2 — Flux administratifs

**Objectif :** accès de gestion (SSH, VNC), de nature transverse, isolés pour la lisibilité.
Source : [`diagrammes/vue_flux_admin.mmd`](diagrammes/vue_flux_admin.mmd)

```mermaid
%% Vue FLUX ADMINISTRATIFS : acces de gestion (SSH, VNC).
%% Ces flux sont transverses et concernent l'administration des composants.
flowchart LR

    ADMIN["Administrateurs<br/>(zone INT)"]

    subgraph CIBLES["Composants administres"]
        direction TB
        C1["Composants ADS<br/>(c4i, mil, map, wpn, vpn)"]
        C2["Composants BG<br/>(radar, bg-gsc)"]
        C3["Serveurs INT<br/>(infra, SIEM)"]
    end

    ADMIN -->|"SSH 22"| C1
    ADMIN -->|"SSH 22"| C2
    ADMIN -->|"SSH 22"| C3
    ADMIN -->|"VNC 5900-6011"| C1
    ADMIN -->|"VNC 5900-6011"| C3

    classDef adm fill:#1e3a5f,stroke:#93c5fd,color:#fff;
    classDef cib fill:#374151,stroke:#9ca3af,color:#fff;
    class ADMIN adm;
    class C1,C2,C3 cib;
```

---

## Vue 3 — Zones de confiance et interconnexions

**Objectif :** zones, frontières, pare-feux, points d'interconnexion et accès VPN.
Source : [`diagrammes/vue_zones_confiance.mmd`](diagrammes/vue_zones_confiance.mmd)

```mermaid
%% Vue ZONES : zones de confiance, frontieres, pare-feux, interconnexions, VPN.
%% Tout flux inter-zone transite par les pare-feux (maillage de filtrage).
flowchart TB

    subgraph BG["Zone BG - Border Guard (10.10.12.0/24) | NON FIABLE"]
        RADAR["radar-ads"]
        BGGSC["bg-gsc"]
        FWBG{{"fw-bg<br/>100.97.10.242"}}
    end

    subgraph INT["Zone INT - SI interne (10.10.8.0/24) | DE CONFIANCE"]
        SRVINT["Infra + SIEM + postes"]
        FWINT{{"fw-int<br/>100.97.10.115"}}
    end

    subgraph ADS["Zone ADS - Systeme de defense (10.10.9.0/24) | CRITIQUE"]
        C4I["c4i-ads (coeur)"]
        VPNADS["vpn-ads"]
        FWADS{{"fw-ads<br/>100.97.10.114"}}
    end

    %% Interconnexions filtrees entre pare-feux (maillage)
    FWBG ==>|"flux filtres"| FWADS
    FWINT ==>|"flux filtres"| FWADS
    FWBG ==>|"flux filtres"| FWINT

    %% Acces VPN : point d'entree securise vers le coeur
    VPNADS -.tunnel VPN.-> C4I

    %% Le radar (BG) communique avec le coeur (ADS) a travers les pare-feux
    RADAR -.via fw-bg / fw-ads.-> C4I

    classDef fw fill:#000,stroke:#f87171,color:#fff,stroke-width:2px;
    classDef untrust fill:#3f2d1e,stroke:#fcd34d,color:#fff;
    classDef trust fill:#1e3a5f,stroke:#93c5fd,color:#fff;
    classDef crit fill:#7f1d1d,stroke:#fca5a5,color:#fff;
    class FWBG,FWINT,FWADS fw;
```

---

## Note de lecture

- **Flèches pleines** = flux ou interconnexions ; **étiquettes** = protocole/port quand connu.
- **Pointillés** = liens spécifiques (tunnel VPN, flux inter-zone passant par les pare-feux).
- **Hexagones** = pare-feux ; **rouge** = cœur `c4i-ads`.
- Les ports ne sont indiqués que lorsqu'ils sont fournis par le contexte.
