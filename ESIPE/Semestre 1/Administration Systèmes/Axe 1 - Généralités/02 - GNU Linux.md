[[Axe 1 - Généralités]]
30/09/2022
****

Linux est le noyau de l'OS GNU/Linux 
GNU/Linux est l'implémentation libre de l'OS UNIX (développé par AT&T dans les années 60) en 1991 par Linus Torvalds

La première distro Debian arrive en 1993 
Le GUI GNOME arrive en 1997 
Arrivée du projet commercial Ubuntu (Basé sur Debian) en 2004 
Microsoft commence à contribuer au noyau Linux en 2009 (jusqu’à nos jours) 

Il est possible de faire fonctionner certaines appli Windows avec Wine. 


Une distribution est munie d'un Noyau Linux possédant une version spécifique 
La version est notée sous la forme suivante: x.y.z 
- x: Version de kernel 
- y: Version Majeure 
- z: Version Mineure 
*`uname -r` pour avoir la version*


****
## Gestion de paquets

Chaque distribution a son propre gestionnaire (apt, pacman ...).
On utilisera debian pour ce cours, donc on passera par `apt`.

| Apt install | Installer un paquet                             |
| ----------- | ----------------------------------------------- |
| Apt update  | Mise à jour de la liste des paquets disponibles |
| Apt upgrade | Mise à jour des paquets                         |

Il existe des Versions LTS (Long-Term Support) qui arrivent tous les deux ans. Elles promettent des màj sur une dizaine d'années. Cela est souvent réservé aux serveurs.


Les dépôts contenant les paquets sont stockés sur des sites web officiels et miroirs. 
Les dépôts possèdent un type de paquet :

| Main      | Paquets libres                                                |
| --------- | ------------------------------------------------------------- |
| Non-free  | Paquets non libres                                            |
| Contrib   | Paquets libres mais ayant des dépendances en dehors de "main" |
| Backports | Dernières versions encore en développement                    |


****
## Debian

Debian est un OS Universel depuis 1997 
- Debian fonctionne sur de nombreuses architectures (processeurs variés, Raspberry, Arduino…) 
- Noyau Linux

Debian est entièrement composé de logiciels libres 
`lsb_release -a` donne la version de la distribution


****
## Pages `man` (manual)

Trois catégories (man \<val> blabla): 
- 1: Commandes shell 
- 2: Appels systèmes 
- 3: Appels librairie

*Si le chiffre est omis, le manuel des commandes shell (1) est présenté.*


**Keybinds pratiques :**
![[binds.png]]

L'avantage du manuel — comparé à une recherche google — est d'être sur qu'il est raccord avec notre OS
	*Qu'on ne tombe pas sur un man BSD si est sur Arch*


****
## Notes

Le shell (terminal) émule les terminaux physiques de l'époque. 
L'éditeur de l'époque est **ed**.
