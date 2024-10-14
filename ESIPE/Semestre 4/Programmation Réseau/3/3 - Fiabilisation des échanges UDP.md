[[Programmation Réseau]]
****

UDP, à l'inverse de TCP, n'offre aucun moyen d'éviter les pertes ou de garantir l'ordre d'arrivée des trames. Ce n'est pas fiable. 
Ce problème est relativement invisible en local, mais devient très problématique en client-serveur distant. De plus, il est impossible de connaitre les raisons de la perte de données (la requete est perdue, ou la réponse est perdue, ou le paquet est juste retardé/lent …). 

Nous allons utiliser un proxy (relais) pour introduire artificiellement de l'instabilité dans le réseau :
![[proxy.png]]

Le seul mécanisme possible est de renvoyer le paquet au bout d'un certain délai jusqu'a recevoir une réponse de la part du destinataire. Mais combien de temps attendre ?

****
## UDP en Java

Comme évoqué précédemment, on passe par un `DatagramChannel` pour communiquer via UDP, et on utilise la méthode d'instance `receive()` pour recevoir de **manière bloquante**. 

Le problème est donc que si l'on est **bloqué en attente, on ne peut plus rien faire ni rien renvoyer**. Le code du TP2 fonctionne donc en local, mais est défectueux s'il communicait en dehors de cette limite. 

Il va donc falloir passer par de la **concurrence** pour régler le problème : 
- Un thread d'envoi  
- Un thread de réception

On va, plus précisément, passer par une logique **producteur/consommateur** (vu en cours de concurrence au s3, c'est le truc le plus facile en plus) afin d'éviter l'attente active et le surplus de données stockées : 
- **File d'attente protégée commune** aux deux threads (parralèle, comme OpenMP un peu)
- Consommateur mis en attente **si rien à consommer dans la file** 
- Producteur mis en attente **si plus de place dans la file**


On peut, grâce à une `BlockingQueue`, borner l'attente du consommateur (celui qui extrait de la file d'attente les réponses des requêtes) :
![[queue.png]]

****
## Méthodologie

Le programme se comportera ainsi :
![[behaviour.png]]


Il reste cependant des points à éclaircir et ou il est important de se questionner : 
- Implémentation de la BlockingQueue 
    - ArrayBlockingQueue ou LinkedBlockingQueue ? 
    - Quelle taille ? 
        
- Deux threads pour un seul channel 
    - Qui arrête quoi ? 
    - Qui ferme quoi ? 
        
- Partage de la zone de stockage (mémoire) 
    - Les deux threads accèdent à la meme zone mémoire, **attention à la data race**

