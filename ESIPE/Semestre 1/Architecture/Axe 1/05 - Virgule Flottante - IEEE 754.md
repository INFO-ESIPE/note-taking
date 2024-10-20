[[Axe 1]]
02/10/2022
****

A l'inverse de l'exemple précédent, qui mettait en place un principe de virgule fixe, cette norme prend en considération le principe de virgule flottante, pouvant être déplacé à gauche ou à droite. 

Ici, un exposant permettra de savoir ou la virgule sera à placer. 

Le nombre à virgule x sera représenté sous divers formats :

| Nom    | Taille | Bit de signe | Exposant | Mantisse |
| ------ | ------ | ------------ | -------- | -------- |
| Half   | 16     | 1            | 5        | 10       |
| Single | 32     | 1            | 8        | 23       |
| Double | 64     | 1            | 11       | 52       |
| Quad   | 128    | 1            | 15       | 112      |

*Le bit de signe est représenté par la lettre "s" 
La partie exposant est représenté par "e" 
La partie mantisse est représenté par "m"*


****
## Données

Comme auparavant, si le bit de signe vaut 0, le nombre est positif, et inversement. 
L'exposant correspond à la valeur de la puissance, permettant de bouger la virgule 
La mantisse contient la partie fractionnaire du nombre écrit en représentation scientifique.


****
## Exemples

**Exemple 1: 0d12,625**
On sait que ça représentation en virgule fixe donne 0b1100,101. 

La représentation scientifique de ce nombre sera donc: 1,100101x2³ 
A noter:  
- Le premier chiffre de la représentation scientifique sera toujours "1", et sera directement suivi par la virgule. 
- 2^3 = On décale 3 fois de plus la virgule vers la droite 


On peut à présent représenter le nombre à l'aide de la norme IEEE 754 
	*On partira du principe qu'on souhaite obtenir un "single", à savoir une valeur sur 32 bits* 

1. Le nombre est positif, le bit de signe sera "0" 
**0** 00000000 00000000000000000000000 


2. L'exposant (la puissance) est "3". On devrait logiquement écrire "00000011".  Cependant, un exposant peut être négatif ! 
	On calcule l'intervalle avec cette formule: D = (2^n-1) -1 
	n correspond au nombre de bits sur lequel on code l'exposant (Ici "8") 

On pose donc le calcul suivant:  
D = (2^8-1) - 1 
D = (2^7) -1 
D = 128 - 1 
D = 127 

Ainsi:  
- 127 correspond à "0" 
- Les nombres supérieurs à 127 seront des valeurs positives (128 = 1, 129 = 2, …) 
- Les nombres inférieurs à 127 seront des valeurs négatives (126 = -1, …) 


On sait que notre exposant vaut "3". On calcule 127 + 3 ce qui donne 130.
	*Astuce de représentation: Si le nombre est positif, on passe le bit de poids fort (Valant 128) à 1, et on complète les bits manquants. *


Ainsi, l'exposant décalé sera 130. On pose 130 sur les 8 bits 
0 **10000010** 00000000000000000000000 


3. Pour la mantisse de 23 bits, on place la partie fractionnaire en tant que bits de poids forts (sur la gauche), puis on complète les cases vides par des 0. 
	*A noter: On ne stocke QUE la partie fractionnaire, on exclue donc le "1" qui précède la virgule*

0 100000010 **100101**00000000000000000 


**Exemple 2 - 0d0,375 **

0d0,375 = 0b0,011
0b0,011 = 1,1x2⁻² 

1. Bit de signe. Le nombre est positif, donc 0
**0** 00000000 00000000000000000000000 


2. Exposant 

Exposant = -2, un nombre négatif. 
-2 = 127-2 
-2 = 125 

0 **01111101** 00000000000000000000000 


3. Mantisse 

On code la partie décimale (Ici, "1") 
0 01111101 **1**0000000000000000000000


****
## Représentations spéciales

Il existe des représentations à part, permettant de symboliser des éléments précis: 

| Valeur  | S   | E   | M     |
| ------- | --- | --- | ----- |
| Zéro    | 0   | 0…0 | 000…0 |
| +Infini | 0   | 1…1 | 000…0 |
| -Infini | 1   | 1…1 | 000…0 |
| NaN     | 0   | 1…1 | 010…0 |
*NaN (Not a Number) = Symboliser une valeur qui n'a aucun sens logique*
