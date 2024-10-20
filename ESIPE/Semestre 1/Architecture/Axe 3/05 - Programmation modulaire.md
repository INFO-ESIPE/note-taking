[[Axe 3]]
12/10/2022
****
## Etiquettes locales

Il peut devenir difficile de gérer toutes les étiquettes d'un programme, car elles finiront par devenir nombreuses.

Il est possible de passer par des étiquettes locales qu'on écrit `.etiq:` :
```assembly
etiq_globale:
	.action:
		INSTR_1
		jmp .action

autre_etiq_globale:
	.action:
		INSTR_2
		jmp .action
```


****
## Modules

Comme pour les autres langages de programmation, il est possible de répartir son projet assembleur sur plusieurs fichiers. On les découpes en **modules**.

Un module contient une collection de fonctions destinées à être utilisées depuis l'extérieur. Un module est constitué :
- D'un fichier source `.asm`
- D'un fichier d'en-tête `.inc` (inclusion)

Le seul fichier source qui ne contient pas d'en-tête est celui spécifiant la fonction principale `main`.


On autorise une étiquette d'instruction à être visible depuis l'extérieur en la précédent par `global ETIQ`. De plus, on renseigne dans le fichier d’en-tête l’existence de la fonction par avec `extern ETIQ`.
	*On remarque que cette logique est très similaire à celle du C, avec les fichiers headers `.h` qui permettaient d'inclure des fichiers externes.
	L'avantage de cette méthode est que les inclusions fonctionnent comme des API. On connait les fonctions que l'on peut appeler, mais on ne voit pas forcément l'implémentation derrière (code pur du fichier `.c`). Puisqu'on suit une logique d'API, on documentera la fonction dans le fichier d'en-tête (au dessus de la clause `extern ETIQ`, de la même manière que pour les fichiers header en C.*


Pour inclure un module **M** dans un fichier **F.asm**, on invoque la directive suivante au tout début du fichier :
```assembly
%include "M.inc"
```

Encore une fois, équivalent à :
```c
#include "M.h"
```


On compile ainsi :
```bash
nasm -f elf32 Main.asm
nasm -f elf32 M1.asm
...
nasm -f elf32 Mk.asm
ld -o Exec -melf_i386 -e main Main.o M1.o ... Mk.o
```


****
## Exemple

On définit un module "ES".
**ES.asm :**
```assembly
global print_char
print_char:
	...

global print_string
print_string:
	...
```

**ES.inc :**
```assembly
; Documentation de la fonction
extern print_char

; Documentation de la fonction
extern print_string
```

**Main.asm :**
```assembly
%include "ES.inc"
...
call print_string
...
```


