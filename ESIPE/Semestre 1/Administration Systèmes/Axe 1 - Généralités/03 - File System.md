[[Axe 1 - Généralités]]
****

Debian utilise **ext4fs** comme File System (FS). 
Ext4fs est un FS journalisé
- Enregistre les modifications dans un journal avant de les faire sur les fichiers 
- Permet une récupération 
- NTFS permet aussi cela grâce au Volume Shadow (mais pue la merde quand même)

Il existe de nombreux autres FS, chacun ayant ses propres caractéristiques et fonctionnalités.
	*Btrfs et ext4 sont pas mal*


****
## Arborescence Linux

Les fichiers sont organisés sous forme d'arborescence. Les répertoires à la racine sont les plus importants, chacun sert un but précis :
![[tree.png]]


****
## Commandes génériques

| Commande     | Exemple                                        | Résultat                                                                                                                                             |
| ------------ | ---------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------- |
| pwd          | `pwd`                                          | Affiche l'emplacement actuel (répertoire courant)                                                                                                    |
| cd           | `cd ~/Documents`                               | Se mouvoir entre les répertoires                                                                                                                     |
| ls           | `ls -lA /usr/bin`                              | Liste les élements présents dans le dossier ciblé                                                                                                    |
| ln           | `ln file1.txt file2.txt`                       | Créer des liens durs                                                                                                                                 |
| ln -s        | `ln -s /path/file.txt linkname.txt`            | Créer des liens symboliques                                                                                                                          |
| rm, rmdir    | `rm -rf /` (faites pas ça hein)                | Supprime un fichier, supprime un dossier                                                                                                             |
| mv           | `mv unnamed.csv renamed.csv`                   | Déplacer un fichier<br>**Astuce** : permet aussi de renommer un fichier                                                                              |
| cp           | `cp save save.bak`                             | Copier un fichier                                                                                                                                    |
| mkdir, touch | `mkdir scripts && touch ./scripts/myscript.sh` | Créer un dossier, créer un fichier                                                                                                                   |
| cat          | `cat file1.txt`                                | Affiche le contenu d'un fichier                                                                                                                      |
| file         | `file executable.bin`                          | Détermine le type d'un fichier via **Magic byte**                                                                                                    |
| man          | `man awk`                                      | Affiche le [[02 - GNU Linux#Pages `man` (manual)\|manuel d'une commande, fonction ...]]                                                              |
| locate       | `locate mypasswords.txt`                       | Cherche les fichiers dans le système via **indexation**.<br>Les index sont stockées dans une base de données qu'on peut mettre à jour via `updatedb` |
| find         | `find $HOME/ -name "file.txt"`                 | Cherche les fichiers en **temps réel** dans le répertoire spécifié                                                                                   |


****
## Disque

L'utilisateur se représente les fichiers sur disque comme une arborescence, ce qui est en réalité — bien évidemment — pas le cas. 
Les fichiers sont souvent découpés en plusieurs morceaux éparpillés sur le disque. En réalité, c'est le File System qui aggrège les valeurs et en sort une arborescence, tout en y ajoutant des métadonnées.


Un disque est composé des partitions et d'un **Master Boot Record (MBR)** — qui contient les informations utiles pour démarrer le système (boot).
Chaque partition est formée en suivant la structure suivante (structurée comme un tableau en C, donc des éléments de taille fixe) : 
- **Boot block** : Utilisé si la partition contient un système d'exploitation 
- **Super Block** : Endroit qui contient les métadonnées concernant la partition (petit emplacement) 
- **Inode Table** : Table des inodes
	Un **inode** est un élément de la partition. Chaque inode représente donc un fichier, mais contient aussi énormément de métadonnées concernant le fichier (propriétaire, permissions....). Cependant, le nom du fichier ne figure pas dans les métadonnées. L'inode contient un champ de données "directes" qui pointe vers le contenu réel du fichier (data blocks).

	La contrainte que pose l'inode est sa taille. Comme mentionné précédement, on peut se représenter la structure comme un tableau en C. L'inode est donc un élément du tableau qui doit posséder une taille fixe identique aux autres. Etant donné le caractère variable des fichiers (essentiellement leur taille), on utilise des **indirects (indirections)** pour pointer vers des blocs de données externes qui contiendront les données du fichier, ou d'autres indirections. Un inode ne possède que 3 indirections au maximum, mais chaque indirection permet de pointer sur plusieurs autres indirections, c'est donc exponentiel et on n'a pas réellement besoin de plus de 3 indirections par inode. Cela résout donc le problème des fichiers très volumineux.

- **Data Blocks** : Contenu réel des fichiers et répertoires


****
## Répertoires UNIX

Un répertoire — sous UNIX — est un fichier spécial qui associe un nom de fichier à un numéro d'inode. Cela lui permet de représenter une séquence d'entrées. 

Si le dossier "home" est associé au numéro d'inode 17, on consultera ce qui se trouve a l'inode 17. Si cet inode est un répertoire, on continue se principe d'arborescence.
![[dirs.png]]

Les entrées "." (répertoire actuel) et ".." (répertoire parent) sont systématiquement incluses et comportent les numéros d'inodes associés ("." aura l'inode du répertoire actuel, et ".." l'inode du parent).
	*Si cela n'existait pas, on aurait du mal à s'y mouvoir via chemin relatifs...*


****
## Fichiers

Un répertoire est un fichier, car **tout est un fichier (everything is a file)**
	*Tout est représenté comme une fréquence d'octet (stream of bytes)*

Il existe plusieurs entitées représentés par des fichiers: 
- Fichiers normaux (ce qu'on entend communément comme "fichier") 
- Répertoires
- Liens symboliques (symlinks)
- Inter-process communications 
    - Pipes and FIFOs 
    - Network sockets 
- L'équipement hardware 
    - Block Devices (Disques) 
    - Character devices (claviers, imprimantes…) 
- Pseudo-devices 
    - `/dev/null` (e.g. gcc 2>/dev/null) 
    - `/dev/random` (e.g. cat -v /dev/random) 
    - `/dev/zero` (e.g. cat -v /dev/zero) 

Ils partagent tous la même API. On peut donc gérer chacun de ces cas de la même manière.


****
## Liens

Il existe deux types de liens :
- Liens durs (Hard link)
- Liens symboliques (Symlink) 

Il est possible qu'un fichier apparaisse plusieurs fois sur le FS, et le lien est la pour représenter le fichier a plusieurs endroits sans avoir à en faire une copie. 
	*Ce méchanisme explique pourquoi le nom de fichier est dissocié des métadonnées de l'inode...*


Les liens durs est donc une **référence vers une inode fonctionnant sur tous les fichiers standards, mais cela exclu les répertoires**. 
	*On l'interdit afin d'empecher les boucles et d'éviter à des algorithmes ou programmes mal conçus (ou voulant économiser la mémoire, ou voulant éviter une complexification de l'algo) de faire une récursion...*

Le lien symbolique n'est rien de plus qu'**un fichier contenant un path**. On peut le voir comme un fichier texte tout simple contenant le chemin vers la detination. 
	*Contrairement au hard link, ce lien casse si le fichier de base (sur lequel pointe le lien) est supprimé.*


****
## File Hole

Il est possible de déplacer le curseur (via `lseek()`) dans tout le fichier que l'on édite, y compris **une position située bien après la supposée fin du fichier** (`EOF`, End Of File). 

Dans ce cas, chaque octet post-`EOF` sera représenté par un 0, afin d'atteindre la position du curseur. Cela résulte en la création d'un fichier très volumineux si on décale le curseur loin (e.g. 1_000_000).


**Exemple :**
On effectue une copie de ce fichier (cat fichier > fichier_copy) pour voir si il y a une différence de taille. 
On peut demander à ls de donner la taille logique (`ls -lh`), mais aussi la taille par bloc (`ls -lsh`).

On remarque que la taille logique restera identique, mais la taille réellement utilisée sera bien moindre qu'un fichier avec 1 million de vraies valeurs. 
Cela s'explique par le fait que lseek compense avec des "0" qui sont représentés implicitement, il n'y a pas réellement des milliers d'octets de 0 sur le disque.

Le fichier obtenu par `cat` aura la taile totale car il entre les 0 en dur sans **neuristique**, tandis que les autres auront la taille minimale d'un bloc (environ 4 Ko).


*A noter qu'on ne peut pas allouer partiellement un bloc, à partir du moment ou un fichier prend au minimum un octet (un caractère), il va allouer un bloc entier.*
