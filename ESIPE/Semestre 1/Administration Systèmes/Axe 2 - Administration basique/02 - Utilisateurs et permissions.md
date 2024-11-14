[[Axe 2 - Administration basique]]
04/10/2022
****
## Permissions

Linux étant un système multi-utilisateur, il est normal que tout le monde — en dehors de l'administrateur **(root)** et des **sudoers** (ayant la possibilité de lancer une commande avec "sudo") — ne puisse pas tout faire.

Cela force la présence d'un **système de privilèges** au sein de l'OS. 

Commandes générales :

| Commande             | Objectif                                         |
| -------------------- | ------------------------------------------------ |
| `who`                | Qui est actuellement connecté                    |
| `passwd`             | Changement du mot de passe                       |
| `adduser`, `uderadd` | Ajouter des utilisateurs                         |
| `deluser`, `userdel` | Supprimer des utilisateurs                       |
| `addgroup`           | Ajout d'un groupe                                |
| `usermod`            | Modifier un compte utilisateur                   |
| `chown`              | Change le propriétaire d'un fichier/répertoire   |
| `chgrp`              | Change le propriétaire du groupe                 |
| `chmod`              | Change les permissions sur un fichier/répertoire |

Note : `adduser`, contrairement à `useradd`, va automatiquement créer un **home** pour le nouvel utilisateur. Useradd ne fait qu'ajouter l'utilisateur.
	*`useradd` est donc mieux pour créer des comptes de service, la ou `adduser` est plus approprié pour ajouter des comptes utilisateurs classiques*


Un fichier/dossier ne peut faire partie que d'un seul groupe. 
La commande **chmod** permet de modifier les droits d'un utilisateur sur un fichier ou un répertoire. 

**Méthode 1 :** 
`chmod o+x fichier.py`

**Méthode 2 :**
`chmod 760 fichier.py`


**Guide :**
![[chmod.png]]
