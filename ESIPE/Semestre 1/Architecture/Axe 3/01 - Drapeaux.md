[[Axe 3]]
10/10/2022
****

À tout moment lors de l'exécution d'un programme, le **registre de drapeaux (flags)** est un registre de 32 bits qui contient des informations cruciales sur l'état de la dernière instruction exécutée.


Les **drapeaux** sont des bits individuels qui stockent des informations binaires, chacune indiquant un état particulier sous forme de **booléens** (1 pour "oui"/vrai, et 0 pour "non"/faux).

Chaque bit dans ce registre représente une information spécifique sur le résultat des opérations effectuées par le processeur. Le registre de drapeaux agit comme un ensemble d'**indicateurs** permettant de savoir, par exemple, si une opération a généré un dépassement de capacité, si une valeur est nulle, etc.


**Exemple d'utilisation - Instruction de comparaison :**

L'instruction de comparaison **`cmp`** est couramment utilisée pour **comparer deux valeurs**. Cette instruction effectue en réalité la soustraction **VAL_1 - VAL_2** sans stocker le résultat, mais elle **met à jour le registre de drapeaux** en fonction du résultat de cette opération.

Après l'instruction **`cmp VAL_1, VAL_2`**, les drapeaux sont modifiés de la manière suivante :
- **Si `VAL_1 - VAL_2 = 0`** : Le drapeau **ZF** (_Zero Flag_) est positionné à **1**.
- **Si `VAL_1 - VAL_2 > 0`** : Le drapeau **ZF** est mis à **0** et le drapeau **CF** (_Carry Flag_) est à **0**.
- **Si `VAL_1 - VAL_2 < 0`** : Le drapeau **ZF** est mis à **0** et le drapeau **CF** est positionné à **1**.

**ZF (Zero Flag)** : Vrai si le résultat de l'opération est nul
**CF (Carry Flag)** : Vrai s'il y a eu un **emprunt** (retenue de sortie) lors de l'instruction
**SF (Sign Flag)** : Vrai si l'instruction produit un résultat négatif
**OF (Overflow Flag)** : Vrai si l'instruction produit un dépassement de capacité


Un descripteur de taille (byte, word, dword) peut être ajouté afin de préciser la taille des valeurs :
![[flags.png]]
