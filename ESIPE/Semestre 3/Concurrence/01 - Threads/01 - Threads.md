[[Concurrence]]
****
La concurrence est un concept permettant l'exécution de plusieurs morceaux de code en même temps. Ce concept s'utilise dans deux cas: 
- Appels I/O bloquants (requêtes base de données, API distante...), ce cas est le plus fréquent pour du dev logiciel et celui que l'on va traiter 
- Calcul massif nécessitant d'exploiter pleinement le CPU
	*Ici on préfère utiliser OpenMP ou MPI*


****
## Threads et processus

Lorsqu'un programme est exécuté, un processus est lancé par l'OS avec un espace mémoire dédié et un **fil d'exécution (thread)**.

Bien qu'un programme ait, par défaut un seul thread, **il peut en contenir plusieurs (d'ou l'intéret du cours).**


Chaque thread possède **sa propre pile d'exécution**, mais **partagent un même tas** (une même méoire centrale).
- La pile unique contient les paramètres et variables locales en Java. 
- Le tas partagé contient les champs des objets en Java.

Le **scheduler** est une entité gérée par l'OS qui décide de **l'ordre d'exécution des threads**. On utilisera la classe `java.lang.Thread` pour gérer tout ce qui y est lié. 

Tous les langages de programmation ont la même façon de mettre en place la concurrence. Aucun langage n'invente des concepts de concurrence, ils reprennent ce que l'OS et les processeurs fournissent. L'OS n'a, par ailleurs, pas réellement d'impact sur la concurrence de notre programme.


Un `Thread` ne prend aucun paramètre et ne renvoi rien, c'est donc un `Runnable`.
```java
@FunctionalInterface
public interface Runnable {
	/* public abstract */ void run();
}
```

On initialise et démarre le fil d'exécution ainsi (depuis Java 19):
```java
Runnable runnable = ...;
Thread thread = Thread.ofPlatform().start(runnable);
```


