[[Axe 4 - Disques]]
11/10/2022
****

**RAID (Redundant Array of Independent/Inexpensive Disks)** permet de répartir des données sur plusieurs disques en même temps.
	*Deux disques durs sont donc — au minimum — nécessaires*

L'objectif est d'effectuer des sauvegardes sûres. 
	*Il est donc recommandé d'utiliser des disques de marques différentes (au cas ou les disques ont un dysfonctionnement au même moment)*


**Les grappes apparaissant à la machine comme une unité logique de stockage unique.**

Avantages du RAID: 
- Tolérance aux pannes 
- Améliorer la sécurité des données (éviter les pertes de données) 
- Accroître les performances des machines


****
## Types de RAID

Les niveaux de RAID vont de RAID-0 à RAID-6. 

Les plus répendus: 
- **RAID-0: Disques entrelacés (Striping)**
	- Les données sont réparties uniformément entre plusieurs disques, ce qui augmente les performances, en particulier en lecture et écriture
    - Aucune redondance, données réparties sur plusieurs disques 
    - Pas de tolérances aux pannes (si un disque tombe en panne, toutes ses données sont perdues)
    - Insuffisant, souvent couplé à un autre niveau de RAID 
    
- **RAID-1: Disques en miroir (Mirroring)** 
    - Les données sont dupliquées sur deux ou plusieurs disques. Chaque disque contient une copie exacte des données
    - Redondance. Si un disque tombe en panne, les données sont toujours accessibles
    - Réduit l'espace de stockage d'au moins une moitié (données écrites en double)
    
- **RAID-5: Association de RAID-0 et d'un système de parité réparti sur plusieurs disques (Striping avec parité)**
    - Les données sont entrelacées (striping) sur plusieurs disques, et une parité est calculée et distribuée sur tous les disques. La parité permet de reconstruire les données en cas de défaillance d'un disque.
    - Redondance grâce à la parité
    - Performance en lecture et écriture. La perte d'espace — due à la parité — n'affecte qu'un seul disque
    - Reconstruction des données possible (Permet de manipuler les disques/fichiers à chaud), mais dégrade les performances 
    - Très utile pour les bases de données 
    
- **RAID-10 (RAID 1+0): Combinaison de RAID-1 et RAID-0**
    - Les disques sont d'abord configurés en miroirs (RAID-1), puis ces ensembles de disques en miroir sont ensuite entrelacés (RAID-0). Cela combine la redondance du RAID-1 avec les performances du RAID-0. 
    - Redondant, mais prend la moitié de l'espace pour la redondance (même problème que RAID-1)
    - Excellentes performances, RAID le plus rapide 
    - Très couteux, nécessite au moins 4 disques (dont la moitié sera inexploitable)
    - **Différence avec RAID-0+1** : RAID-10 est plus résilient aux pannes que RAID-0+1. RAID-10 peut tolérer la défaillance de plusieurs disques tant que chaque paire miroir reste intacte, alors que RAID-0+1 est plus vulnérable en cas de panne.


****
## Manage MD Devices (`mdadm`)

`mdadm` est l'utilitaire sous Linux qui permet de **créer, gérer, surveiller et réparer** des ensembles RAID logiciels (appelés **dispositifs MD**, pour "Multiple Device").

**Caractéristiques principales :**
- **Gestion de tous les types de RAID** : RAID-0, RAID-1, RAID-5, RAID-6, RAID-10, etc
- **Création et assemblage d’ensembles RAID** : On peut utiliser `mdadm` pour créer un ensemble RAID à partir de plusieurs disques ou partitions
- **Surveillance** : `mdadm` peut surveiller un ensemble RAID en fonctionnement et faire une alerte en cas de panne d'un disque ou d'un problème de synchronisation
- **Reconstruction** : Si un disque tombe en panne, `mdadm` peut reconstruire le RAID après le remplacement du disque défectueux


Les partitions RAID gérées par `mdadm` sont de type **Linux RAID autodetect** (`0xFD`), ce qui permet au système d’exploitation Linux de détecter automatiquement les partitions RAID au démarrage.


**Quelques commandes :**

| **Commande**       | **Description**                                                |
| ------------------ | -------------------------------------------------------------- |
| `mdadm --create`   | Créer un nouvel ensemble RAID                                  |
| `mdadm --assemble` | Assembler un ensemble RAID existant                            |
| `mdadm --add`      | Ajouter un disque à un ensemble RAID (ou un disque de secours) |
| `mdadm --remove`   | Retirer un disque d'un ensemble RAID                           |
| `mdadm --stop`     | Arrêter un ensemble RAID                                       |
| `mdadm --detail`   | Afficher les détails d'un ensemble RAID                        |
| `mdadm --monitor`  | Surveiller un ensemble RAID et envoyer des alertes             |

