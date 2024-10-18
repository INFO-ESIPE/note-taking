[[Axe 2]]
****
## Langage bas niveau

Un langage bas niveau est un langage ayant une forte dépendance à la structure matérielle de la machine. 

**Avantages**: 
- Gestion parfaite des ressources matérielles 
- Gestion parfaite de la mémoire 
- Exécution rapide 
    
**Inconvénients**: 
- Pas d'abstraction 
- Pas de programmation "rapide" et "facile"


****
## Langage machine

Le Langage machine est un langage binaire compris par le processeur en vue d'être exécuté. 
	*Chaque processeur possède son propre langage machine.*

Prenons deux modèles de processeurs (P1 et P2): P1 est compatible avec P2 si toute instruction formulée par P2 peut être comprise et exécutée par P1. 


En général, une instruction commence par un **opcode** permettant de définir la nature de l'instruction. 
Cet opcode est suivi des suites de bits codant les **opérandes** de l'instruction. 
	*Exemple: 01101010 00010101 
	Ici, l'opcode est le premier octet 0b01101010. 
	L'opérande est donc la suite, 0b00010101. 
	Cette instruction ordonne de placer l'opérande (21)dix en tête de pile.*


****
## Langage d'assemblage

Le langage d'assemblage (ou assembleur) est un langage a mi-distance entre le programmeur et la machine. 
- La machine convertit très simplement et rapidement un programme en assembleur vers le langage machine souhaité. 
- L'Assembleur est un langage "facile" à comprendre et à manipuler par l'homme 
	*Facile par rapport à du binaire ...*


Le but final est, au travers d'une traduction littérale (mot par mot), de passer de l'assembleur au langage machine très simplement. 
Les opcodes sont codés via des **mnémoniques**, à savoir des mots-clefs bien plus compréhensibles et manipulables que des suites binaires. 

Un langage d'assemblage est dédié en général à un seul processeur. Il y a donc, plus ou moins, **autant de langages d'assemblage que de processeurs**.


****
## Assemblage et désassemblage

L'assemblage est la **traduction d'un programme en assembleur vers le langage machine** 
Le désassemblage est **l'opération inverse** qui est menée par un désassembleur.

![[disas.png]]
