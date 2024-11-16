[[Axe 1 - Généralités]]
30/09/2022
****
**Table of Contents**
```table-of-contents
```

****
## Système d'exploitation

### Role

L'OS est la couche intermédiaire entre les applications et la machine physique (le matériel).
De ce fait, l'OS assure l'**aspect de portabilité** (indépendance par rapport au hardware)

L'OS assure la concurrence entre les différentes applications/utilisateurs sur l'OS afin de partager les différentes ressources sans collision. 
On a deux types de ressources: 
- **Ressource spatiales**: Mémoire, disque (HDD, SSD...) 
- **Ressource temporelle**: Durée "équitable" (fair) d'accès au CPU pour chaque application


### Typologie

**Multi-tâches** - Arrivé vers 1960, Permet une exécution simultanée de plusieurs programmes. 
	*Tous les systèmes modernes le sont désormais.*

**Multi-utilisateurs** - Conçu pour être utilisé par plusieurs utilisateurs 
- En général via un réseau
- Utilisé surtout pour les serveurs 
- Sécurisation des connexions

**Multiprocesseurs** - Conçu pour exploiter une machine munie de plusieurs processeurs 

**Temps réel** - Garantit que les opérations seront effectuées en respectant un délai strict 
	*Utilisé en particulier dans le domaine de l'industrie ou de l'aéronautique*


### Composition

**Interface de programmation** - Mise à disposition d'une API (Application Programming Interface), un ensemble de fonctions que l'on peut appeler afin de signifier à l'OS ce qu'on souhaite réaliser — et qu'il le face à notre place.

**Ordonnanceur** - Contrôle du déroulement des programmes, Exécution simultanée (Multi-tâche)

**Communication inter-processus** - Echange d'informations entre les processus 

**Gestion de la mémoire**
- Allocation: Réservation de mémoire 
- Protection: La mémoire est utilisée uniquement par le programme qui l'a réservé (empêche la réécriture) 
- Pagination: Décomposer la mémoire en zones de taille fixe 

**Mémoire virtuelle (Swap)** - Simuler la présence de mémoire centrale en utilisant un autre type de mémoire (HDD par exemple), permet d'Exécuter plus de programmes que ce que la mémoire centrale permet 

**Pilotes** - Contiennent les instructions permettant d'utiliser certains périphériques 

**Système de fichiers** - Structure arborescente dans laquelle sont stockés les données 

**Réseau** - Niveaux 1 (Physique) à 4 (Transport) du modèle OSI 

**Contrôle d'accès** - Authentification (Identification des utilisateurs) et Autorisations (Droit d'accès sur les fichiers) 

**Interface Utilisateur** - Fournit une interface Graphique (GUI) ou texte (CLI) 

**Logiciels Utilitaires** - Logiciels fournis par défaut avec l'OS 
- Interpréteur de commandes 
- Installation & Mise à jour 
- Surveillance de l'utilisation 
- Configuration 
- Droit d'accès 
- …


****
## Noyau

Le noyau est un ensemble de programmes formant la base minimale de l'OS, possédant un espace de la mémoire qui est protégé.

Certains choix sont fait, à savoir de choisir si certaines fonctionnalités sont inclues dans le noyaux (noyaux riche ou non).
	*Exemple: le GUI 
	- Windows: Intégré au système 
	- Linux: Programme utilisateur (Serveur X)*


### Architecture

Il existe différent types d'architecture pour les noyaux :
- **Monolithique**: Tout les fonctionnalités du système (Système de fichiers, Pilotes …) sont **exécutées dans l'espace noyau**. 
	*FreeBSD, Linux* 

- **Micro-Noyau**: Contient seulement le strict minimum (Ordonnanceur, Mémoire Virtuelle). Les autres services (File System, Pilotes) **fonctionnent dans l'espace utilisateur** (hors du noyau). 
	*Mac OS X, Mimix, QNX (pour les systèmes embarqués)* 

- **Hybride** - Mi-chemin entre le monolithique et micro-noyau. Il garde l'approche monolithique pour sa performance, mais permets à certaines composantes d'être exécutées dans l'espace utilisateur (comme le micro-noyau).
	*Windows NT (Noyau Win10 & 11)*

- **Exo-Noyau**: Expérimental, ucune protection de la mémoire (Cas très particuliers, notion hors cours). Très performant mais risqué ...
	*ExOS, Nemesis*


### Modes d'exécution

Deux modes d'exécution: 
- **Mode utilisateur**: Dédié — en général — aux applications, shells et librairies, interdit certaines instructions liées au CPU, pas d'accès direct à la mémoire (Virtual memory à la place), et les crashs ne font pas planter le système (SIGSEGV, SIGFPE...).

