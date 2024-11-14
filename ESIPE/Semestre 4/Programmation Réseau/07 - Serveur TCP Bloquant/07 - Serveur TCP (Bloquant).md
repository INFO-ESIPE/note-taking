[[Programmation Réseau]]

****
**Table of Contents**
```table-of-contents
```

****
## Serveur Itératif

Le serveur TCP va, comme pour le serveur UDP, écouter sur un port bien précis. On va attacher ce port à une `ServerSocketChannel`, qui va permettre d'accepter une connexion avec la méthode `ssc.accept()`. 

On ferme le channel avec `ssc.close()`, rien de nouveau ici.


Le channel de base **ne peux rien faire d'autre qu'accepter une connexion**. Dès que'une connexion est acceptée (méthode bloquante tant que personne ne tente de se connecter), la méthode renvoi un `SocketChannel` qui représente le client.

Il est important de **catch les exceptions de communication pour un client**, afin que ça ne casse pas tout le serveur en propageant l'erreur. 
Si un client se déconnecte (parce qu'il passe sous un tunnel ou une bétise du genre), ce n'est pas nécessaire de fermer tout, on va juste avoir une `IOException` (plusieurs genres possibles) qu'on va catch et log. 

*Comme pour un fast-food, on sert actuellement les clients un par un, si l'un est impoli et part en pleine commande, ce n'est pas la peine de fermer tout le restaurant.*


**Il y a une abstraction d'acceptation des clients par l'OS**. Étant donné qu'on va traiter les clients un par un, il peut y avoir de nouveaux clients qui arrivent pendant qu'on en traite un. **L'OS va accepter le client sur l'OS socket TCP/IP mais le code (la JVM) ne l'accepte pas encore. Il est en état de connexion pendante (pending).**


Pour servir un client accepté, on effectue des read/write comme on a déja vu pour les `SocketChannel`. 
On suivra le patterne basique suivant pour traiter les clients en mode bloquant :
![[pattern.png]]

**A noter**: Si c2 se connecte pendant qu'on sert c1, il est en état pending et est en attente d'accept par notre code (et donc la JVM). Cependant, si on prend trop de temps a accepter, l'OS va interrompre la connexion qu'il a accepté avec le client, il va le timeout (car il en déduit qu'un délai trop élevé à été dépassé et que le serveur n'est pas en état de traiter sa demande).

Puisque l'OS garantit cette sécurité de timeout (qui, en général, est la pour empecher des attaques par flux…), **on n'accepte que les demandes qu'on est totalement prêt à gérer**. On n'accepte jamais toutes les connexions qu'on va stocker en attendant de blabla… L'OS va gérer ça pour nous.

****
## Serveur Concurrent

Dans la vraie vie, on ne fait jamais de serveur itératif car cela ne respecte pas complètement la notion de serveur, vu qu'on traite les clients un par un et pas en même temps

On privilégie un serveur concurrent qui s'articulant ainsi: 
- 1 thread (manager) global qui va accepter les clients 
- 2 threads par client accepté: 
    - 1 Thread reader 
    - 1 Thread writer
![[concurrent.png]]

Dès qu'un client se déconnecte, **on tue les threads du client**. 
*En gros, deux employés macdo vont servir un client, et dès que le client est servi on tue les deux employés. C'est triste mais c'est comme ça qu'on va faire pour l'instant.*

Dans la vraie vie, on va capper un nombre max de threads pour l'application globale et on va ré-utiliser les threads (dès que le couple de thread a servi un client, on le définit comme étant prêt à en servir un autre). On parle de **Fixed pre-started**.

****
## Threads Virtuels

Dans le futur, on passera par des **threads virtuels**, qui sont "jetables" et peu couteux. On pourra faire des millions de threads pour un coup minimum sur l'ordinateur. 
Les threads virtuels fonctionnent moyennement a l'heure actuelle, si on introduit des instructions "synchronized" dans le code, le thread virtuel fonctionne mal et sera transformé en thread normal, ce qui pose un énorme problème problème :
	Si l'on fait 100_000 threads virtuels (ce qui est possible, car peu couteux en ressources) et que ces threads se transforment en threads normaux en plein milieu de l'exécution de notre programme, tout explose. 

En attendant que ce comportement soit revu, on se contente de threads normaux beaucoup plus fiables et moins expérimentaux.

