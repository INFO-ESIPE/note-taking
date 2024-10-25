[[Axe 2]]
02/10/2022
****

Un programme assembleur est un fichier texte possédant une extension **.asm**
Il est composé de différentes parties ayant un but précis: 
- Invoquer des directives 
- Définir des données initialisées 
- Réserver de la mémoire pour des données non initialisées 
- Contenir une suite d'instructions

Exemple de programme :
```assembly
% include io.asm ; directive

section .data
c1: db 'A' ; donnée initialisée

section .bss
ta: resd 10 ; données réservées

section .text
main:
	push c1 ; instruction
	add eax, 4
```


****
## Expression des valeurs

**Nombres**: 
- Base dix: 0, 10020, -90 
- Hexadécimal: 0xA109C2FF, -0x98 
- Binaire: 0b001, 0b1111011 
    
**Caractères**: 
- Directement: 'a', '5', "Z' 
- Code ASCII: 10, 120, 65 

**Chaines de caractères**: 
- Directement: 'bonjour', 0 
- Caractère par caractère: 'a', 'b', 45, 0 
    
**Important**: "0" est le **marqueur de fin de chaine de caractères** (équivalent à "\0" en C). Une chaine devra donc impérativement finir par une valeur de 0 (sans cotes)


****
## Registres

Un registre est un emplacement de 32 bits interne au processeur (ou 64 bits sur une architecture x64). 

Un registre est considéré comme une variable globale. 

Pour une architecture 32bits, il y a un total de 4 registres de travail, les **General-Purpose Registers (GPRs)**: 
- E**A**X - **A**ccumulateur, souvent utilisé pour des opérations mathématiques, ou pour contenir le résultat (return) d'une opération
- E**B**X - **B**ase, contient souvent des paramètres de fonction
- E**C**X - **C**ompteur, souvent utilisé pour les boucles (équivalent à la variable `i`)
- E**D**X - **D**ata, usage générique 
	*Les quatres registres principaux (E**X**X) peuvent être utilisés comme souhaité, mais les conventions veulent que l'usage indiqué soit celui à suivre quand on le peut.
	Il existe encore 4 autres registres\* (ESI, EDI, EBP et ESP), mais on va faire genre que y'a que ceux-la pour l'instant.*


Chacun est donc représenté sur 32 bits. Cependant, chacun de ces registres peuvent être découpés en plus petits registres :
![[registry.png]]
	*Prenons ECX :
	ECX = Le registre comprenant les 32 bits 
	CX = Les 2 octets de poids faible de ECX 
	CL = L'octet de poids faible
	CH = Le second octet (celui qui suit CL)*

En fait, ces registres de 32 bits sont eux-mêmes des sous-registres pour ceux de 64 bits *(RAX, RBX, RCX et RDX)* — qu'on ne verra pas ici, puisqu'on va étudier l'architecture x86.
	*La modification d'un **sous-registre** (AX, AH et AL dans le cas de EAX) impacte évidemment le registre global*



\* Informations sur les registres manquants (32bits / 64bits) :

| Nom                          | Variable        | Rôle                                                                                                                                                        |
| ---------------------------- | --------------- | ----------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Pointeur d'instruction       | EIP / RIP       | Contient l'**adresse de la prochaine instruction à exécuter**                                                                                               |
| Pointeur de tête de pile     | ESP / RSP       | Contient l'**adresse actuelle du sommet de la pile (stack pointer)**                                                                                        |
| Pointeur de base de pile     | EBP / RBP       | Utilisé pour contenir l'**adresse d'une donnée dans la pile**, souvent pour référencer des variables locales ou des paramètres de fonctions.                |
| Registre de drapeaux         | EFLAGS / RFLAGS | Contient des informations sur le résultat de la dernière opération (comme les drapeaux de dépassement, zéro, ou de retenue) ainsi que des bits de contrôle. |
| Registre source d’index      | ESI / RSI       | Utilisé principalement comme **pointeur source** pour les opérations de copie ou de manipulation de chaînes.                                                |
| Registre destination d’index | EDI / RDI       | Utilisé principalement comme **pointeur destination** pour les opérations de copie ou de manipulation de chaînes.                                           |
