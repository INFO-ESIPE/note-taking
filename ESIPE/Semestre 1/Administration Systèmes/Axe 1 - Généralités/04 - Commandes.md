[[Axe 1 - Généralités]]
30/09/2022
****
**Table of Contents**
```table-of-contents
```

****
## Bash

**Bash** est un interpréteur de commande présent nativement sur linux. Il interprète des lignes de commandes dans un langage machine. 
	*Il en existe d'autres plus modernes et configurables, comme zsh*

Il permet de créer des scripts shell (.sh), et vérifie les erreurs (validité des lignes).


Il possède un historique :
`/home/user/.bash_history` (`.zsh_hirstory` pour zsh)

Le fichier `/home/user/.bashrc` (`.zshrc` pour zsh) est exécuté dès que l'utilisateur lance un terminal (et que bash est son shell par défaut).
	*On peut voir les shell par défaut des utilisateurs en affichant le contenu du fichier `/etc/passwd` (une ligne par utilisateur/service, le shell est spécifié en dernier sur la ligne*

Ce script bashrc permet de spécifier des configurations à appliquer à chaque lancement — afin de faciliter la vie de celui qui l'utilise.
	*On peut définir des alias, modifier le PATH, etc...*
```
# aliases
alias ..='cd ..'
alias ls='ls --color'
alias ll='ls -l'
alias la='ll -A'
alias c='clear'

# env
export PATH="$HOME/.local/bin:$PATH"
```


****
## Shell

Donne une interface en ligne de commandes (disponible même sur un système CLI). 
De nombreuses variantes existe: 
- Bash, zsh, dash, PowerShell 

Bash est un Shell interactif par défaut pour Debian (Couleurs, auto-completion, User input)


*Ceci est différent des **émulateurs de terminal** comme Alacritty, Ghostty...*


****
## Commandes

Commandes internes
- Spécifiques à Bash 
- Pas de processus lancé 
- Commandes simples: `cd`, `echo`… 
- Commandes composées: `if`, `for`… (pour plus tard dans le cours)

Commandes externes: 
- Code dans un fichier (bin) 
- Création d'un processus 
- Commandes UNIX 
- Toutes les commandes situées dans le PATH 


Les commandes qu'on lance dans bash sont des binaires accessibles depuis notre position dans l'arborescence. Certains sont accessibles de partout, car ces derniers sont dans le **PATH** (variable d'environnement listant les répertoires exécutables du système).
	*On peut afficher les différents répertoires pris en charge actuellement par le PATH avec `echo $PATH`

Pour savoir ou est situé une commande qu'on exécute, on utilise les commandes suivantes (au choix) :
```bash
whereis <cmd> (indique les localisations)
type <cmd> (indique localisation ou si builtin)
```


**Astuce :** La commande `whatis <cmd>` permet d'obtenir les détails basiques sur une commande. Pour de plus amples détails, utiliser le manuel via `man`.


****
## Opérateurs

L'opérateur `|` (pipe) permet de connecter la sortie d'une commande avec l'entrée d'une autre.
**Exemple** : Nombre d'utilisateurs connectés à une machine 
`who | wc -l`


L'opérateur `&&` permet de chainer les commandes
**Exemple** : Créer un répertoire puis y créer un fichier
`mkdir nouveau_dossier && touch ./dossier/nouveau_fichier.txt`


L'opérateur `>` permet de renvoyer le résultat d'une commande vers un fichier 
**Exemple** : `whatis ls > fichier.txt `
	*Si aucun fichier n'existe, il sera automatiquement créé.
	Si un fichier du même nom existe déjà, il sera écrasé.*


L'opérateur `>>` permet l'ajout des lignes résultant d'une opération Bash. Contrairement à `>`, il ajoute ces lignes à la fin du fichier, au lieu de l'écraser. 
	*Cet opérateur est très utile pour alimenter des fichiers de log*


L'opérateur `<` permet une redirection de l'entrée standard, à savoir, rediriger le contenu d'un fichier vers une commande 

**Exemples** :  
`wc < monfichier.txt` 

`mysql ma_bdd < script.sql > sortie.txt` 
- L'utilisateur se connecte à la bdd "ma_bdd" 
- Il soumet les requêtes dans le fichier script.sql 
- Il récupère le résultat des opérations dans le fichier "sortie.txt"

