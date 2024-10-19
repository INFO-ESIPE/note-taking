[[Axe 4]]
****
## Horloge

Jusqu'ici, nous avons suivi une architecture simplifiée suivant l'architecture suivante :
1. **Instruction Fetch (IF)**
	**Chargement** de l’instruction et mise à jour du pointeur d’instruction
	EIP de sorte qu’il contienne l’adresse de la prochaine instruction à exécuter

2. **Instruction Decode (ID)**
	**Identification** de l’instruction. Les arguments éventuels de l’instruction sont placés dans l’unité arithmétique et logique.

3. **Execute (EX)**
	**Exécution** de l’instruction par l’unité arithmétique et logique

4. **Write Back (WB)**
	**Ecriture** (éventuelle) dans des registres ou de la mémoire


**Exemple 1 :** `add eax, [adr]`
1. (IF) L’instruction `add` est chargée et EIP est incrémenté de 4 afin qu’il pointe vers la prochaine instruction

2. (ID) Le système repère qu’il s’agit de l’instruction `add` et il lit les 4 octets situés à partir de l’adresse `adr` dans la mémoire, ainsi que la valeur du registre EAX

3. (EX) L’unité arithmétique et logique effectue l’addition entre les deux valeurs chargées

4. (WB) Le résultat est écrit dans EAX

**Exemple 2 :** `jmp adr`

1. (IF) L'instruction `jmp` est chargée et EIP est incrémenté de 4 afin qu’il pointe vers la prochaine instruction (il ne pointe donc pas encore sur l’instruction d’adresse `adr` souhaitée)

2. (ID) Le système repère qu’il s’agit de l’instruction `jmp` et il lit la valeur de l’adresse `adr`

3. (EX) L’adresse d’instruction cible est calculée à partir de la valeur de `adr`

4. (WB) L’adresse d’instruction cible est écrite dans EIP

****

On remarque, au travers de ces exemples, que la moindre action — même une instruction aussi simple qu'une addition — n'est pas **atomique** (elle ne se fait pas en une seule action).

Ces actions doivent être **rythmées et synchronisées par l'horloge du processeur (clock cycle)** que l'on mesure en Hertz.
	*Un processeur avec une fréquence de 3 GHz effectuera 3 milliards de cycles d'horloge par seconde...*

Chacune des étapes ci-dessus nécessite **au minimum un cycle d'horloge (tick)**.
	*Cela peut prendre bien plus d'un cycle par instruction. Par exemple, la division `div` est difficile à calculer pour un processeur, et va nécessiter plusieurs dizaines de cycles afin de réaliser l'étape EX*


**Problématique :** Comment optimiser l'exécution des instructions ?


****
## Pipeline

Le **pipeline** est une technique utilisée par les processeurs pour améliorer la performance de l'exécution des instructions séquentielles. 
L'idée de base est de diviser le processus d'exécution d'une instruction en plusieurs étapes distinctes (ou **phases**), chacune pouvant être exécutée en parallèle avec d'autres.

![[pipeline.png]]


Chaque étape est prise en charge par un **étage** du pipeline. 
	*Dans l'exemple ci-dessus, il y a quatre étages.*


Du a l'entrelacement dans l'exécution des instructions, différents **aléas** peuvent survenir :
- **Aléas structurels** - Conflit d'utilisation d'une ressource matérielle
- **Aléas de données** - Résultat dépendant d'une précédente instruction
- **Aléas de contrôle** - Saut vers un autre endroit du code

La solution générique consiste à suspendre l'exécution de l'instruction qui pose problème jusqu'a à ce que le problème ce résolve.
	*Cette mise en pause d'instructions va créer des **bulles** dans le pipeline*


****
## Aléas structurels

Ce problème survient si deux instructions accèdent au même moment à la même ressource.

![[structural.png]]

Ici, la 1ère (écriture en mémoire) et 4ème (fetch) instructions toutes deux sollicitent au même moment le bus reliant l’unité arithmétique et logique et la mémoire.


Pour résoudre le problème, on va temporiser la seconde instruction (bulle) :
![[bulle.png]]
	*Cela règle la collision entre la première et dernière instruction. Cependant, on aura le même problème avec la seconde instruction. On va donc répéter la bulle jusqu'a temps que le problème ne se manifeste plus.*


****
## Aléas de donnée

Cet aléa survient si une instruction **I** à besoin du résultat d'une instruction **I'** qui n'a pas encore produit son résultat exploitable :
![[ESIPE/Semestre 1/Architecture/Axe 4/data.png]]
	*Ici, une mauvaise valeur de EAX sera chargée en tant qu'argument par la seconde instruction...*

La solution sera de **temporiser les trois dernières étapes** de la seconde instruction :
![[data-solve.png]]
	*On fait deux bulles, car sinon on obtient un aléa structurel (**ID** et **WB**).


Il est également possible d'arranger le code d'une certaine façon afin de limiter le nombre de bulles — et ainsi accroitre la vitesse d'exécution.

Prenons le code C suivant :
```c
a = a + b;
c = c + d;
```

On peut écrire ce code en assembleur de ces deux façons :
```assembly
; Traduit directement du programme C.
; Instructions mov et add trop proches !!
mov eax, [adr_a]
mov ebx, [adr_b]
add eax, ebx
mov [adr_a], eax
mov ecx, [adr_c]
mov edx, [adr_d]
add ecx, edx
mov [adr_c], ecx
```

```assembly
; Optimisé. On sépare deux isntructions se partageant la même donnée par 
; d'autres instructions indépendantes. Cela diminue le nombre de bulles
mov eax, [adr_a]
mov ebx, [adr_b]
mov ecx, [adr_c]
mov edx, [adr_d]
add eax, ebx
add ecx, edx
mov [adr_a], eax
mov [adr_c], ecx
```


****
## Aléas de contrôle

Ce dernier **se manifeste systématiquement lorsqu'une instruction de saut est exécutée**.
	*Si I est une instruction de saut, il est possible qu’une instruction I′ qui suit I dans le pipeline ne doive pas être exécutée (car on passe à une toute autre instruction). Il faut donc éviter/annuler l’étape **EX** de I′ (parce qu’elle modifie la mémoire alors qu'elle ne doit pas)*

La solution sera simplement de ne jamais exécuter les étapes des instructions sautées une fois que le **WB** de l'instruction de saut est exécuté.
	*C'est lors du WB de l'instruction `jmp` modifie le contenu du registre EIP (et ce même si l'adresse de saut est calculée à l'étape **EX** juste avant).*


****
## Mémoire
