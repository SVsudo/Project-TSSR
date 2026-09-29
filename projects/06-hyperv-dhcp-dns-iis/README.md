# Déploiement de services Windows Server sous Hyper-V (DNS, DHCP, IIS)

## Contexte

Mise en place et validation méthodique d'une petite infrastructure virtualisée sous Hyper-V : un serveur DNS/Web, un serveur DHCP et un poste client, tous en réseau interne.

## Compétences démontrées

- Création et configuration de machines virtuelles Hyper-V (réseau interne, ressources)
- Installation et configuration du rôle **DNS** (zone de recherche directe, enregistrements A)
- Installation et configuration du rôle **DHCP** (étendue, pool d'adresses, réservations par adresse MAC, options d'étendue)
- Hébergement de deux sites web distincts sous **IIS** (ports différenciés) avec résolution DNS dédiée
- Diagnostic réseau méthodique : `ipconfig /all`, gestion des disques, tests de connectivité croisés entre serveurs et client
- Documentation rigoureuse par capture d'écran à chaque étape, réflexe de validation avant livraison

## Démarche de validation (exemple de méthode appliquée)

1. Vérification de la configuration réseau de chaque VM au niveau de l'hyperviseur
2. Vérification des paramètres système (nom, RAM, disques) directement depuis l'OS
3. Tests `ipconfig /all` sur chaque machine
4. Tests de ping croisés entre toutes les machines (serveur ↔ serveur, serveur ↔ client, machine physique ↔ VMs)
5. Vérification des enregistrements DNS puis résolution effective de chaque nom déclaré
6. Test d'accès aux deux sites web, en local puis depuis le client
7. Vérification du bail DHCP, y compris avec réservation d'adresse par MAC

Cette approche — configurer puis systématiquement valider à chaque couche (hyperviseur → OS → réseau → service applicatif) — reflète une méthode de travail transposable à des infrastructures de production.
