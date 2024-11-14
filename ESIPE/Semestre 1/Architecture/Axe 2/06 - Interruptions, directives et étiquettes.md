[[Axe 2]]
06/10/2022
****
**Table of Contents**
```table-of-contents
```

****
## Interruption logicielle

L'exécution d'un programme se fait instruction par instruction. Dès qu'une instruction est traitée, le processeur s'occupe de celle suivante. 
Cependant, certaines instructions ont besoin d'interrompre temporairement l'exécution normale du programme pour mener à bien des tâches spécifiques. Ce sont des **méthodes bloquantes**
	*Notion abordée plus en détail au semestre 3 et 4 avec la programmation concurrente et la programmation réseau...*

Parmi celles-ci, il y a: 
- L'écriture de texte sur la sortie standard 
- La lecture d'une donnée sur l'entrée standard 
- L'écriture d'une donnée sur le disque 
- La gestion de la souris 
- La communication via le réseau 
- La sollicitation de l'unité graphique ou sonore

Afin de mener à bien ces objectifs, des instructions particulières ont été mises en places. Ces instructions spéciales — **interruptions** — sont des demandes spéciales envoyées au système d'exploitation pour qu'il prenne temporairement le contrôle et exécute une routine dédiée (souvent appelée **gestionnaire d'interruption**).


On appelle une interruption — ou le processeur va interrompre le programme en cours et passer temporairement la main au noyau de l'OS — via la commande suivante (pour une architecture x86) : 
```
int 0x80
```

Le registre "EAX" doit contenir le **code système de la tâche à effectuer** (un peu comme un identifiant unique associé à chaque type d'action) :

| Code | Rôle                           |
| ---- | ------------------------------ |
| 1    | Arrêt et fin de l'exécution    |
| 3    | Lecture sur l'entrée standard  |
| 4    | Ecriture sur l'entrée standard |
| ...  |                                |
 
 Les autres registres de travail contiendront des arguments nécessaires à la tâche en question  


**Attention**: Le traitement d'une interruption peut modifier le contenu des registres de travail. Il est préférable de sauvegarder leur valeur dans la pile avant d'exécuter cet appel.
	*On passera par les instructions `push` et `pop`, plus d'informations dans la suite du cours*

**Exemples :**
![[int.png]]


`int 0x80` effectue en réalité un **appel système (syscall)**. L'appel précis varie en fonction des paramètres que l'ont fournit dans les registres.
En assembleur x64, on passe par l'instruction `syscall`, cependant, cette dernière n'existe pas en x86, d'ou l'utilisation de `int 0x80` à la place :
```assembly
; code x86
mov eax, 1        ; numéro de l'appel système (exit)
mov ebx, 0        ; code de retour
int 0x80          ; appel système

; équivalent code x64
mov rax, 60       ; numéro de l'appel système (exit)
mov rdi, 0        ; code de retour
syscall           ; appel système
```
*Plus d'informations sur les appels systèmes dans le cours de Programmation Systèmes du S3.*


****
## Directives

Une **directive** est un élément d'un programme **destiné à informer l'assembleur** de certaines configurations ou instructions préliminaires. 
Contrairement aux instructions machine, **les directives ne sont pas traduites en langage machine** et n'affectent pas directement l'exécution du programme. Elles servent à organiser et préparer le code.

**Exemples d'utilisation :**
- La définition d'une constante 
	```assembly
	%define MAX_VAL 100 ;
	```

- L'inclusion d'un fichier 
```assembly
%include "lib.asm" ; Inclusion d'un fichier externe appelé "lib.asm"
```
*Le contenu du fichier inclus sera traité comme s'il faisait partie intégrante du fichier principal, et ses instructions ou déclarations seront accessibles dans le programme._*


****
## Etiquettes

En assembleur, **les instructions** d'un programme sont **associées à des adresses** en mémoire, comme n'importe quelle autre donnée. Pour faciliter la gestion de ces adresses, il est possible de définir des **étiquettes (labels)** qui permettent de nommer des adresses spécifiques dans le programme.
	*Les principaux objectifs sont d'associer un nom symbolique à une certaine instruction, ou encore de faciliter les sauts vers une instruction spécifiques au travers des **instructions de contrôle de flux** (jmp, je, jne).*

Syntaxe: 
`ETIQ: INSTR` 
	*ou "ETIQ" est le nom de l'instruction, INSTR est l'instruction associée*


**Exemple :**
![[etiquette.png]]


**Remarque**: **`main`** est souvent une **étiquette d'instruction spéciale** qui marque le point d'entrée du programme. Sa valeur est l'adresse de la première instruction à exécuter lors du démarrage.