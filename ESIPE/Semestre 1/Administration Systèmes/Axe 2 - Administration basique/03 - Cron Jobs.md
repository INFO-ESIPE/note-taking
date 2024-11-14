[[Axe 2 - Administration basique]]
11/10/2022
****
**Table of Contents**
```table-of-contents
```

****
## Cron

Cron est le service qui gère la **planification des tâches**. 

Ces tâches sont variées (sauvegardes, mises à jour, lancement de programmes), et s'exécute à un instant précis et un rythme précis.
	*`crontab` est le programme qui permet de programmer ces actions régulières sur la machine*


Pour influer sur les cronjobs, on passe par l'édition des tables de configuration de cron.


****
## Commandes Crontab (individuel)

Le démon cron lit le fichier `/var/spool/cron/crontabs/<user>` et exécute les commandes qui s'y trouvent 
	*Ces fichiers sont donc reliés à un seul utilisateur.*

Les commandes dans ce fichier seront lancées en fonction des privilèges de l'utilisateur propriétaire de la Crontab (en général l'utilisateur a qui la Crontab est dédiée). 

| Commande   | Action                        |
| ---------- | ----------------------------- |
| Crontab -e | Créer ce fichier              |
| Crontab -r | Supprimer ce fichier          |
| Crontab -l | Visualiser toutes les Crontab |


Un utilisateur lambda ne pourra pas exécuter la commande `reboot` via une Crontab.


****
## Cron (global)

Placer un script .sh dans un des répertoires suivants fonctionne également: 
- /etc/cron.daily 
- /etc/cron.weekly 
- /etc/cron.monthly 

Le fichier /etc/crontab permet de gérer ces fichiers ci. 
![[cron.jpg]]

**Ces cron jobs seront exécutés pour chaque utilisateur.**


****
## Commande `at`

Cette commande permet de lancer une commande pour plus tard; à un jour donné, à une heure donnée, et ce via une commande à la syntaxe relativement simple. 


L'avantage de `at` est qu'il ne laisse aucune trace d'exécution. 
	*Une fois que cette commande est exécutée, elle n'existe plus. Il permet donc de créer un job planifié, mais qui ne sera lancé qu'une seule et unique fois.*
