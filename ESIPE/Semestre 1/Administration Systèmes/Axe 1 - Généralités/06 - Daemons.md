[[Axe 1 - Généralités]]
05/10/2022
****
## Daemon

**"Daemon"** est synonyme de **"service"**.
Un service est un processus qui apporte des fonctionnalités : 
- Aux utilisateurs d'une machine 
- Aux autres machines sur le réseau (service distant)

Ils ont pour caractéristiques: 
- De tourner sans discontinu 
- D'être indépendant des utilisateurs 
- Ils ne sont pas associés à un terminal
- Ils ont pour parent le processus init
- Ils attendent qu'on leur demande quelque chose (comme un serveur)

Les scripts de démarrage/arrêt des démons sont présents dans le répertoire `/etc/init.d`


On appelle un service par son path, en ajoutant une option, ou alors via la commande `systemctl` (System control) : 
- `start`, `stop`, `restart`, `reload` (mettre à jour sans redémarrer) 

Il existe des services directement utiles à la machine: 
- Services de gestion de réseau 
- Mise à jour de l'heure avec un serveur de temps (NTP) 
    *En France, ce serveur est basé sur Paris (UTC+1)*
- Planificateur de tâche (CRON) 
- Gestion de disques avec redondance en RAID (mdadm) 
- Annuaire LDAP 
- Serveurs web/ftp
- ...


**Lister les services activés :** 
`systemctl list-util-files | grep -i "enabled"`

**Activer ou désactiver un service au démarrage :** 
`sudo systemctl enable <service>.service`