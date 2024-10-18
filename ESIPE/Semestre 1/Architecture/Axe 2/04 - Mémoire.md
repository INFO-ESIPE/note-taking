[[Axe 2]]
****

Les registres n'offrent pas suffisamment de mémoire pour construire des programmes élaborés. C'est pour cette raison que l'on a introduit le principe de mémoire.

La mémoire est segmentée en plusieurs parties: 
- **La zone statique (segment de données)**
	- Contient **le code (instructions du programme) et les données statiques (variables globales, constantes, variables initialisées).**
	Ces données statiques seront présente pendant toute la durée du programme, et garderont une adresse fixe en mémoire.

- **Le tas (heap)**
	- Stockage dynamique, essentiellement **pour les données à taille et durée de vie indéterminée à la compilation**. 
	- Espace alloué manuellement (via `malloc`, `realloc`, `free` en C, `new` et `delete` en C++).
	- La taille du tas change au fil de l'exécution, s'agrandit quand on alloue des choses, et se réduit quand on les libères. Cette tâche est gérée par le **garbage collector** du langage (s'il y en a), sinon le programmeur se démmerde hein 

- **La pile (stack)**
	- Utilisée pour le **stockage automatique des variables locales, des paramètres de fonctions, des adresses de `return`...**
	- Logique **Last In First Out (LIFO)**, comme une pile d'assiettes.
	- Taille variable, s'agrandit ou rétrécit en fonction des fonctions appelées et retournées.
	- Gérée automatiquement par le processus


En mode **protégé**, chaque programme en exécution possède son propre environnement de mémoire. 
	*Les adresses y sont relatives et non absolues.*

De ce fait, un programme en exécution ne peut empiéter ou réécrire sur la mémoire d'un autre.

En général, la lecture/écriture en mémoire suit la convention du **Petit boutisme (Little Endian)**.


****
## Lecture en mémoire

On considère l'instruction suivante: 
	`mov` REG, 'ADR'
Cette instruction place la valeur obtenue depuis l'adresse ADR (dans la mémoire) dans le registre REG.

![[read.png]]
	*ou X+3 est l'octet de poids fort et X celui de poids faible (du bas vers le haut)*


**Exemple détaillé**

![[readfull.png]]

Chaque ligne identifiée par un "x", correspondrait dans un cas concret à une adresse hexadécimale (0x00000000 - 0xFFFFFFFF) 

**Mettre une adresse entre crochets permet donc de récupérer la valeur qu'elle contient.**
Cette valeur est ensuite stockée dans des registres. 

**Attention**: la taille du registre sur lequel on place la valeur, et cette dernière, doivent correspondre.

****
## Ecriture en mémoire

On considère l'instruction suivante: 
	`mov` DT 'ADR', VAL 
Cette instruction écrit la valeur VAL dans la mémoire, à l'adresse 'ADR'. 


Le champ **DT** est un **descripteur de taille**. Il précise la taille que prendra VAL.

| Descripteur de taille | Taille (octets) |
| --------------------- | --------------- |
| byte                  | 1               |
| word                  | 2               |
| dword                 | 4               |

Exemple avec les véritables DT ci-dessus :
![[dt.png]]

On peut, de plus, passer directement un registre au champ VAL. Si tel est le cas, le champ DT ne devra pas être considéré :
![[dt-instr.png]]


**Exemple détaillé**

![[ESIPE/Semestre 1/Architecture/Axe 2/write.png]]

