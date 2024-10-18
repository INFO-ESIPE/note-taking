[[Axe 2]]
****
## Section - Données initialisées (.data)

Partie facultative du programme qui regroupe des définitions de données pointées par des adresses.

On définit une donnée de la façon suivante: 
```assembly
id: DT VAL
```
ou "id" est une **étiquette** (identificateur), "DT" est le descripteur de taille et "VAL" la valeur qu'on souhaite lui donner.
	*La véritable adresse que prendra cette valeur sera attribuée par le système.*

La syntaxe du DT sera formulée ainsi :

| Descripteur de taille | Taille (octets) |
| --------------------- | --------------- |
| db (byte)             | 1               |
| dw (word)             | 2               |
| dd (double)           | 4               |


**Exemples :**
![[data.png]]


Il est également possible de définir plusieurs données de manière concise de la manière suivante : 
```assembly
id: times NB DT VAL
```
ou "NB" sera une valeur positive qui va créer NB occurences de VAL.

**Exemples :**
![[data-multi.png]]


****
## Section - Données non-initialisées (.bss)

Partie facultative du programme qui regroupe des déclarations de données non initialisées pointées par des adresses.

On définit une donnée de la façon suivante :
```assembly
id: DT NB
```
ou id est — encore — une étiquette, NB une valeur positive et DT un descripteur de taille parmi les valeurs suivantes :

| Descripteur de taille | Taille (octets) |
| --------------------- | --------------- |
| resw (byte)           | 1               |
| resw (word)           | 2               |
| resd (double)         | 4               |
| resq (quadword)       | 8               |

Cette commande **réserve une zone mémoire commençant à l'adresse ID, et pouvant accueillir NB données ayant une taille définie par DT**.

**Exemple :**
![[alloc.png]]


****
## Section - Instructions (.text)

Une **étiquette de code** devra être créée, afin de définir le point d'entrée du code, et de rendre ce dernier visible globalement :
```assembly
section .text
global main
main:
	; INSTR
```

"main" est l'étiquette de code.
"global main" rend l'étiquette visible pour l'édition des liens.
