[[Programmation Réseau]]

****
**Table of Contents**
```table-of-contents
```

****
## Bloquant vers Non-bloquant

Par défaut, les méthodes `send()` et `receive()` du `DatagramChannel` sont bloquantes. Nous avions gérer ce problème en passant par des threads. 

Il est cependant possible de rendre le dc non-bloquant:
```java
DatagramChannel dc = DatagramChannel.open();
dc.configureBlocking(false);
```


****
## DatagramChannel non-bloquant

En mode non-bloquant, l'appel à `receive` **retourne immédiatement, même si aucun paquet n'a été reçu par le système** : 
- Si paquet présent: Les données sont copiées dans la zone de travail du bb et l'adresse de l'expéditeur est renvoyée 
- Si pas de paquet: Le buffer n'est pas modifié et l'appel renvoi null
```java
var dc = DatagramChannel.open();
dc.configureBlocking(false); 

var exp = dc.receive(buff);
if (exp != null){
    System.out.println("Packet received !");
} else {
    System.out.println("No packet received");
}
```


En mode non-bloquant, l'appel à `send` **retourne immédiatement, même si on ne peut pas envoyer le paquet sans délai dans le buffer système a cause de manque de place** : 
- Si assez de place : Les données de la zone de travail du bb sont consommées et envoyées dans le buffer système 
- Si pas assez de place : Le buffer n'est pas modifié et aucune donnée n'est envoyée
```java
var dc = DatagramChannel.open();
dc.configureBlocking(false); 
// ...
dc.send(buffer, sender);
if (!buffer.hasRemaining()){
    System.out.println("Packet sent!");
} else {
    System.out.println("No packet sent.");
}
```


On peut être tenté de se mettre en attente de réception de données lorsqu'on est en mode non-bloquant, de cette façon:
```java
var buffer = ByteBuffer.allocate(BUFFER_SIZE);
var dc = DatagramChannel.open();
dc.configureBlocking(false); 

InetSocketAddress sender = null;
while(sender == null) { // oops
  sender = dc.receive(buffer);	 
}
```

==A ne jamais faire ! Attente active …==

Comment faire alors ? On utilise un **nouveau mécanisme appelé "Sélecteur"** qui va nous prévenir quand un paquet est arrivé (`receive()`) ou que l'envoi est possible car assez de place (`send()`).


****
## Sélecteur

Le `java.nio.channels.Selector` est un mécanisme qui, comme évoqué précédemment, permettant **d'être notifié lorsqu'un paquet arrive et/ou quand on peut en envoyer un.** 

Le sélecteur va **surveiller un ou plusieurs** `DatagramChannel(s)` **qui sauront enregistrés auprès de lui.** 

Quand on interroge le sélecteur, **ce dernier renvoi l'ensemble des** `DatagramChannel` **sur lesquels au moins une des deux opérations est possible :**
![[ESIPE/Semestre 4/Programmation Réseau/05 - UDP Non-bloquant/selector.png]]


On créé un Selector avec la method factory `Selector.open()`.

Le selector passe par la classe `SelectionKey` pour stocker les informations relatives à un `DatagramChannel` qu'il surveille. 
Un Selector **maintient deux ensembles de** `SelectionKey` : 
- Les clefs surveillées : `selector.keys()`
- Les clefs sélectionnées : `selector.selectedKeys()`, sur lesquelles un évènement surveillé a été détecté


Chaque `SelectionKey` contient : 
- Un Channel qui correspond au DatagramChannel enregistré pour cette clef, accessible en lecture via l'accesseur `selectionKey.channel()` 
- Un integer qui code les opérations pour lesquelles on souhaite être notifié, consultable via l'accesseur `selectionKey.interestOps()` 
    - `SelectionKey.OP_READ` pour la réception seulement 
    - `SelectionKey.OP_WRITE` pour l'envoi seulement 
    - `SelectionKey.OP_READ | SelectionKey.OP_WRITE` pour l'envoi ET la réception.

On peut modifier les opérations d'intéret via la méthode `selectionKey.interestOps(int interestOps)`.
