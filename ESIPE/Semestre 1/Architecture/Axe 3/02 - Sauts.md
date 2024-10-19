[[Axe 3]]
****
## Sauts Inconditionnels

À chaque instant de l’exécution d’un programme, le registre **EIP (pointeur d’instruction)**
contient l’adresse de la prochaine instruction à exécuter.
	*Le programme récupère l'adresse de l'instruction **I** contenue dans EIP, met à jour EIP avec l'adresse de l'instruction **I'** suivante, puis traite l'instruction **I** en cours.*

**Note:** On ne peut pas intervenir *directement* sur la valeur contenue dans EIP.


Le programme s'exécute donc de manière linéaire, instruction par instruction dans l'ordre ou elles apparaissent originellement dans le fichier assembleur. Cependant, il est possible de rompre cette ligne d'exécution en réalisant des **sauts**. 

Les sauts (jmp) consistent à changer la valeur du pointeur d'exécution — contenue dans EIP — afin de sauter à l'endroit du code souhaité :
```assembly
mov ebx, 0xFF
jmp e1
mov ebx, 0

e1:
	mov eax, 1
```
Ici, l'instruction `mov ebx, 0` n'est pas exécutée, car on passe d'abord sur l'étiquette `e1`.


Les sauts effectués avec `jmp` sont dits **inconditionnels**. Ils s'exécutent peu importe la situation dès lors qu'on atteint l'instruction.


****
## Sauts Conditionnels

Un saut conditionnel est un saut qui n'est réalisé **que si une condition impliquant le registre "flags" est vérifiée**. Si celle-ci n'est pas vérifiée, l'exécution se poursuit de manière linéaire et le saut est ignoré.

| Instruction           | Condition de saut |
| --------------------- | ----------------- |
| je (j equal)          | VAL_1 = VAL_2     |
| jne (j not equal)     | VAL_1 != VAL_2    |
| ji (j lower)          | VAL_1 < VAL_2     |
| jle (j lower equal)   | VAL_1 <= VAL_2    |
| jg (j greater)        | VAL_1 > VAL_2     |
| jge (j greater equal) | VAL_1 >= VAL_2    |

**Exemple :**
```assembly
cmp eax, ebx
jl inferieur
jmp fin

inferieur:
	mov eax, ebx

fin:
	; ...
```

Ici, on commence par effectuer une comparaison entre les valeurs contenues respectivement dans EAX et EBX — ce qui va modifier les flags.
On peut ensuite effectuer le saut conditionnel `jl`:
- Si la valeur de EAX est strictement inférieure à celle de EBX, on passe à l'étiquette `inferieur`
- Sinon, l'exécution se poursuit, et on atteint le saut inconditionnel qui nous amène à la fin du programme


****
## Simulation de structures de contrôle

**If**

C:
```c
if (a == b) {
	// code
}
// ...
```

asm x86:
```assembly
cmp eax, ebx
je then
jmp end_if

then:
	; code

end_if:
	; ...
```


**If else**

C:
```c
if (a == b) {
	// code if
} else {
	// code else
}
// ...
```

asm x86:
```assembly
cmp eax, ebx
jne else
; code if
jmp end_if

else:
	; code else

end_if:
	; ...
```


**While**

C:
```c
while (a != b) {
	// code
}
// ...
```

asm x86:
```assembly
while:
	cmp eax, ebx
	je end_while
	; code
	jmp while

end_while:
	; ...
```


**Do While**

C:
```c
do {
	// code
} while (a != b);
```

asm x86:
```assembly
do:
	; code
	cmp eax, ebx
	jne do
```


**For**

C:
```c
for (int i = 0, i < 10, i++) {
	// code
}
// ...
```

asm x86 (première méthode):
```assembly
mov eax, 0

for:
	cmp eax, 10
	jge end_for
	; code
	add eax, 1
	jmp for	

end_for:
	; ...
```

asm x86 (seconde méthode):
	*On peut utiliser l'instruction `loop` qui va **décrémenter la valeur contenue dans ECX** à chaque boucle. Tant que ECX ne vaut pas 0, on boucle sur l'étiquette.*
```assembly
mov ecx, 10

new_for:
	; bloc
	loop new_for

; ...
```

Le code est pas exactement le même, mais l'idée est un peu similaire (juste c'est fait dans l'autre sens...)

