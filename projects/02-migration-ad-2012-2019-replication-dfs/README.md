# Migration Active Directory & haute disponibilité (2012 → 2019)

## Contexte

Étude de cas pour une ESN fictive : mise en place initiale d'un annuaire Active Directory par script, puis migration de l'infrastructure vers une version plus récente de Windows Server sans interruption de service, et enfin mise en place de la haute disponibilité (réplication AD multi-maître + réplication DFS du serveur de fichiers).

## Démarche en 4 étapes

### 1. Initialisation de la structure (scripts batch)
Création automatisée des OU par service, des groupes, des utilisateurs et des dossiers personnels/partagés à partir d'un fichier CSV, avec droits NTFS appliqués par script (`dsadd`, `dsmod`, `icacls`).

### 2. Migration vers Windows Server 2019
- Mise à jour manuelle du schéma AD (`adprep /forestprep`, `/domainprep`, `/rodcprep`) plutôt qu'automatique, pour isoler et contrôler chaque étape sensible
- Promotion du nouveau serveur comme second contrôleur de domaine (jamais de bascule directe)
- Migration des données via **Robocopy** (`/COPYALL /SEC`) pour préserver intégralement les permissions NTFS
- Refonte des droits selon la méthode **AGDLP** (Account → Global Group → Domain Local Group → Permission)
- Décommissionnement de l'ancien serveur une fois la réplication validée

### 3. Réplication du contrôleur de domaine
Ajout d'un second contrôleur de domaine en parallèle, vérification de la réplication (`repadmin /replsummary`), test de bascule côté client (`nltest /dsgetdc`).

### 4. Réplication du serveur de fichiers (DFS)
Mise en place d'un espace de noms DFS avec réplication DFS-R continue entre deux serveurs de fichiers, testée par extinction volontaire du serveur principal.

## Compétences démontrées

- Scripting d'annuaire (création en masse d'utilisateurs/groupes/droits depuis une source de données)
- Migration de schéma AD et gestion des rôles FSMO
- Méthode AGDLP pour une gestion des droits évolutive
- Réplication multi-maître Active Directory et réplication DFS-R
- Conception pour la continuité de service (élimination des points de défaillance unique)
- Justification argumentée de chaque choix technique (voir section *Synthèse* ci-dessous)

## Synthèse des choix techniques

| Choix | Justification |
|---|---|
| Migration progressive (ajout d'un second DC) plutôt qu'une bascule directe | Élimine le risque de perte totale de service en cas d'échec ; retour arrière possible tant que l'ancien serveur n'est pas rétrogradé |
| Mise à jour manuelle du schéma via `adprep` | Permet de contrôler et vérifier une opération irréversible indépendamment de la promotion |
| Migration des données via Robocopy plutôt qu'une copie standard | Préserve les permissions NTFS et évite une reconfiguration manuelle source d'erreurs |
| Réplication AD + DFS-R | Élimine le SPOF (single point of failure) sur l'authentification et l'accès aux fichiers |
| Espace de noms DFS plutôt que chemins UNC directs | Bascule transparente pour le client en cas de panne d'un serveur nommé |

## Pistes d'amélioration identifiées

- Réplication des serveurs sur un second site géographique
- Sauvegarde selon la méthode 3-2-1
- Réécriture des scripts batch en PowerShell
- Étude de faisabilité d'une migration vers le Cloud
