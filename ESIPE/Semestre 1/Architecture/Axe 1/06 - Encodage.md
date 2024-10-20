[[Axe 1]]
02/10/2022
****
## American Standard Code for Information Interchange (ASCII)

ASCII est un encodage introduit dans les années 60. En ASCII, un caractère est représenté sur un octet **avec un bit de poids fort valant systématiquement 0** (donc, factuellement les valeurs sont représentées sur 7 bits). Les valeurs de l'octet seront traduites en hexadécimal.

Tableau ASCII:
![[ascii.png]]

La première partie de l'octet (partie verticale du tableau) sera codée jusqu’à une valeur de 
0\x7. 
La seconde partie ira jusqu’à 15 (0xF), et représentera la partie horizontale.


****
## Latin-1 (ISO 8859-1)

Un caractère reste codé sur un seul octet, cependant, le bit de poids fort ne possède plus une valeur fixée sur 0. 

Ce codage permet donc d'inclure plus de valeurs (surtout des caractères avec accents, typique des langues européennes originaires du latin, d’où le nom)/


****
## UNICODE

Unicode est une version étendue du codage ASCII. Il apparait en 1991. 
Cette fois-ci, un caractère est représenté sur 2 Octets.
	Avantage: Permet d'encoder des caractères latins, mais aussi cyrilliques, grecs … 
	Problème: Un caractère prends deux fois plus de place


****
## UTF-8

UTF-8 est la solution moderne au problème.
Il offre deux possibilités: 
1. Si le caractère à représenter est un caractère ASCII, alors il sera représenté sur un octet par son code ASCII 
2. Sinon, il sera représenté par son code Unicode. Il sera cependant codé sur une valeur allant de 2 à 6 octets.

Avantages:  
- Un texte UTF-8 peut être traduit en ASCII et inversement 
- Il est impossible d'avoir un octet ayant une valeur de 8 "0" (Pas de gaspillage) 

Problème: 
- Un caractère (hors ASCII) pouvant avoir une taille allant de 2 à 6 octets, il sera nécessaire de gaspiller quelques bits afin de mettre en place la forme et le nombre d'octets.


**Exemple:**

"A" 
A pouvant être représenté en ASCII, il sera représenté sur 7 bits (et le bit de poids fort valant toujours 0) 
	*A = **01000001***


"è": 
è peut être encodé sur 2 octets. 
Pour préciser le nombre total d'octets qui sera utilisé (car nous sommes hors ASCII), on le représente au début du premier octet. 

**110**00000 **10**000000 
	*Le nombre de "1" sur le premier octet indique le nombre total d'octets. Ici, il y a deux 1, donc deux octets.*
	**Ces "1" seront toujours suivi d'un seul "0" **

Chaque octet qui suit le premier commencera par **0b10**, afin d'indiquer que ce dernier suit l'octet précédent afin d'encoder un caractère Unicode. 

On place ensuite les valeurs sur les bits vides, que l'on réunit en une seule et même valeur: 
	**110**00000 **10**000000 = 00000 + 000000 
	= 00000000000 (11 bits libres) 

Donc :
	*è = 110**00011** 10**101000***


"ਗ਼"
Ce caractère sera encodé sur 3 Octets. 

Le premier octet aura donc 3 "1" suivi d'un "0". Chaque octet suivant ce dernier débutera par un "1" suivi d'un "0" :
	**1110**0000 **10**000000 **10**000000 

On code la valeur sur 16 Bits (4+6+6) 
	*ਗ਼ = 1110**0000** 10**101001** 10**011001***