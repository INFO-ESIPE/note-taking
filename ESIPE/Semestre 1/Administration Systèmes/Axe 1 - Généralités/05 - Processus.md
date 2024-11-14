[[Axe 1 - Généralités]]
04/10/2022
****
**Table of Contents**
```table-of-contents
```

****
## Processus

Comme mentionné précédemment, l'OS a pour but de mieux gérer et partager les ressources de l'ordinateur.

Ainsi, l'OS va faire croire à chaque tâche/application qu'elle a son propre processeur et sa propre mémoire vive, afin de simplifier les choses pour l'application. Cela va demander, cependant, au kernel de faire un certain travail. 

Le processus représente donc une application en cours d'exécution. Le noyau va donner à chaque processus l'illusion qu'il possède la totalité du hardware dont il a besoin.
	*Chaque processus possède donc son propre environnement d'exécution.*


Un processus a plusieurs caractéristiques : 
- Il possède un espace mémoire 
- Il peut utiliser des ressources (fichiers, périphériques…) 
- Ces droits dépendent de l'utilisateur qui a lancé le programme


****
## Modes d'exécution

Il est possible d'exécuter un processus en arrière plan en ajoutant l'opérateur `&` à la fin d'une commande :
```bash
sleep 30 &
```


La commande `bg` (ou le raccourci `Ctrl + Z`) permettent de placer le programme en cours d'exécution sur le shell en arrière-plan. 
La commande `fg` (foreground) permet de ramener le programme au premier plan. 

*Mettre un programme en arrière-plan s'avère utile au lancement de programmes volumineux et gourmand en ressources, par exemple.*


****
## Arborescence des processus sous Linux

Un processus est différent d'une "application". Plusieurs processus peuvent être liés à une même application.
	*Par exemple, Firefox tourne sur plusieurs processus*

Chaque processus est identifié par **un numéro unique (PID: Process Identifier)** 
Chaque processus possède un **parent (PPID: Parent Process Identifier)**
Chaque processus possède un propriétaire (utilisateur qui l'a lancé) 

Au démarrage du système, le processus **systemd (ou init)** est lancé. Il est — directement ou non — le parent de tous les processus du système.
	*Ce processus est responsable du lancement des autres processus
	Le PID **1** lui est réservé.* 


La commande `pstree` permet d'obtenir l'arborescence des processus et des **threads**. 
	*Les threads sont des simili-processus lancés par des processus actifs, afin de découper les tâches et ressources (en cas d'actions bloquantes, ou de calcul intensif). Plus de détails dans [[Concurrence|Le cours de programmation concurrente du S3]]*

La commande `ps` donne une liste plus complète, mais pas organisée de la même manière. 


L'appel système `fork()` va créer un nouveau processus fils en duplicant le processus parent. 
Les deux processus vont partager beaucoup de propriétés (même code, même fichiers ouverts si le parent à ouvert des fichiers...), mais le PID sera cependant différent. 
Le PID du nouveau processus est renvoyé par la fonction (au parent), 0 est renvoyé au fils, et -1 est renvoyé au parent en cas d'erreur. Les procesuss sont complètement isolés, et donc, les processus ne partagent pas les valeurs en mémoire (les variables etc). 


****
## Scheduling

Chaque processus pense avoir un espace CPU dédié à lui, cependant, le scheduler existe afin de répartir "équitablement" les coeurs du processeur.
	*Chaque thread/processus a besoin d'un coeur du CPU. Il y a des centaines de thread/processus actifs sur une machine, et il est peu ordinaire qu'il y ait autant de coeur disponibles. **On ne peut pas tous les maintenir actif constamment...***

On appelle cette méthode de partage du temps de CPU le **pseudo-parallélisme**. Le scheduler passe rapidement d'un processus à un autre ce qui donne l'illusion que chaque processus tourne sans discontinnu, ce qui est factuellement faux.


Le pseudo-parallélisme a lieu sur chaque coeur (sauf sur des architectures spécifiques ou il y a des coeurs réservés au kernel, ce qui n'est ni un bon choix ni l'objet du cours).

Un processus peut donc être dans 3 états différents: 
- **Running** - En cours d'exécution normale
- **Ready** - En attente d'être schedulé
- **Blocked** - Processus suspendu (sleep), processus en attente des ressources de la part du kernel...


****
## Mémoire

Chaque processus pense — comme pour ce qui est du CPU — posséder son propre espace de mémoire vive qui est partitionné depuis la vraie RAM physique (chacun à sa pile et son tas).

En réalité, **la mémoire vive est décomposée en mémoire virtuelle**, et chaque processus peut allouer des blocs au hasard parmi l'espace disponible. 
Chaque allocation va d'abord consulter un tableau qui fait le lien entre mémoire virtuelle et l'espace mémoire physique de la RAM. 
	*L'OS fait une **abstraction** de la mémoire vive.*


****
## Signaux

Les processus peuvent communiquer entre eux par envoi de messages (signaux). 
- Asynchrone 
- Nombre limité 
- Réactions prédéfinies 

Arrêter un processus: 
- **En douceur** - SIGINT (sid 2), équivalent à un `Ctrl + C`
- **Brutal** - SIGKILL (sid 9), ne peut pas être inhibé


La commande `kill -<sid> <pid>` permet, au final, d'expédier un signal à un processus en cours 
	*On peut lister tous les signaux disponibles avec `kill -l`*

*Note: `kill -9 <pid>` est très pratique*


****
## Commandes utiles

| Commande    | Objectif                                                                                                                                           |
| ----------- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| `top`       | Gestion des processus (organisé par celui qui consomme le plus de ressources CPU et Mémoire)<br>*Astuce: Btop++ et htop sont encore mieux*         |
| `kill`      | Tue un processus (et potentiellement les processus/threads dépendants du processus) <br><br>*Un processus peut refuser d'être tué*                 |
| `sigkill`   | Kill mais plus puissant (signal id 9)                                                                                                              |
| `free`      | Etat de la mémoire vive                                                                                                                            |
| `iostat -h` | Etat du processeur                                                                                                                                 |
| `nohup`     | Lancer une commande résistante aux déconnexions (hangups)<br><br>(Le processus continue d'exister même si le terminal est fermé par l'utilisateur) |
| `nice`      | Modifie la priorité (scheduling priority) d'un processus                                                                                           |
