[[Programmation Réseau]]

****
## Serveur TCP

Comme pour l'UDP, on commence par configurer le `socketChannel` en nonblocking:
```java
socket.configureBlocking(false);
```

Ainsi, les méthodes `accept()`, ainsi que `read()` et `write()` sont désormais non-bloquants et s'exécutent instantanément sans attendre.

On passera une fois de plus par le sélecteur, et chaque nouvelle connexion sera surveillée par le sélecteur:
```java
var selector = Selector.open();

var ssc = ServerSocketChannel.open();
ssc.bind(new InetSocketAddress(7777));
ssc.configureBlocking(false);

// register to selector for accepts
SelectionKey serverKey = ssc.register(selector,SelectionKey.OP_ACCEPT);
```
> On remarque la nouvelle opération `OP_ACCEPT` qu'on n'avait pas vu jusque la.


Traitement des clefs:
```java
void treatKey (SelectionKey key) {
	if (key.isValid() && key.isAcceptable()) {
		doAccept(key);
    }
    if (key.isValid() && key.isWritable()) {
        doWrite(key);
    }
    if (key.isValid() && key.isReadable()) {
        doRead(key);
    }
}
```

On passe par une boucle de traitement:
```java
while (!Thread.interrupted()) {
	selector.select(this::treatKey); // method reference is cleaner
}
```

Pour laquelle on oublie pas de throw une `UncheckedIOException` étant donné qu'on passe par une lambda. 
Dans un cas sérieux, on passe par des contextes pour stocker et manipuler les informations relatives à chaque connection (sc). On obtient le treatKey suivant:
```java
private void treatKey(SelectionKey key) {
	try {
	} catch (IOException ioe) {
		// lambda call in select requires to tunnel exception with Unchecked
		throw new UncheckedIOException(ioe);
	}

	try {
		if (key.isValid() && key.isWritable()) {
			((Context) key.attachment()).doWrite();
		}
		if (key.isValid() && key.isReadable()) {
			((Context) key.attachment()).doRead();
		}
	} catch (IOException e) {
		logger.log(Level.INFO, "Connection closed with client due to IOException", e);
		silentlyClose(key);
	}
}
```
> For details on tunneling, see [[07 - Exceptions#Tunneling|this]].

On remarque que la gestion des exceptions est coupée en deux:
- L'`IOException` du `doAccept` qui n'est pas supposé arriver et qui indique un problème grave. On l'attrape plus haut et on la log comme sévère:
```java
try {
	selector.select(this::treatKey);
} catch (UncheckedIOException tunneled) {
	logger.severe("UIOE thrown, exiting");
	throw tunneled.getCause();
}
```

- Les `IOExceptions` dues à un client, bénignes. On se contente de logger ça en tant qu'information et de fermer le client par une méthode `silentlyClose` qui détruit la connexion sans lever d'exception:
```java
private void silentlyClose(SelectionKey key) {
	var sc = (Channel) key.channel();
	try {
		sc.close();
	} catch (IOException e) {
		// We ignore the exception and do nothing (silent)
	}
}
```