- **Mode kernel (ring 0)**: Dédié au noyau (kernel), aucune limitation d'accès à la mémoire et au CPU, les erreurs peuvent faire planter la machine. 

*Note: Ce qui est inclus ou non dans l'espace kernel/utilisateur dépend de l'architecture du Kernel ([[#Architecture|cf. partie sur l'architecture des noyaux]])
Pour plus de détails sur les CPU Protection rings, voir [[01 - Hardware Virtualization#CPU Virtualization|ici]].*


On va constamment passer d'un mode à l'autre de manière tacite, ce qui occasionne des ralentissements. 
	**Exemple :** Dans le cas ou l'on ouvre un fichier, le système prend implicitement la main pour effectuer l'action *(réserve des ressources...)*, et permet ensuite à l'application dédiée d'ouvrir le fichier ciblé.


Afin de garder une persistance des données entre le passage d'un état à l'autre **(switch)**, les **données doivent être copiées temporairement à un autre endroit de la mémoire**, avant d'être re-placées à l'endroit d'origine une fois les opérations en mode kernel terminées, c'est pourquoi cela s'avère si lent.


****
## Appels Systèmes (syscalls)

Comme mentionné auparavant, on utilise une **API** pour indiquer à l'OS ce qu'on souhaite faire afin qu'il le fasse à notre place. 
	*L'OS rend service aux applications.*
	
Le but est d'éviter les ralentissements du swap tout en permettant un ensemble d'opérations utiles et nécessaires aux applications. 
Une application peut effectuer des appels systèmes directement, mais en général, on passe par une librarie qui donne un set de fonctions correctement structurées et qui fait le travail à notre place 
	*Fonction `printf()` qui va effectuer un appel système `write` sur le **system call handler***

![[syscall.png]]
	*L'appel "syscall" effectue la bascule vers le mode kernel.
	Les méthodes autres que `syscall` sont des **wrapers** qui vont effectuer — dans leur implémentation — l'appel systeme pour l'utilisateur. On peut observer ce comportement avec la commande `strace ./fichier_binaire` qui va lister tous les appels systèmes effectués par le binaire* 


Exemple plus concis :
![[rings.png]]

Plus de détails sur les syscalls dans le cours de Programmation Systèmes du S3.


****
## Timeline UNIX

**1964 - Multics**
Multics est une collaboration entre le MIT, General Electrics et Bell Labs ayant pour objectif de créer un Système d'exploitation se rapprochant d'un système "cloud", à savoir un OS stocké sur un serveur accessible a distance. 

Le projet était très visionnaire pour l'époque. Le projet ne s'est pas concrétisé par mauvaise estimation de la charge de travail, ce qui a poussé Bell Labs à se retirer du projet. 

Bell Labs était le laboratoire de la companie téléphonique de l'époque (monopole total). L'entreprise ayant beaucoup de moyens, elle offrait des moyens assez conséquents à ce laboratoire, ce qui a permis à deux employés (Ken Thompson & Dennis Ritchie) de faire toutes les expérimentations qu'ils souhaitaient. 


**1969 - UNIX**
Les deux ont développé l'OS UNIX, réalisant que l'ordinateur qu'ils utilisaient manquait de fonctionnalités (Editeur de texte, applications diverses, compilateur...). 

Contrairement à Multics (multi-utilisateur), UNIX (qui sonne un peu comme "unique") ne permet pas de concurrence entre plusieurs utilisateurs, il n'y en a donc qu'un seul. 

UNIX s'est surtout répendu dans des universités au début et ce gratuitement en demandant aux deux développeurs, Bell Labs n'ayant pas le droit de commercialiser les systèmes informatiques. On peut estimer que cette logique est à la base de la philosophie "Open-Source". 


**1983 - GNU (GNU Not Unix) Project** 
UNIX étant devenu propriétaire et payant entre temps, Richard Stallman à lancé le projet GNU qui, contrairement à son nom, à pour but de ré-implémenter UNIX (création de bash, gcc...). 


**1987 - Minix**
Un mini-UNIX permettant d'apprendre plus simplement aux étudiants — en cours de système d'exploitation — comment marche un OS. 


**1991 - Linux**
Linus Torvalds, ayant accès à tous les outils de GNU mais n'ayant pas les moyens d'acheter un OS UNIX, a développé son propre OS pour son ordinateur. Il a donc développé un kernel Open-Source sous la license GNU.  


*Pour — beaucoup — Plus de détails: [https://commons.wikimedia.org/wiki/File:Unix_history-simple.svg](https://commons.wikimedia.org/wiki/File:Unix_history-simple.svg)*


****
## Familles d'OS

### Famille UNIX
- **Distributions GNU/Linux** - Systèmes d'exploitation basés sur le noyau Linux, avec différentes distributions adaptées à divers usages.
    - **Fedora** : Distribution innovante sponsorisée par Red Hat, souvent à la pointe des nouvelles technologies.
    - **Ubuntu** : Distribution populaire, orientée vers l'utilisateur grand public, maintenue par Canonical.
    - **Debian** : Distribution stable et robuste, base d'Ubuntu, appréciée pour ses qualités de serveur.
    - **Arch Linux** (la meilleure avec NixOS) : Distribution rolling-release minimaliste et hautement personnalisable, visant les utilisateurs avancés.
    - **CentOS** : Version gratuite et communautaire de Red Hat Enterprise Linux (remplacée par CentOS Stream en 2021).
    - **Red Hat Enterprise Linux (RHEL)** : Distribution commerciale orientée entreprises et serveurs.
    - **Linux Mint** : Distribution facile d'utilisation, dérivée d'Ubuntu avec un environnement de bureau convivial (pour les débutants).
    - **SUSE Linux** : Distribution orientée entreprises, avec une version openSUSE pour la communauté.
- **Android** :  
    Système d'exploitation pour appareils mobiles, basé sur le noyau Linux et développé par Google.
		*Plus d'informations dans le cours d'Interfaces Graphiques du S4*
    
- **macOS / iOS** - Systèmes d'exploitation propriétaires d'Apple, basés sur le noyau **XNU** (dérivé de Mach et de BSD), avec un environnement graphique fermé.
    - **macOS** : Pour ordinateurs de bureau et portables Apple.
    - **iOS** : Pour iPhone et iPad.

- **UNIX libre (BSD)** - Famille de systèmes d'exploitation dérivés de l'original UNIX, entièrement libres et utilisés principalement dans des contextes spécialisés ou serveur.
    - **FreeBSD** : Système d'exploitation BSD très répandu pour les serveurs.
	    *En général, c'est l'OS utilisé pour les consoles de jeu*
    - **NetBSD** : Système BSD connu pour sa portabilité, fonctionne sur de nombreuses architectures matérielles.
    - **OpenBSD** : BSD orienté sur la sécurité et la fiabilité.


### Famille Windows
- **Windows Desktop** - Systèmes d'exploitation de bureau et grand public développés par Microsoft.
    - **Windows 7, 10, 11** : Versions populaires et stables.
- **Windows Server** - Versions spécifiques de Windows optimisées pour des environnements serveurs, avec des fonctionnalités adaptées aux entreprises.
    - **Windows Server 2008, 2012, 2016, 2019, 2022** : Différentes versions de Windows Server, évoluant avec les besoins des infrastructures réseaux.


### Mainframe
- **1. IBM z/OS Family (Mainframe IBM)** - Les systèmes d'exploitation pour mainframes IBM (aussi appelés systèmes "Z") sont les plus courants et se répartissent en plusieurs familles, chacune avec des spécificités uniques.
	- **z/OS** : Le principal système d'exploitation pour les mainframes IBM modernes, successeur de MVS (Multiple Virtual Storage). Il est capable de gérer des millions de transactions par seconde avec des fonctionnalités avancées de sécurité, de gestion des ressources et de compatibilité avec des applications anciennes et modernes.
	- **z/VM** : Système d'exploitation de virtualisation (Virtual Machine) permettant de faire tourner plusieurs systèmes d'exploitation en parallèle sur un seul mainframe. Il est souvent utilisé pour créer des environnements virtuels sous Linux ou z/OS.
	- **z/VSE** (Virtual Storage Extended) : Un système d'exploitation plus léger, utilisé dans des environnements de mainframe moins exigeants que z/OS. Il est apprécié pour sa simplicité et est compatible avec des applications anciennes.
	- **Linux on IBM Z** : IBM supporte plusieurs distributions de Linux sur ses mainframes, comme **Red Hat Enterprise Linux**, **SUSE Linux Enterprise Server** et **Ubuntu Server**. Linux on IBM Z permet de bénéficier de la robustesse du matériel mainframe tout en utilisant un système d'exploitation ouvert.

- **2. Systèmes Bull / GCOS Family (Mainframe Bull)**
	- **GCOS (General Comprehensive Operating System)** : Utilisé principalement par Bull pour ses mainframes (qui étaient souvent déployés en Europe). GCOS est une famille de systèmes d'exploitation qui a évolué pour gérer les besoins de gestion de bases de données, de transaction, etc. Il existe plusieurs variantes, comme GCOS 7 et GCOS 8.
