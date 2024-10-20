[[Axe 2 - Administration basique]]
04/10/2022
****

Les fichiers de log sont en général situés dans le répertoire `/var/log`.

Les fichiers de log principaux: 

| Emplacement                  | Objectif                                                       |
| ---------------------------- | -------------------------------------------------------------- |
| /var/log/syslog              | Contient les logs essentiels au système (erreurs, connexions…) |
| /var/log/proftpd/proftpd.log | Contient toutes les connexions au serveur profrpd              |
| /var/log/auth.log            | Contient la liste des connexions des utilisateurs (Debian)     |

Pour ajouter facilement une ligne à un fichier de log: 
`echo "blabla" >> /var/log/le_log`

