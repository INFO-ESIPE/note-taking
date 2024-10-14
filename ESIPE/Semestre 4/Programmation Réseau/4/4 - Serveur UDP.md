[[Programmation Réseau]]

****

Un client UDP va, comme pour la plupart des applications sur un ordinateur personnel, s'attacher à un port aléatoire libre (supérieur à 1023).  

Un serveur UDP va devoir, lui, attacher son `DatagramChannel` à un port bien précis afin de standardiser les échanges :
![[bind.png]]

Si le port 7777 n'est pas libre, la méthode lève une `BindException`. On ne peut pas libérer un port depuis Java, il faut donc **manuellement fermer le serveur qui occupe le port**.


On remarque que `bind()` prend ici un `InetSocketAddress` au lieu d'un simple port, pourquoi ? 
**Un serveur n'initie jamais de contact, il ne fait que réagir aux requêtes des clients.**

Le serveur est donc composé d'une boucle très simple : 
1. Réception de la requête venant d'un client 
2. Construction de la réponse 
3. Envoi de la réponse au client


Exemple avec un serveur echo :
![[echo.png]]

****
## Stateless et Statefull

On dit qu'un protocole est **stateless** quand la réponse à une requête d'un client ne dépend que de cette même requête, et donc, d'aucune de ses requêtes précédentes. 
*(Tous les protocoles qu'on a vu jusqu'ici dans ce cours sont stateless.)*

A l'inverse, un protocole est **statefull** si la réponse à une requête dépend de ses précédentes requêtes.


Pour faire un serveur statefull, il faudra donc que le serveur mémorise les informations sur les paquets reçus pour chaque client, sans les mélanger ! 
Le serveur maintient les informations dans la collection suivante : 
```java
HashMap<InetSocketAddress, ClientData>
```


**ClientData** est une classe stockant les informations nécessaires. 
Il est cependant primordial de stocker le moins d'informations possibles et de mettre en place certains mécanismes afin de libérer la mémoire occupée par les informations stockées dans la Map.


Exemple avec un serveur count statefull :
![[count.png]]

