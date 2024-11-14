[[Concurrence]]
****
**Table of Contents**
```table-of-contents
```

****
## Mutex

La non-atomicité des instructions ainsi que l'entrelacement des threads posent de gros problèmes qui nécessitent une solution permettant d'inclure une logique de synchronisation.

Cette solution est une **section critique** (lock, verrou). 


Le plan conceptuel le plus courrant et simple du verrou est le **mutex (MUTual EXclusion)**. 
Cette section critique permettrait de s'assurer que des threads différents ne puissent pas exécuter un même bout de code en même temps, et qu'on **essaye de lire une valeur avant que l'écriture soit totalement finie** (totalement répercutée sur la RAM et non conservée uniquement en cache ou dans une variable locale).

Cependant, **le lock ne garantit pas que le thread soit dé-schedulé en plein milieu de la section critique**. Il empêche effectivement que deux threads accèdent et exécutent un même bout de code en même temps, mais ne garantit pas l'écriture en RAM. Pour répondre à cette seconde contrainte, il sera nécessaire d'**appliquer un lock sur chaque action de lecture et d'écriture de la classe**.

*Pour notre `count` de la dernière fois :*
```java
public class Counter {
	private int value;

	public void addALot() {
		for (int i = 0; i < 10_000; i++) {
			// Début section critique
			this.value++;
			// Fin section critique
		}
	}

	public int value() {
		return this.value;
	}

	public static void main(String[] args) throws InterruptedException {
		var counter = new Counter();

		var thread1 = Thread.ofPlatform().start(counter::addALot);
		var thread2 = Thread.ofPlatform().start(counter::addALot);

		System.out.println("Final value: " + counter.value());
	}
}
```


****
## Bloc de synchronisation

Le bloc `synchronized` est une barrière en mémoire qui sert à empécher l'exécution concurrente d'un même bloc de code, pouvant résulter en un état incohérent des données de l'application.

Ce bloc **prend (acquire) un jeton** associé au moniteur avant l'exécution du code interne au synchronized, **et rend (release) ce jeton une fois sorti du bloc de synchro**. 

**Un seul thread peut avoir le jeton**. Si le thread est dé-schedulé au sein du bloc de synchro (et qu'il possède donc le jeton), **il le conserve** et aucun autre thread n'a donc la clef donnant accès au code.


**La synchro empêche également le JIT de déplacer les lignes de code** (au cours du processus d'optimisation de code) en dehors du bloc de synchronisation, et ce pour d'évidentes raisons. Les instructions peuvent être ré-organisées au sein du bloc même sans soucis, mais ça ne sort jamais du bloc.


La synchro ne garantit pas le bon comportement du code concurrent en dehors de ce même bloc. Si on effectue des opérations non-atomiques et non-synchronisées depuis un main (e.g :
```java
System.out.println(user.nom() + " " + user.prenom())
```
), alors la synchro perd tout son intéret et le code va potentiellement ouvrir sur des erreurs de concurrence. 
Il est donc important, même si la classe est bien conçue et thread-safe, de **respecter les principes de concurrence en dehors de la classe**.


On constate donc que la synchro n'empeche pas le dé-scheduling — et donc un état incohérent des données — mais va cependant empêcher aux autres d'avoir accès à cet état incohérent. On a donc l'impression que l'état n'est jamais incohérent ce qui est en réalité faux, mais on ne tombera jamais sur cet état car c'est impossible.


****
## Mutex - Implémentation et utilisation en Java

On ajoute à la classe qu'on souhaite rendre thread-safe un `Object` qui sert de lock, aussi appelé **"moniteur"** :
```java
public class Counter {
	private int value;
	private final Object lock = new Object();

	public void addALot() {
		for (int i = 0; i < 10_000; i++) {
			synchronized(lock) { // Début section critique
				this.value++;
			} // Fin section critique
		}
	}
}
```

Le moniteur choisi se doit de respecter certaines conditions: 
- Il doit être un **Objet**, donc pas primitif (int, long...) ni null 
- L'objet ne doit **pas utiliser d'interning** (String literals, Integer), afin que le même objet serve de verrou pour différents besoins 
- Le lock ne doit pas pouvoir être utilisé par d'autres threads et va donc respecter certains principes d'encapsulation: 
    - Le champ servant de moniteur doit être **private** car il ne doit pas être accessible en dehors de la classe 
    - Le champ doit être **final afin que sa référence ne change jamais** 
    - L'objet doit être **créé dans le constructeur** de la classe 
    - On **ne fournit jamais un accesseur vers le moniteur** 
- L'objet doit servir de référence commune


Les verrous en Java sont dit **"ré-entrants"**, un même thread peut prendre plusieurs fois le même lock :
```java
public class Foo {
	private int value;
	private final Object lock = new Object();

	public void setValue(int value) {
		synchronized(lock) {
			this.value++;
		}
	}

	public void reset() {
		synchronized(lock) {
			setValue(0);
		}
	}
}
```


****
## Interning

On désigne par interning une optimisation évitant des allocations de mémoire superflues pour des objets représentant les mêmes valeurs. 
	*Si il existe cinquante String "Bob", alors le JIT alloue un espace mémoire pour la toute première, mais va ensuite renvoyer une **référence/représentation canonique** (équivalent à un pointeur C) vers le tout premier String bob, afin de ne pas allouer cinquante fois la même chose.*

Le fonctionnement profond est le suivant : 
	Il existe une méthode "intern()" sur la classe String qui effectue l'action mentionnée ci-dessus. Il existe donc, au sein du programme, une piscine de chaînes de caractères qui est maintenue par la classe String. Cet ensemble est initialement vide.
	Quand la méthode d'intern est appelée, on regarde si la string existe dans la piscine (via la méthode `equals()`, surtout pas `==` hein...)
	S'il s'avère qu'elle est présente dans l'ensemble, on renvoi la string originelle, celle dans la piscine, on n'alloue rien. Sinon, on ajoute la string à la piscine et on retourne une référence à l'objet.


Le JIT effectue, par défaut, de l'interning sur les objets de la classe `String` et `Integer`. Cela est fait implicitement, c'est pourquoi on n'utilise jamais réellement la méthode `interning()`, car le JIT le fait pour nous.


****
## Classe "Thread-Safe"

Une classe thread-safe est une classe **pouvant être utilisée par plusieurs threads sans jamais avoir d'état incohérent** (définition floue).  
Comme mentionné ultérieurement, le thread-safe garantit la bonne conception de la classe elle-même, mais pas sa bonne utilisation à l'extérieur. 

La majorité des classes Java ne sont pas thread-safe (`HashMap`, `ArrayList`...). Cela va donc radicalement changer notre façon de concevoir notre code. Les seules classes Java thread-safe sont celles du package `java.util.concurrent`. 
	*Note: un record (ou toute autre classe non-mutable) est donc, par définition, thread-safe*
