[[Programmation Réseau]]

****
**Table of Contents**
```table-of-contents
```

****
## Client TCP

On utilise également les **sélecteurs pour les clients TCP Non-bloquant**, avec les memes méthodes d'utilisation (boucle de sélection, contextes ...) 

On ajoute cependant deux nouveautés: 
- **Multithread**: Le thread responsable de la boucle de sélection devra interragir avec les autres threads 
    - Thread de lecture au clavier (action bloquante donc nécessairement dans un thread à part) 
    - Threads de l'interface graphique (UI), ce qui n'est pas traité dans ce cours 
        
- Gestion de connexion en mode non-bloquant avec la méthode `socketChannel.connect()`, qui n'est plus bloquante.

****
## Connexion

On passe donc, comme auparavant pour le client bloquant, par socketChannel.connect() pour initier une connexion. Cependant, au lieu de bloquer jusqu'a l'établissement de connexion, la méthode initie la procédure de connexion et passe à la suite immédiatement. 

Il existe une méthode `socketChannel.finishConnect()` qui renvoi un booléen pour indiquer si la connexion est établie. 
Si le sélecteur est enregistré en `SelectionKey.OP_CONNECT`, alors on est notifié quand la connexion est établie.

Ici on enregistre en `OP_CONNECT`:
```java
var sc = SocketChannel.open();
sc.configureBlocking(false);
sc.connect(serverAddress);
var sKey = sc.register(selector, SelectionKey.OP_CONNECT);
```

Tant que ce status est actif, on appelle la méthode `doConnect(key)` car la connexion n'est toujours pas validée.  

Comme pour les autres opérations, le sélecteur n'est pas 100% fiable (voir doc des sélecteurs) alors on vérifie que le signalement est bel et bien correct en appelant `finishConnect()`:
```java
private void doConnect(SelectionKey key) throws IOException {
	if (!sc.finishConnect()) {
		return; // the selector gave a bad hint, it happens
	}
	key.interestOps(SelectionKey.OP_READ);
}
```

Si la connexion est avérée (finishConnect renvoi `true`), alors on change notre champ d'intéressement (ici on passe à la lecture).


****
## Actions et Multithread

On effectue une boucle de sélection similaire à celle du serveur non-bloquant, mais on y remplace doAccept par doConnect pour gérer les connexions en mode non-bloquant pour client:
```java
while (!Thread.interrupted()) {
	selector.select(this::treatKey);
}

// ...

void treatKey(SelectionKey key) {
	if (key.isValid() && keu.isConnectable()) {
		doConnect(key);
	}
	if (key.isValid() && keu.isWritable()) {
		doWrite(key);
	}
	if (key.isValid() && keu.isReadable()) {
		doRead(key);
	}
}
```
> On gère les exceptions comme il faut, bien entendu (voir cours précédent).


**Le problème majeur est le suivant**: Les informations que l'on va envoyer via le thread principal (qui gère la boucle) vont provenir d'ailleurs (d'autres threads, celui responsable de lire au clavier en général).

### Les sélecteurs ne sont pas thread safe ! 
Si on doit modifier les interestOps des clefs ou faire un register, il faut nécessairement le faire **dans le thread qui exécute la boucle de sélection**.


Il ne faut jamais utiliser de `blockingqueue` pour faire communiquer le thread console et le thread du sélecteur. Notre code va ajouter une entité a traiter (un message) dans la blockingqueue, puis va réveiller le selecteur avec `selector.wakeup()`, sauf que le JIT peut inverser les deux lignes, et va, potentiellement, réveiller avant d'ajouter le travail (pas bon du tout). 

Le but sera donc de faire une classe (avec des méthodes) **thread-safe** car, bien que les lignes peuvent toujours être inversées dans le `synchronize`, cela n'impacte pas la logique finale étant donné que le lock est gardé jusqu'a la fin. Le problème ne se manifèstera pas car le tout s'exécutera qu'à la sortie du `synchronize`.
