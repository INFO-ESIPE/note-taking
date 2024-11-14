[[Programmation Réseau]]

****
## Serveur TCP

Comme pour l'UDP, on commence par configurer le `socketChannel` en nonblocking :
![[ESIPE/Semestre 4/Programmation Réseau/08 - Serveur TCP Non-bloquant/nonblock.png]]

Ainsi, les méthodes `accept()`, ainsi que `read()` et `write()` sont désormais non-bloquants et s'exécutent instantanément sans attendre.

On passera une fois de plus par le sélecteur, et chaque nouvelle connexion sera surveillée par le sélecteur :
![[ESIPE/Semestre 4/Programmation Réseau/08 - Serveur TCP Non-bloquant/selector.png]]

On remarque la nouvelle opération `OP_ACCEPT` qu'on n'avait pas vu jusque la.


Traitement des clefs :
![[keys.png]]


On passe par une boucle de traitement :
![[ESIPE/Semestre 4/Programmation Réseau/08 - Serveur TCP Non-bloquant/loop.png]]

Pour laquelle on oublie pas de throw une `UncheckedIOException` étant donné qu'on passe par une lambda. 
Dans un cas sérieux, on passe par des contextes pour stocker et manipuler les informations relatives à chaque connection (sc). On obtient le treatKey suivant :
![[ESIPE/Semestre 4/Programmation Réseau/08 - Serveur TCP Non-bloquant/treatkey.png]]


On remarque que la gestion des exceptions est coupée en deux:
- L'`IOException` du `doAccept` qui n'est pas supposé arriver et qui indique un problème grave. On l'attrape plus haut et on la log comme sévère :
	![[severe.png]]

- Les `IOExceptions` dues à un client, bénignes. On se contente de logger ça en tant qu'information et de fermer le client par une méthode `silentlyClose` qui détruit la connexion sans lever d'exception :
	![[benign.png]]


