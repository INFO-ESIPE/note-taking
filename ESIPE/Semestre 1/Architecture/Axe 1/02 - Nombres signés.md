[[Axe 1]]
****
Pour représenter un nombre négatif, on ajoute un bit de signe. Ce bit sera de poids fort (à gauche). 
	0 = Positif 
	1 = Négatif

Si un des entiers nécessite moins de n bits pour être codé, on complètera avec des 0 entre le bit de signe et ceux liés à sa valeur.


Cette manière de représenter un entier en le codant en adjoignant un bit de signe est appelé la magnitude signée. 
Sur 8 bits, on a : 
- 45 = 0b**0**0101101 
- -45 = 0b**1**0101101

Si le nombre est trop grand pour pouvoir permettre cette méthode, il ne rentre évidemment pas dans la plage.

Cette méthode est relativement concise et simple.


****
## Complément à un

La méthode consiste à inverser tout les bits d'un nombre afin d'obtenir sa valeur négative 
La négation d'une valeur binaire est la valeur inverse de chaque bit. 

Exemple pour 24: 
	**0**0011000 
	**1**1100111


****
## Complément à deux

Sur 8 bits, on a la valeur u= 0 110 1000 
- La première étape consiste à effectuer un complément à un (inverser les valeurs de chaque bit) 
- La seconde étape consiste à ajouter 1, ce qui permet de décaler les valeurs (addition binaire): 
    1 110 0111 +1 =  
    1 110 1000 

Ce procédé permet de nous débarrasser du 0 en double 
Cela permet de réaliser une addition dont le résultat est compris entre -128 et 127.  
Si ce résultat dépasse, on créer un débordement (overflow).
