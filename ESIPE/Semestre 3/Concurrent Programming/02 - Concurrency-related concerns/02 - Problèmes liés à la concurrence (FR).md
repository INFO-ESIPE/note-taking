[[Concurrence]]
****
**Table of Contents**
```table-of-contents
```

****
## Problème actuel

Les threads peuvent être **dé-schedulés en plein milieu d'un ensemble de lignes de code d'une fonction, mais surtout, en plein milieu d'une seule opération**. 
Cela est du au fait que la majorité des opérations sont dites **non-atomiques** (ou "non-synchronisée"). 

On a observé dans le TP que, toutefois, cette incohérence ne peut arriver avec `System.out.println` car la méthode utilise un **lock** qui est relaché une fois l'affichage complètement fini.
	*détails au prochain cours*


****
## Mémoire et Non-atomicité

On rappelle que les threads partagent un même espace mémoire (le tas). 
L'utilisation d'opérations non-atomiques entraine des problèmes conséquents: 
- Accès (lecture, écriture) entrelacés
- Optimisation du code par le JIT
- Utilisation de cache par le processeur


Une **data race** est une situation ou **plusieurs threads accèdent à une zone mémoire identique**:
```java
public class Counter {
	private int value;

	public void addALot() {
		for (int i = 0; i < 10_000; i++) {
			this.value++;
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

Dans cet exemple, la data race à lieu sur le champ `value` de l'objet `counter`.

Dans un monde idéal, `value` devrait valoir 20000. Mais ça ne sera — presque, genre vraiment c'est improbable hein — jamais le cas.
	*Théoriquement, on pourrait même se retrouver avec `value` qui vaut 2. Détails dans le TP.*
