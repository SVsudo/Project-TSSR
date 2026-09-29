# Adressage IP et segmentation en sous-réseaux (VLSM)

## Contexte

Série d'exercices de calcul et de segmentation d'adresses IP : classement par classe, découpage en sous-réseaux de tailles fixes et de tailles variables (VLSM) à partir de contraintes métier (nombre d'hôtes par service).

## Compétences démontrées

- Classement d'adresses IP par classe (A/B/C/D/E)
- Segmentation en sous-réseaux de taille égale (ex. `/26` → 4 sous-réseaux)
- **VLSM** (Variable Length Subnet Masking) : segmentation à tailles variables selon des besoins réels en hôtes (25, 51, 120, 200 hôtes...)
- Calcul d'identifiant de sous-réseau, masque décimal/CIDR, adresse de broadcast et « pas » entre sous-réseaux
- Paramètres IP nécessaires à un poste client (adresse, masque, passerelle, DNS)
- Notions Wi-Fi (SSID, bandes 2,4 GHz vs 5 GHz)

## Exemple : VLSM sur une base `/23` avec 5 besoins hétérogènes

| Sous-réseau | Besoin | Plage | Masque | CIDR |
|---|---|---|---|---|
| 1 | 25 hôtes | 18.32.2.0 – 18.32.2.31 | 255.255.255.224 | /27 |
| 2 | 120 hôtes | 18.32.2.32 – 18.32.2.157 | 255.255.255.128 | /25 |
| 3 | 51 hôtes | 18.3.2.158 – 18.3.2.239 | 255.255.255.192 | /26 |
| 4 | 200 hôtes | 18.3.2.160 – 18.3.3.157 | 255.255.255.0 | /24 |
| 5 | 30 hôtes | 18.3.3.158 – 18.3.3.97 | 255.255.255.224 | /27 |

Cette logique d'allocation au plus juste (plutôt qu'un découpage uniforme) limite le gaspillage d'adresses IP, une pratique directement réutilisée dans les projets d'infrastructure plus avancés de ce portfolio.
