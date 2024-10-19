[[Axe 3]]
****
## Call

On souhaite avoir accès à un mécanosme qui nous permet d'écrire des fonctions et de les appeler (comme pour un langage haut niveau standard).

On passe pour cela par l'insturction suivante :
`call ETIQ`

De la même manière que l'instruction `jmp`, on saute vers une étiquette. Cependant, cette instruction va, au préalable, **empiler l'adresse de l'instruction qui la suit dans le programme**.
	*L'intéret est de pouvoir revenir au code principal une fois la fonction terminée (comme en C, par exemple), au lieu de continuer linéairement et de dépasser la fonction.*

Les deux codes ci-dessous sont donc équivalents :
```assembly
call function
function:
	; code fonction
; code suite
```

```assembly
push suite
jmp function
suite:
	; code suite
```


****
## Ret

Donc, on voit que l'intérêt d'enregistrer l'adresse de l'instruction qui suit un `call ETIQ` repose sur le fait que l'exécution peut revenir à cette instruction.

Cette fonctionnalité est offerte par l'instruction `ret`, qui va dépiler la donnée en tête et sauter à l'adresse qu'elle spécifie.
	*Il faudra donc que la valeur en tête de pile soit effectivement l'adresse empilée par `call`*

**L'instruction `ret` est donc un return, qui nécessite une adresse vers laquelle se replier afin de reprendre le fil d'exécution du programme.** Call et ret marchent ensemble (sauf si l'on souhaite empiler à la main).

On peut schématiser l’action produite sur l’exécution d’un programme par le couple `call`/`ret` ainsi :
![[couple.png]]
	*C'est exactement le comportement d'une fonction dans un langage haut niveau*


****
## Exemples d'utilisation en paire

Considérons la suite d'instruction suivante et observons l'état de la pile et du pointeur d'instruction au fil de l'exécution :
```assembly
mov ecx, 8        ; Adresse a1.
call loin         ; Adresse a2.
suiv: add ecx, 24 ; Adresse a3.
loin: add ecx, 16 ; Adresse a4.
ret               ; Adresse a5.
...               ; Adresse a6.
fin:              ; Adresse a7.
```

![[call-ret.png]]


****
## Conventions

Il est conseillé de respecter les **conventions d'appel du C** lorsque l'on écrit des fonctions en assembleur — et donc qu'on utilise le couple `call`/`ret`

On respectera les points suivants :
1. On **empile les arguments de la fonction** (on ne les stocke pas dans les registres, donc). Ils seront récupérés dans la fonction.
2. Le **résultat** — unique — de la fonction est **écrit dans EAX**
	*Comme on l'a vu, l'instruction `ret` ne prend pas d'opérandes en paramètres. Elle ne renvoit pas de valeur, elle se contente de retourner à une adresse.*
3. La **pile doit être identique** avant et après l'appel de la fonction
4. Toutes les valeurs des registres autres que EAX, ECX et EDX doivent être également identiques

L'objectif est que le programme garde toujours un état consistant (qu'on n'ait pas des valeurs qui disparaissent par magie, une pile qui change ...).


On respecte ce squelette pour l'écriture d'une fonction :
```assembly
function:
	push ebp
	mov ebp, esp
	; CODE
	pop ebp
	ret
```
*On sauvegarde toujours la valeur de EBP **(framepointer)** à l'entrée de la fonction, avant de passer au code réel, car sa valeur est susceptible de changer dans notre code.
Le registre permet d'accéder aux arguments :*
```assembly
ebp + 4 * (i + 1)
```


Et celui-ci pour l'appel de la fonction :
```assembly
push ARG_N
...
push ARG_1
call function
add esp, 4 * N ; Ou N est le nombre de dword empilés (arguments)
```


On obtiendra une pile de ce genre :
![[function.png]]
