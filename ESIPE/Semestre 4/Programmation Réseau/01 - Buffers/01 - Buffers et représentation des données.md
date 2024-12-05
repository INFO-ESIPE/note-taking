[[Programmation Réseau]]
****
**Table of Contents**
```table-of-contents
```

****
## Introduction

Tout ce qui s'échange sur le réseau prend systématiquement la forme d'une suite d'octets. 

La moindre action nécessite un certain nombre d'informations afin que chaque acteur puisse correctement interpréter les valeurs échangées. Tout ceci est défini par le protocole utilisé. 

Par exemple, pour échanger un entier, il faut savoir sur combien d'octets cela sera représenté, si l'entier est signé, et l'endianness (droite -> gauche ou inverse).
![[endian.png]]

- BigEndian: Octet de poids fort en premier :
	- 61 E2 82 AC -> 1642234540
- LittleEndian: Octet de poids faible en premier :
	- 61 E2 82 AC -> AC 82 E2 61 -> 2894258785

Java ne sait pas traiter les entiers non-signés (vu que c'est de la merde en gros). De ce fait — et pour le projet — les entiers seront toujours signés :))))))) 

Astuce pour le projet: représenter les entiers comme en Rust (i8, i16, i32, i64)

****

Une chaines de caractère nécessite qu'on connaisse le charset (jeu de caractères) utilisé : 
- ASCII: 1 octet = 1 caractère 
- UTF-8: Le nombre d'octets peut varier 
- ISO-8859-1 (ISO Latin 1)

Java utilise **UTF-16 en interne pour représenter les char sur 2 octets** (16 bits), attention.

****
## java.nio.bytebuffer

La logique voudrait qu'on utilise des tableaus d'octets `byte[]` pour manipuler les valeurs échangées sur le réseau, mais on ne doit jamais faire ça car le garbage collector a tendence à y toucher sans notre concentement. 
**Les tableaux de bytes sont "garbage collectés".** 

Il existe, cependant, une classe **ByteBuffer** qui remplacent ces tableaux de bytes, tout en nous donnant accès à un ensemble de méthodes bien pratiques pour le manipuler au mieux.


Le concept est similaire à un tableau d'octets (avec une capacité limite), mais on y ajoute deux indications qui définissent la zone de travail: 
- Position: Début de la zone de travail (case inclue) 
- Limite: Fin de la zone de travail (case exclue)
![[buffer_basic_representation.png]]

La method factory `ByteBuffer.allocate(int capacity)` créé un bytebuffer. 
**Par défaut, position = 0 et limit = capacity.** 
La zone de travail est donc toujours continue (il n'y a pas de trous dans la zone de travail qui sortent de cette même zone).

On doit faire un `ByteBuffer.allocateDirect(int capacity)` dans le cas ou **l'on utilise un buffer qui va rester toute la durée du programme.** Si le buffer est temporaire, on préfère utiliser un allocate standard. Donc, si on créé un buffer volatile dans une boucle juste pour une simple opération, on n'utilise **pas** allocateDirect. 

Cela vient du fait que l'allocateDirect n'est pas garbage collecté, car, à l'inverse du allocate, elle sera gérée directement par l'OS. Donc, on évite d'instancier des objets (allouer de la mémoire) qui ne sera pas gérée par le garbage collector pour des raisons d'optimisation.


**Opérations de base:**
- `put(byte)` : Écrit l'octet b sur la position 
- `get()` : Lit et retourne l'octet sur la position

Ces méthodes font avancer la position de 1 octet. Si la position dépasse la limite, `BufferOverflowException` :
![[buffer_evolution.png]]

La méthode `flip()` (bien retenir ça, c'est le truc le plus important du cours) permet de lire ce qu'on vient d'écrire en plaçant la limite à la position actuelle et en replaçant la position à 0 :
![[buffer_flip.png]]

On peut recaler au début du buffer tout ce qui n'a pas été consommé et qui est dans la zone de travail via `compact()` :
![[buffer_compact.png]]


**Autres méthodes utiles :
- `remaining()` : Donne la taille de la zone de travail (nb accès restants via put/get) 
- `hasRemaining()` : Si il reste de la place ou non (remaining > 0) 
- `position()` : Donne la position courante  
- `position(int pos)` : Fixe la position courante 
- `limit()` : Donne la limite courante  
- `limit(int pos) `: Fixe la limite courante 
- `clear()` : Remet le buffer comme neuf (vidé, position 0 et limite à maxpos)

**Des alternatives pour les types primitifs existent :** `putInt, putLong, getInt, getShort` ...


On peut modifier l'endianess avec la méthode `order(ByteOrder)`. 
`ByteOrder.nativeOrder()` donne l'ordre natif de la plate-forme, qui devrait être `BigEndian`.
*(On s'en sert jamais en vrai, pas super important)*

****
## Charset

Le **charset** est l'association d'un code (sur un ou plusieurs octets) pour chaque caractère. 
**L'encodage** est la traduction d'une suite de caractère en une suite d'octets : 
	String -> Byte[] 

Le décodage est l'opération inverse: 
	byte[] -> String 


Un objet `java.nio.charset.Charset` représente ce jeu de caractères : 
```java
Charset charset = Charset.forName("UTF-8"); 
// ou 
Charset UTF8 = StandardCharsets.UTF_8; // plus pratique
```


On peut encoder et décoder facilement un ByteBuffer avec les méthodes d'instance du même nom : 
```java
ByteBuffer buffer = charset.encode(String s); 

String str = charset.decode(bb).toString();
```


Ces approches — simplistes — ont cependant des inconvénients majeurs : 
- `encode()` va créer un bytebuffer à chaque appel 
- `decode()` doit contenir tous les octets à décoder dans le bytebuffer passé en paramètres. Si l'on veut décoder un fichier de 2 Gb, le buffer devra contenir ces 2 Gb.

On peut, à la place, utiliser les classes `CharsetEncoder` et `CharsetDecoder`:
```java
charset.newDecoder()
	.onMalformedInput(CodingErrorAction.REPLACE)
	.onUnmappableCharacter(CodingErrorAction.REPLACE)
	.decode(buffer);
```


L'état de sortie est indiqué par un objet de la classe `CoderResult`.  
Les sorties suivantes sont possibles : 
- **Underflow :** il n'y a plus d'entrée à traiter, ou l'entrée est insuffisante. 
- **Overflow :** un débordement est signalé lorsque la place restante dans le buffer de sortie est insuffisante. 
- **Malformed-input error :** une erreur d'entrée mal formée est signalée lorsque la séquence d'entrée n'est pas bien formée. 
- **Unmappable-character error :** une erreur de caractère non applicable est signalée si une séquence d'entrée désigne un caractère qui ne peut être représenté dans le jeu de caractères de sortie.

****

## FileChannel

Permet de lire et écrire des octets à partir d'un fichier:
```java
Path path = Paths.get("~/test.txt");
// open in read-mode
try (FileChannel fc = FileChannel.open(path, StandardOpenOption.READ)) {
	// ...
}

// open in write-mode
try (FileChannel fc = FileChannel.open(path,
				     StandardOpenOption.CREATE,
				     StandardOpenOption.WRITE,
				     StandardOpenOption.TRUNCATE_EXISTING)){
	// ...
}
```


La méthode `fc.read(ByteBuffer bb)` lit depuis le fichier via le canal fc. La valeur de retour **se comporte comme scanf**, elle retourne le nombre d'octets lus et -1 si on atteint la fin du fichier.

***Important:*** Le nombre d'octets que `read()` va prendre est variable, cependant, cette valeur dépend systématiquement de la zone de travail actuelle. Il est donc impossible qu'un read prenne plus d'octets que l'espace disponible, par contre, il reste indispensable de le vider de temps en temps (flip + write + clear).


La méthode `fc.write(ByteBuffer bb)` écrit les `bb.remaining()` octets du buffer dans le canal.
Exemple basique:
```java
var path = Paths.get(args[1]);
var buff = ByteBuffer.allocate(Integer.BYTES);  // 4 bytes
try(FileChannel fc = FileChannel.open(path, StandardOpenOption.CREATE, 
        StandardOpenOption.WRITE, StandardOpenOption.TRUNCATE_EXISTING);
        Scanner scan = new Scanner(System.in)) {
    while (scan.hasNextInt()) {
        buff.putInt(scan.nextInt());
        buff.flip();
        fc.write(buff);
        buff.clear();
    }
}
```

Exemple optimisé:
```java
var path = Paths.get(args[1]);
var buff = ByteBuffer.allocate(BUFFER_SIZE);
    
try(var fc = FileChannel.open(path, StandardOpenOption.CREATE, 
        StandardOpenOption.WRITE, StandardOpenOption.TRUNCATE_EXISTING);
        var scan = new Scanner(System.in)) {
    while (scan.hasNextInt()) {
        if (buff.remaining()<Integer.BYTES){
            buff.flip();
            fc.write(buff);
            buff.clear();
        }
        buff.putInt(scan.nextInt());
    }
    buff.flip();
    fc.write(buff);
}
```


