# Étude de cas : conception réseau segmentée (VLAN, ROAS, NAT) — Cisco

## Contexte

Une entreprise (bureau d'ingénierie) anticipe un agrandissement et demande une refonte de son infrastructure réseau : segmentation par service, routage inter-VLAN sécurisé, accès distant chiffré et accès contrôlé depuis Internet vers un serveur web interne.

## Conception

### Adressage
Plan d'adressage calculé sur mesure à partir d'une plage `/16`, avec sous-réseaux dimensionnés au plus juste par service (33 employés Ingénieur, 30 Conception, 7 Responsable, 5 Direction, 3 Administrateurs, 10 Serveurs), IDSR/masque/passerelle/broadcast fournis pour chacun.

### VLANs et routage
6 VLANs (Conception, Ingénieur, Direction, Responsable, Administrateur, Serveurs), routés via **ROAS (Router-on-a-Stick)** avec sous-interfaces `dot1Q` sur le routeur central, et VLAN natif dédié (`999`) pour désactiver les ports inutilisés et limiter les sauts inter-VLAN malveillants.

### NAT et accès externe
NAT overload (PAT) pour la sortie Internet de tous les VLANs, et NAT statique pour exposer un serveur web interne sur Internet via une ACL entrante dédiée.

## Compétences démontrées

- Conception d'un plan d'adressage IP optimisé par contrainte d'effectifs
- VLAN 802.1Q, VLAN natif de sécurité, ports en mode trunk avec restriction explicite des VLANs autorisés
- Routage inter-VLAN par sous-interfaces (ROAS) avec `ip helper-address` pour le relais DHCP
- NAT/PAT et NAT statique avec ACL entrante ciblée
- Sécurisation de l'administration à distance (SSH, clé RSA)
- Répond en anglais à des questions techniques (anglais professionnel courant)

## Extrait : configuration du switch central

```
hostname Sw-Core
enable secret <mot_de_passe>
service password-encryption
no ip domain-lookup
ip domain-name lan.local
crypto key generate rsa
2048
ip ssh version 2
line vty 0 4
 login local
 transport input ssh
line console 0
 login local

vlan 2
 name Conception
vlan 3
 name Ingenieur
vlan 4
 name Direction
vlan 5
 name Responsable
vlan 10
 name Serveurs
vlan 90
 name Administrateur
vlan 999
 name BLACKHOLE

interface range fa0/3-24
 switchport mode access
 switchport access vlan 999
 shutdown
```

## Extrait : routage inter-VLAN et NAT (routeur)

```
interface g0/0.2
 encapsulation dot1Q 2
 ip address 172.12.0.65 255.255.255.224
 ip helper-address 172.12.0.115
 ip nat inside

interface g0/1
 description Internet
 ip address 45.35.25.100 255.255.255.0
 ip nat outside

access-list 1 permit 172.12.0.0 0.0.0.255
ip nat inside source list 1 interface g0/1 overload
ip nat inside source static tcp 172.12.0.116 80 45.35.25.1 80

ip access-list extended ACL_WAN_IN
 permit tcp any host 45.35.25.1 eq 80
interface g0/1
 ip access-group ACL_WAN_IN in
```

## Ressources associées

- `Réalisation_réseaux_3.pdf` — questions théoriques associées (couche OSI, NAT, VLAN natif) et tableau de correspondance VLAN/ports