L'object est créé via la method factory `ofPlatform()`.
La création de l'objet ne lance pas le fil d'exécution. Pour cela, il faudra le démarrer via la méthode `start()`, qui va effectuer les actions suivantes :
- Allocation d'une nouvelle pile 
- Demande la création d'un fil d'exécution (thread de l'OS) 
- Le thread de l'OS exécute run() de Runnable 
- Une fois l'exécution de run() finie, le thread de l'OS meurt

L'objet java est mort une fois qu'il est libéré par le **garbage collector**.

Exemple:
```java
public class Application {
	private static void myRun() {
		System.out.println("Hello myRun");
	}

	public static void main(String[] args) {
		Runnable runnable = () -> {
			System.out.println("Hello main");
		};
		Thread thread1 = Thread.ofPlatform().start(Application::myRun);
		Thread thread2 = Thread.ofPlatform().start(runnable);
	}
}
```


Il n'y a **pas de relation père-fils** entre les threads (contrairement aux forks en Programmation Système). 
Si on lance un thread depuis le main, le main peut mourrir avant le thread concurrent, cela n'arrête pas l'autre thread. 

**Les threads ne communiquent pas entre eux**, ils ne peuvent pas se relancer entre eux (ni se relancer tout court).


****
## Scheduler

Afin de bien répartir les fils d'exécutions sur les différents coeurs du CPU, le scheduler est chargé de trouver un coeur libre afin d'y lancer le thread. 
Un thread occupe son coeur: 
- Jusqu'au prochain appel bloquant (I/O) 
- Pendant un quantum de temps maximum

Le scheduler est supposé donner le même temps de CPU à chaque thread, on dit qu'il est **"fair" (équitable)**.
	*En réalité, il est indiqué que le scheduler est équitable car il `donne du temps` de CPU a chaque thread, mais il n'est pas indiqué qu'il `donne le même temps` à chaque thread. Le quantum de temps est beaucoup trop variable pour parler de réelle équité entre threads.*

Étant donné que le scheduler de l'OS prend cette répartition à sa charge, **on ne contrôle rien**, ni l'ordre dans lequel les threads sont lancés, ni le temps pendant lequel ils vont s'exécuter.


Il existe, en java (et d'autres langages utilisant une nomenclature et un concept relativement variant), deux types de threads:
- **Thread platform**: Schédulés par l'OS, ce qui requiert une commutation de contexte (context switch: userland / kernel) 
- **Thread virtuel (expérimental depuis java 19)**: Schédulés par le scheduler de la JVM, ce qui a pour bénéfice de réduire les problèmes de latence.
	*Ces Threads sont beaucoup moins gourmands en ressources. Un programme peut avoir plusieurs milliers de threads virtuels sans soucis de performance, ce qui n'est pas possible avec des threads `ofPlatform`.*

C'est la même classe `Thread` qui gère les deux types. Si l'on souhaite créer des threads virtuels, on procède ainsi:
```java
Runnable runnable = ...;
Thread thread = Thread.ofVirtual().start(runnable);
```

On se contentera d'utiliser les threads `ofPlatform` pour ce cours.


Le scheduler JVM créé autant de threads plateforme que de coeurs et schedule les threads virtuels sur ces threads platform. On ne contrôle toujours rien sur le comportement du scheduler, c'est la principale caractéristique de la programmation concurrente. 

Un thread virtuel occupe un thread plateform: 
- Jusqu'au prochain appel bloquant (I/O) 
- Si un thread virtuel ne fait pas d'appel bloquant, alors un nouveau thread platform est créé pour que le scheduler reste fair


****
## Propriétés d'un Thread

On peut nommer un thread. Cela ne doit être fait uniquement **que pour du débug**, et jamais pour passer des valeurs au Thread comme un golmon
```java
var thread = Thread.ofPlatform().name("Listener thread").start(runnable);
```

On peut **récupérer le Thread qui est entrain d'exécuter le bout de code** :
```java
Thread.ofPlatform().name("Listener thread").start(() -> {
	var groot = Thread.currentThread(); // ici
	System.out.println("I am " + groot);
	System.out.println("My name is " + groot.getName());
});
```


Du fait qu'on ne gère pas du tout l'ordre d'exécution, dans le cas ci-dessous, les noms s'afficheront dans un ordre impossible à prédire :
```java
public static void main(String[] args) {
	Runnable runnable = () -> {
		System.out.println(Thread.currentThread().getName());
	};

	Thread.ofPlatform().name("Thread 1").start(runnable);
	Thread.ofPlatform().name("Thread 2").start(runnable);

	System.out.println(Thread.currentThread().getName());
}
```

*On ne sait pas lequel des trois threads va s'afficher en premier...*


****
## Mort d'un thread

On ne peut pas redémarrer ou tuer un thread (contrairement à un process), il est cependant possible de **lui demander gentilment, mais il n'y a aucune garantie** sur le fait qu'il s'arrête.

Un thread **meurt quand la méthode run() est finie**: 
- Soit parce qu'on a touché la fin de la méthode 
- Soit parce qu'on a return pour déclarer la fin de la méthode 
- Soit parce qu'une `Exception` est levée

La JVM ne s'arrête que si tous les threads sont morts, à l'exception des **threads daemons**. Le thread main peut s'arrêter bien avant les autres, cela ne ferme pas la JVM.
	*Un thread daemon est un thread qui, comme mentionné précédemment, n'empêche pas la JVM de s'arreter. 
	Il tourne comme un service. Si tous les threads non-daemon sont morts, alors il dégage et la JVM peut s'arrêter.*


Il est possible, pour le thread courrant, d'**attendre la mort d'un autre thread**, ce afin d'introduire un ordre d'exécution sur le code :
```java
// Noter l'utilisation du throws dans la signature !
public static void main(String[] args) throws InterruptedException {
	var thread1 = Thread.ofPlatform().start(() -> {
		System.out.println("Thread 1");
	});

	var thread2 = Thread.ofPlatform().start(() -> {
		System.out.println("Thread 2");
	});

	thread1.join();
	thread2.join();

	System.out.println("Thread main");
}
```

L'appel `thread.join()` peut lever une `InterruptedException`, erreur déclenchée si l'on demande a un thread de s'arrêter et qu'il est en attente dans sa méthode `join()`.


La méthode `Thread.sleep()` permet de **mettre en pause le thread** pendant n millisecondes. Tout comme join, la méthode est bloquante et susceptible de lever une `InterruptedException`.
Il faut, pour ce cas, **gérer l'exception avec un try-catch et une AssertionError** :
```java
Thread.ofPlatform().start(() -> {
	try {
		Thread.sleep(5000); // 5 sec
	} catch (InterruptedException e) {
		throw new AssertionError(e);
	}
	
	System.out.println("Done");
});
```

