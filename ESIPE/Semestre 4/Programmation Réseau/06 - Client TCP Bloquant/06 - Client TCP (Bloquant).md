[[Programmation Réseau]]

****
**Table of Contents**
```table-of-contents
```

****
## D'UDP à TCP

A l'inverse d'UDP, on va établir une connexion (lecture et écritue symmétrique) entre le client et le serveur. Le client est, généralement, celui qui initie la connexion. 

Cette connexion a **un canal pour lire depuis l'hôte distant**, et **un second canal pour envoyer à ce même hôte**. 
On peut fermer l'un de ces canaux sans que cela interrompe la connexion.
![[canals.png]]


Les garanties de TCP sont évidentes: **Fiabilité, Intégrité et non préservation des limites**(les données n'arrivent pas forcément en une fois).

Si on écrit 100 octets sur une connexion TCP, on pourra recevoir 40 octets puis 60 octets. On pourra aussi recevoir 100 octets. Si on écrit 50 octets puis 50 octets, le serveur pourra recevoir 100 octets en une seule fois.

****
## TCP côté client

Comme pour UDP, plusieurs clients vont interragir avec un serveur via un socket (adresse IP + port). 

Le socket TCP est représenté par un objet `SocketChannel` qu'on créé, comme pour le `DatagramChannel`, via une méthode factory `.open()`. 

On peut utiliser la méthode `.connect(SocketAddress)` sur le channel afin de se connecter:
```java
// creation of the socket (not connected)
SocketChannel sc = SocketChannel.open();

SocketAddress serverAddress = new InetSocketAddress("www.google.com",80);

// connection to server
sc.connect(serverAddress);
```

Une fois connecté, on peut obtenir (`sc.read()`) des données depuis le serveur, ou en envoyer (`sc.write()`). 

Ce comportement (et sa nomenclature) est identique que `FileChannel`. 

Comme pour le `DatagramChannel`, les deux méthodes sont **bloquantes**. Read bloque jusqu'a avoir reçu au moins 1 octet, et write bloque jusqu'a ce que toute la zone de travail ait été écrite.


Pour le read, si le buffer est plus grand que ce qui est récupéré, alors il ne sera pas rempli. **Si ce buffer est trop petit, les données ne seront pas perdues, elles sont mises de côtées et seront ré-accessibles au prochain read.**

La méthode renvoi le nombre d'octets lus, ou -1 si la connexion a été interrompue en écriture par l'interlocuteur *(note qu'on peut, donc, continuer à lui écrire mais qu'on ne recevra plus jamais rien de sa part):*
```java
var buffer = ByteBuffer.allocate(BUFFER_SIZE);
var read = sc.read(buffer);

if (read == -1) {
	System.out.println("Connection closed for reading");
} else {
	System.out.println("Read "+ read +" bytes");
}
```

Pour le write, la **totalité de la zone de travail du buffer est envoyée**. La méthode bloque jusqu'a l'envoi de toute la zone:
```java
var buffer = ByteBuffer.allocate(BUFFER_SIZE);
buffer.putLong(1l);
buffer.putLong(2l);
buffer.flip();
sc.write(buffer);
```

Fermeture du channel (et donc, de la connexion) : 
- Seulement le canal d'écriture: `shutdownOutput()`; 
    - L'interlocuteur est notifié, son read va renvoyer `-1`. 
        
- Seulement le canal de lecture: `shutdownInput()`; 
    - L'interlocuteur n'est pas notifié, opération purement locale 
        
- Tous: `close();` 
    - Libération complète des ressources systèmes


****
## Problématiques

Du à la non préservation des limites en TCP, on ne sait pas si on a bien tout reçu, et comment découper les données reçues.

C'est pourquoi il faut **définir notre propre protocole qui va uniformiser notre découpage et analyse des données que l'on reçoit**. TCP seul ne suffit pas, ce n'est qu'un protocole nous faisant parvenir les données.


On peut :
1. Fermer la connexion pour signaler la fin de l'envoi de données 
    - Cette méthode est valable uniquement si on a un seul bloc monolithique a envoyer. 
    - Plusieurs requêtes nécessite donc plusieurs connexions, ce qui va un peu a l'encontre de toute la logique de TCP 
    - Comment choisir la taille du buffer en réception? 
        
2. Utiliser un marqueur (suite d'octets) signalant la fin des données 
    - Si on a plusieurs données à envoyer, on marque la fin de l'échange de données dans le dernier paquet avec une suite spécifique (e.g. "\r\n" dans le header HTTP) 
    - Complexité à mettre en oeuvre du côté du récepteur 
    - Choix judicieux a faire pour définir cette chaine de rupture, cela exclut cette chaine de pouvoir apparaitre dans les données 
    - Comment choisir la taille du buffer en réception ? 
        
3. Commencer par transmettre la taille des données (principe des chunks en HTTP1.1) 
    - On préfixe le paquet par la taille des données pures (avec un long) 
    - Celui qui envoie doit connaitre la taille totale des données avant de les envoyer
