[[Axe 3]]
****
La pile est une zone de la mémoire dans laquelle des données peuvent être **empilée** (`push`) ou **dépilées** (`pop`).

Comme mentionné précédemment dans le cours, le registre **ESP contient l'adresse de la tête de pile**, et **EBP permet de sauvegarder une position** dans la pile.

![[stack.png]]


La position exacte de la pile dépend essentiellement de l'OS et de l'architecture matérielle.
Sur Linux en x86, la pile commence à l'adresse **0xC0000000** et descend vers les adresses plus basses (proche de **0x00000000**)*
	*On peut observer la range de la pile — et du tas — d'un processus ainsi:*
```bash
nvim /proc/<pid>/maps
```


La taille maximale dépend également de plusieurs facteurs. Sur Linux, la pile est en général limitée à **8 Mo** (ce qui est peu, d'ou l'intéret d'utiliser le tas dans certaines situations).
On peut vérifier la taille exacte avec la commande suivante :
```bash
ulimit -s
```


****
## Manipulation de la pile

Chaque valeur dans la pile — pour cette architecture — sera stockée sous forme de **double mots (dword)**, qui ont une taille de **4 octets**.

**Empiler :** `push VAL`
*Ceci va décrémenter l'adresse stockée dans ESP de 4, et écrit la valeur VAL à cette nouvelle adresse.*

**Dépiler :** `pop REG`
*Ceci recopie les 4 octets stockées à l'adresse pointée par ESP dans le registre REG donné, puis incrémente ESP de 4.*


**Exemple :**

On à le code suivant :
```assembly
push 0x3
pop eax
pop ebx
push eax
```

Avec une pile dans cet état :
![[stack example.png]]


1. `push 0x3`:
![[stack example 1.png]]


2. `pop eax`
![[stack example 2.png]]


3. `pop ebx`
![[stack example 3.png]]

4. `push eax`
![[stack example 4.png]]