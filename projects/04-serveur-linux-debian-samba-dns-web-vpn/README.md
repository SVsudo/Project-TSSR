# Infrastructure Linux : Samba, DNS, Apache2, VPN IPSEC, sauvegardes automatisées

## Contexte

Déploiement d'une infrastructure hétérogène (Linux/Windows) reliant deux sites via un tunnel VPN site-à-site, avec partage de fichiers, résolution DNS, hébergement web et sauvegardes planifiées sous Debian 13.

## Architecture

- **2 pare-feu pfSense** reliés par un tunnel **IPSEC site-à-site (S2S)**
- **Site A** : serveur de fichiers Debian (Samba) + client Windows
- **Site B** : serveur Debian (Bind9 pour le DNS + Apache2 pour le web) + serveur Windows AD

## Compétences démontrées

- Configuration réseau Debian (statique, `/etc/network/interfaces`)
- Partage de fichiers Samba avec droits différenciés par groupe (méthode d'héritage NTFS-like sous Linux via `create mask` / `directory mask` / `force group`)
- Serveur DNS Bind9 : zones directes et inverses, enregistrements A et CNAME
- Serveur web Apache2 avec accès restreint (VPN uniquement)
- **Tunnel IPSEC site-à-site** entre deux pare-feu pfSense (choisi plutôt qu'OpenVPN pour la robustesse native du protocole)
- Scripting Bash de sauvegarde automatisée (`tar`) avec planification **cron**
- Diagnostic par les outils standards (`ip a`, `ip route`, `systemctl`, `ss -ti`)

## Choix d'architecture (justifiés dans le TP)

- Tunnel **IPSEC** plutôt qu'OpenVPN pour la robustesse et la standardisation
- **Samba** sur Debian pour la simplicité de mise en place et d'administration d'un partage de fichiers en environnement mixte
- **Apache2** comme serveur web : solution fiable, stable et open source
- Service **DNS** colocalisé sur le serveur web pour la pratique du DNS sous Linux

## Extrait : script de sauvegarde automatisée

```bash
#!/bin/bash

# Date et heure
DATE=$(date +"%Y-%m-%d_%H-%M-%S")

# Dossier de destination
DEST="/backup"

# Dossiers à sauvegarder
SOURCE="/home /etc /var/www"

# Nom de l'archive
ARCHIVE="$DEST/sauvegarde_$DATE.tar.gz"

# Création de la sauvegarde
tar -czf "$ARCHIVE" $SOURCE

# Message de fin
echo "Sauvegarde réalisée : $ARCHIVE"
```

Planification via `crontab` :
```
0 2 * * * /usr/local/bin/sauvegarde.sh >> /var/log/sauvegarde.log 2>&1
```

## Extrait : partage Samba avec droits par service

```ini
[commun]
    comment = Partage commun
    create mask = 0664
    directory mask = 02775
    force group = partage
    path = /srv/samba/commun
    read only = No
    valid users = @partage @direction

[direction]
    browseable = No
    comment = Partage direction (restreint)
    create mask = 0660
    directory mask = 02770
    path = /srv/samba/direction
    read only = No
    valid users = @direction
```
