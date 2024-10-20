[[Axe 2]]
06/10/2022
****
**Note**: On utilise NASM pour mener à bien toutes les actions décrites ci-dessous.
****
## Compilation et édition de liens

Pour assembler un programme écrit en assembleur à l'aide de **NASM**, on utilise la commande suivante :
```bash
nasm -f elf32 nomProgramme.asm
```

Le flag "-f" permet de spécifier un format de sortie précis. Ici on prend **elf32 (Executable and Linkable Format 32-bit)**, le format courant pour les exécutables sur linux en x86.


L'exécution de cette commande produit un fichier **objet** nommé **`monProgramme.o`**. Ce fichier contient les instructions binaires traduites du code assembleur, mais il ne s'agit pas encore d'un programme exécutable.


****
## Création de l'exécutable

Pour obtenir un programme exécutable à partir du fichier objet, il est nécessaire d'éditer les liens à l'aide de la commande **`ld`** (éditeur de liens) :
```bash
ld -o leProgramme -e main monProgramme.o
```

Le flag "-o" permet de spécifier le nom de sortie de l'exécutable. Le flag "-e" indique le **point d'entrée du programme**, c'est-à-dire que l'exécution commencera à l'adresse correspondant à l'étiquette **`main`** dans le code assembleur.


****
## Remarques pour les architectures x64

Si l'on souhaite faire fonctionner correctement notre programme x86 sur un système d'exploitation **64 bits**, l'option supplémentaire **`-melf_i386`** doit être ajoutée à la commande **`ld`** pour indiquer que l'on souhaite lier un programme en **32 bits** (mode **i386**).

La commande complète devient alors :
```bash
ld -o leProgramme -melf_i386 -e main monProgramme.o
```
