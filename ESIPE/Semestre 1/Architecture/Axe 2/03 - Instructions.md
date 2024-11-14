[[Axe 2]]
02/10/2022
****
**Table of Contents**
```table-of-contents
```

****
## Instruction `mov`

L'instruction `mov` permet de recopier une valeur dans le registre souhaité. 
	*Le premier argument sera le registre, le second sera la valeur à assigner.*
![[mov.png]]

**Il est important que la taille du registre/sous-registre donné soit supérieur ou égal à celle de la valeur passée en paramètre.** Si ce cas arrive, plusieurs comportements peuvent arriver :
- Troncature des bits de poids fort
- Erreur de compilation

![[mov-wrong.png]]

![[mov-valid.png]]


On peut aussi utiliser les crochets pour **recopier des valeurs contenues à un espace mémoire précis** (plus de détails [[03 - Instructions#Addressing modes|ici]]) :
```asm
mov ebx, 0x05 ; met 5 dans le registre ebx
mov eax, ebx  ; recopie 5 dans le registre eax

mov ebx, 0x1234ABCD ; on indique une adresse mémoire contenant une valeur qui                        ; nous intéresse (e.g. "0x50" est la valeur stockée à cette                      ; adresse)
mov eax, [ebx] ; va récupérer la valeur contenue à l'adresse 0x1234ABCD et la                   ; copier dans eax (eax contiendra donc 0x50)
```
*Le registre placé entre crochets devra absolument contenir une **adresse légale** (qui est réservée en RAM pour le programme actuel) 
Si on spécifie une connerie (ou une adresse à laquelle on n'a pas accès), le programme **SEGFAULT** (voir cour de C2 avec les allocations dynamiques)*


L'instruction — plus avancée — `lea` permet un résultat un peu similaire. Elle est surtout utilisée en Reverse Engineering pour récupérer une valeur contenue dans un tableau (dans `esi+4*ecx` par exemple).


****
## Addressing modes

En assembleur, on peut passer des valeurs de différentes façons :
- Register addressing - On donne le contenu brut d'un registre
	`mov eax, ebx ; on copie le contenu brut de ebx dans eax`
- Immediate addressing - On donne une valeur
	`mov eax, 0x10 ; on place 10 dans eax`
- Memory addressing - On récupère la valeur contenue à une adresse mémoire spécifique
	`mov [ebx], eax ; va copier la valeur stockée dans EAX à l'adresse que contient ebx`
	*Un peu comme l'instruction `jmp 0x401438` qui va directement sauter à l'adresse donnée
	Bien entendu, il est attendu que **EBX contienne une adresse correcte à la quelle on a accès***



****
## Opérateurs arithmétiques de base

| **Mnémoniques** | **Comportement**                                                                                                                                                                                                                                                                       |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `add` REG, VAL  | Incrémente le registre REG avec la valeur VAL                                                                                                                                                                                                                                          |
| `sub` REG, VAL  | Décrémente le registre par la valeur spécifiée                                                                                                                                                                                                                                         |
| `mul` REG       | Multiplie **la valeur contenue dans EAX** par celle dans le registre REG, et **place le résultat dans EDX:EAX**. <br>*Le résultat est donc stocké dans une suite de 64 bits dont les bits de EDX sont ceux de poids fort.*                                                             |
| `div` REG       | La valeur que l'on souhaite diviser sera présente dans le registre EAX. <br><br>Le diviseur (ce par quoi on divise la valeur du registre) sera contenue dans le registre REG.<br><br>Le résultat de cette opération sera, une fois de plus, stocké dans le registre double **EDX:EAX** |


****
## Opérateurs logiques

| **Mnémoniques** | **Arguments**           | **Résultats** | **Comportement**                                                                                                                                                                                                                       |
| --------------- | ----------------------- | ------------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `not` REG       | 1111 1010               | 0000 0101     | Inverse tous les bits de la valeur contenue dans le registre **REG** (effectue un "NON" bit à bit). <br><br>Si un bit est 1, il devient 0, et inversement.                                                                             |
| `and` REG, VAL  | 1101 0011 <br>1010 0001 | 1000 0001     | Effectue un "ET" logique bit à bit entre la valeur contenue dans **REG** et **VAL**. Le résultat est placé dans **REG**. <br><br>Seuls les bits qui sont 1 dans les deux opérandes deviennent 1 dans le résultat.                      |
| `or` REG, VAL   | 1110 0011 <br>0111 1010 | 1111 1011     | Effectue un "OU" logique bit à bit entre la valeur contenue dans **REG** et **VAL**. Le résultat est placé dans **REG**. <br><br>Un bit sera 1 si l'un des deux bits correspondants (dans REG ou VAL) est 1.                           |
| `xor` REG, VAL  | 1100 1001 <br>1110 0101 | 0010 1100     | Effectue un "OU exclusif" (XOR) logique bit à bit entre la valeur contenue dans **REG** et **VAL**. Le résultat est placé dans REG. <br><br>Un bit sera 1 uniquement si l'un des deux bits (dans REG ou VAL) est 1, mais pas les deux. |


****
## Opérateurs bit à bit

| **Mnémonique** | **Comportement**                                                                          |
| -------------- | ----------------------------------------------------------------------------------------- |
| `shl` REG, NB  | Décale les bits du registre vers la gauche de NB places, et complète à droite par des "0" |
| `shr` REG, NB  | Décale les bits du registre vers la droite de NB places, et complète à gauche par des "0" |
| `rol` REG, NB  | Réalise une rotation des bits du registre REG à gauche de NB places                       |
| `ror` REG, NB  | Réalise une rotation des bits du registre REG à droite de NB places                       |

*Si on décale vers la gauche, alors les bits les plus à gauche (qui dépassent) vont disparaitre. De même pour le décalage vers la droite...*

