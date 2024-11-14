[[Axe 4 - Disques]]
04/10/2022
****
**Table of Contents**
```table-of-contents
```

****
## Gestion des partitions 

La mémoire de masse est un espace physique (palpable et concret).
A l'inverse, une partition est un **découpage virtuel de cet espace physique**. C'est un **espace logique**. 

La mise en place de partitions a plusieurs objectifs: 
- Organisation des disques 
- Installer plusieurs OS (Dual Boot) 
- Partitions de sauvegardes
- Plusieurs File System (ext4fs, Chiffrement avec LUKS…) 


La commande principale permettant de gérer les partitions est `fdisk`. Cette commande permet de manipuler la **table de partitions**. 

En général, le répertoire `/dev/sda` désigne le premier disque physique, `/dev/sdb` désigne le second disque, ainsi de suite. 

La création de partitions — pour sda — suivra cette logique : `/dev/sda1`, `/dev/sda2`… `/dev/sdan` 


Le logiciel `gparted` permet de fournir un GUI à la commande `fdisk` (et étend ses fonctionnalités).


Il existe deux principaux types de partitions :
- **Partition primaire** - C’est une partition qui contient directement les données ou le système d’exploitation.
    *Un disque peut avoir jusqu’à **4 partitions primaires** (selon le type de table de partition).*

- **Partition logique** - Utilisée dans une **partition étendue**, elle permet de contourner la limite des 4 partitions primaires. On peut créer plusieurs partitions logiques à l'intérieur d'une partition étendue.
    *Nécessaire dans les systèmes plus anciens comme MBR qui limite le nombre de partitions primaires.*


****
## Boot

L'OS est situé sur le support physique, au sein d'une partition.
Un disque dur contient une table de partitionnement 
- **MBR** était le premier logiciel permettant cela
- **GPT (GUID Partition Table) est l'évolution de MBR**. Il fait partie du système UEFI (successeur du BIOS)

**Systèmes de partitionnement :**

| **Caractéristique**            | **MBR**              | **GPT**                                               |
| ------------------------------ | -------------------- | ----------------------------------------------------- |
| Nombre de partitions primaires | 4                    | 128                                                   |
| Nombre de partitions logiques  | Jusqu'à 128          | N/A (les partitions logiques ne sont pas nécessaires) |
| Taille maximale du disque      | 2 To                 | 256 To                                                |
| Utilisation                    | Vieux systèmes, BIOS | Systèmes modernes, UEFI                               |

Il faut définir une **séquence de boot (boot sequence)**, qui définit les partitions sur lesquels ont peut boot, et leur ordre d'exécution. 
- Partition de boot par défaut
- Chargement d'un programme de lancement (grub)
- Choix du noyau (option)


****
## Montage (`mount`/`umount`)

Une fois le FS créé sur une partition, il doit être **monté** dans un répertoire ou à la racine.
**Exemple** : `sudo mount /dev/sdb2 /media/macle` 
	*Cela permet de rendre une clef USB (ou autre support) accessible depuis le répertoire indiqué. Ce processus est souvent automatiquement effectué par l'OS lors de l'insertion d'une clef, ou d'un disque.*


Cependant, la commande `mount` ne fait pas de montage permanent. Ce point de montage aura disparu au redémarrage du système.  

Pour faire cela de manière permanente, on modifie le fichier `/etc/fstab` (File System Table) qui liste les partitions et indique où les monter lors du lancement de la machine.


**Monter un disque de manière permanente**

1. Lister les partitions : `sudo fdisk -l`
2. Récupérer l'UUID de la partition à monter (Universal Unique Identifier) : `sudo blkid`
3. Modifier `/etc/fstab` (en sudoer) :
```
# Note: Chaque valeur d'une ligne doit être espacée d'une tabulation
# Part 1 (/dev/sda2)
UUID=28441657442529C8	/mnt/cinema	ntfs	defaults,noatime,uid=1000,gid=1000,umask=007	0	0

# Part 2 (/dev/sdb2)
UUID=44ebf667-df7f-488c-bdda-a75372b308f6	/mnt/main	btrfs	defaults,noatime,uid=1000,gid=1000,umask=007	0	0
```

4. Prendre les droits sur le point de montage :
```bash
sudo chown -R sunless:sunless /mnt/main
sudo chmod 700 /mnt/main # (no need for recursion, just limit visibility on the entry point)
```
