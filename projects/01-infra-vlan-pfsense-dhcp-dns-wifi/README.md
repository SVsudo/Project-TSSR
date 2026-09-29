# Infrastructure virtualisée : VLAN, pfSense, DHCP, DNS, WiFi géré

## Contexte

Conception et déploiement complet d'une infrastructure réseau segmentée sous Hyper-V, avec pare-feu pfSense, commutateur niveau 3, borne WiFi gérée (VLAN par SSID) et services réseau (DHCP, DNS) intégrés à un domaine local.

## Architecture

- **Pare-feu pfSense** (WAN / LAN / DMZ) : filtrage, NAT (port forward), routage entre VLANs
- **Switch niveau 3** : routage inter-VLAN, trunks 802.1Q, gestion SSH sécurisée
- **4 VLANs** : Client, WiFi, DNS, Administration — chacun avec son propre sous-réseau et sa politique de pare-feu
- **Borne WiFi** : diffusion de 2 SSID mappés sur des VLANs distincts, VLAN de management dédié
- **Serveur Debian** : hébergement d'un site web interne et d'un site public (via port forward pfSense)
- **Serveur Windows** : service DNS, enregistrements des hôtes du domaine

## Compétences démontrées

- Segmentation réseau par VLAN (802.1Q) avec routage inter-VLAN sur switch niveau 3
- Configuration pfSense : interfaces, routes statiques, NAT/port forward, règles de pare-feu par interface (WAN/LAN/DMZ)
- Sécurisation de l'administration à distance (SSH, clé RSA, timeout d'inactivité, comptes nominatifs)
- Configuration d'un point d'accès WiFi professionnel (VLAN par SSID, VLAN de management)
- Attribution de VLAN au niveau de la virtualisation (Hyper-V : ID VLAN par carte réseau/VM)
- DHCP par VLAN avec DNS et domaine associés
- Enregistrements DNS et résolution de noms dans un domaine local
- Validation méthodique de bout en bout (ipconfig, connectivité, accès web par VLAN)

## Extraits de configuration

### Déclaration des VLANs et passerelles (switch niveau 3)

```
enable
configure terminal

ip routing

vlan 10
 name VLAN10-CLIENT
vlan 20
 name VLAN20-WIFI
vlan 50
 name VLAN50-DNS
vlan 90
 name VLAN90-ADMIN
exit

interface Vlan10
 ip address 192.168.10.1 255.255.255.0
 no shutdown
exit
interface Vlan20
 ip address 192.168.20.1 255.255.255.0
 no shutdown
exit
interface Vlan50
 ip address 192.168.50.1 255.255.255.248
 no shutdown
exit
interface Vlan90
 ip address 192.168.90.1 255.255.255.240
 no shutdown
exit
```

### Sécurisation de l'accès SSH

```
hostname svi
ip domain-name <domaine>.local

crypto key generate rsa
2048

ip ssh version 2
enable secret <mot_de_passe>
username admin privilege 15 secret <mot_de_passe>

line console 0
 login local
 exec-timeout 5 0
line vty 0 15
 login local
 transport input ssh
 exec-timeout 5 0
end
```

### Ports trunk (diffusion des VLANs)

```
interface fa1/0/5
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,50,90
 no shutdown
exit

interface fa1/0/10
 switchport trunk encapsulation dot1q
 switchport mode trunk
 switchport trunk allowed vlan 10,20,90
 no shutdown
exit
```

### Pool DHCP par VLAN

```
ip route 0.0.0.0 0.0.0.0 192.168.255.249

ip dhcp excluded-address 192.168.10.1 192.168.10.9
ip dhcp excluded-address 192.168.20.1 192.168.20.9

ip dhcp pool VLAN10-CLIENT
 network 192.168.10.0 255.255.255.0
 default-router 192.168.10.1
 dns-server 192.168.50.2
 domain-name <domaine>.local
 lease 7
exit
```

## Ressources associées

- `Réa_1.pkt` / `Réa.pkt` — maquettes Cisco Packet Tracer *(à ajouter manuellement, fichiers binaires non lisibles depuis le PDF source)*
