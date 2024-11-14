[[Axe 3 - Bash]]
05/10/2022
****
**Table of Contents**
```table-of-contents
```

****
## Objectifs de Bash

Bash (Born Again Shell) est le langage que l'on manipule depuis le début dans le terminal. Bien qu'on ne l'utilisait que pour des lignes de commandes basique, il permet également de réaliser des tâches plus complèxes.

Bash est un langage interprété (comme python ou php…). A l'inverse des langages compilés, un programme va interpréter le code ligne par ligne (avec une vérification de syntaxe).


Ce langage a différents objectifs:
- Automatisation de tâches répétitives 
- Automatiser des configurations 
- Analyse de journaux de log 
- Surveillance du système  
- ...


****
## Caractéristiques principales du langage

Variables (non typées) 
Opérateurs
Structure de contrôle (if, else…)
Fonctions (merdiques)
Opérateurs spécifiques (`;`, `|`, `&&` ...)

Bash manipule principalement l'équivalent de chaînes de caractères.

| Avantages                                                        | Inconvénients                                                                  |
| ---------------------------------------------------------------- | ------------------------------------------------------------------------------ |
| Langage interprété                                               | La syntaxe PUE LA MERDE genre c'est grave hein                                 |
| Peu d'erreurs de typage                                          | Syntaxe doit être précise (l'oubli d'un espace peut  <br>provoquer une erreur) |
| Construction rapide                                              | Caractères ayant une signification variable en fonction du contexte            |
| Permet de connecter des composants écrits dans d'autres langages | Plusieurs façons d'écrire la même chose                                        |


****
## Wildcards

| Caractère | Fonction                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
| --------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| \*        | **Joker (wildcard)**, représente n'importe quelle séquence de caractères (y compris une chaîne vide).  <br>  <br>Exemple : `find /etc -name *apache*` trouve tous les fichiers contenant "apache" dans leur nom.                                                                                                                                                                                                                                                                                                                                                                               |
| ?         | Représente un seul caractère quelconque.  <br>  <br>Exemple : `ls fichier?` liste les fichiers dont le nom est fichier, suivi d'un dernier caractère pouvant être n'importe lequel.                                                                                                                                                                                                                                                                                                                                                                                                            |
| \[]       | Représente un seul caractère parmi ceux spécifiés entre les crochets. Fonctionne comme une liste de valeurs possibles.  <br>  <br>Exemple : `ls fichier[123]` liste les fichiers dont le nom se termine par "1", "2", ou "3".  <br>  <br><br>**Classes de caractères** comme `[:upper:]`, `[:lower:]`, ou `[:digit:]` permettent de désigner des catégories de caractères spécifiques (majuscule, minuscule, chiffre).  <br>  <br>Exemple : `ls [[:digit:]]` liste les fichiers commençant par un chiffre.  <br>  <br>`[!to]` exclut les caractères spécifiés entre crochets (ici, "t" et "o") |


****
## Script

La première ligne d'un script doit commencer par le **shebang "#!"**, en précisant l'interpréteur de commande (`/bin/bash` pour nous): 
```bash
#!/bin/bash 

# ...
```


Le fichier de script n'a pas nécessairement besoin d'extension pour fonctionner, mais l'extension générique est `.sh`

En dehors de son utilisation pour le shebang, le caractère `#` est utilisé pour commencer un commentaire sur une ligne :
```bash
#!/bin/bash

# Enable autocompletion for read input
read_with_completion() {
    local input
    read -e -i "$1" -p "$2" -e input
    echo "$input" # Result is printed in terminal
}
```


****
## Variables

Les variables sont identifiées par un nom 
- Peut contenir lettres et chiffres 
- Pas d'espaces 
- Ne commence pas par un chiffre 

Les variables peuvent avoir trois types 
- **Variable utilisateur** - définies manuellement dans le script
	*username, a, feur…* 
- **Variable prédéfinies du shell** - définies automatiquement par bash, et contrôlent l'apparence et le comportement de ce dernier
	PS1, PATH 
- **Variables des commandes UNIX** - définies par l'OS 
	HOME, TERM 


On effectue l'affectation d'une valeur à une variable sans mettre d'espaces, comme ceci :
```bash
username=romain
```
Pour utiliser la variable — une fois définie — on la précède du symbole `$` — sauf si dans une condition arithmétique utilisant `((` — comme ceci :
```bash
echo $username
```


Les **double quotes** permettent l'interprétation des variables. Leur valeur sera donc substituée :
```bash
echo "Salut $var"
```
Affichera : `Salut romain` (si `var=romain`).


Les **single quotes** désactivent toute interprétation. Le texte est affiché tel quel, sans substitution :
```bash
echo 'Salut $var'
```
Affichera : `Salut $var` (exactement comme écrit, ce n'est pas vraiment ce qu'on veut ...).


****
## Variables spéciales

| Variable   | Action                                                                                                |
| ---------- | ----------------------------------------------------------------------------------------------------- |
| $?         | Donne la **valeur de retour (return code) de la dernière commande**, ou du dernier script/exécutable  |
| $$         | Donne le PID du processus courant                                                                     |
| $0         | Contient le **nom du script ou de la commande** en cours d'exécution.                                 |
| $#         | **Nombre d'arguments** passés au script                                                               |
| $1, 2, 3…n | Récupère l'argument n°\<val>                                                                          |
| $*         | Contient tous les arguments passés au script (sauf $0 donc, car il n'est pas passé par l'utilisateur) |


****
## Structures de contrôle

**If**
```bash
if rm toto; then
	echo "Fichier Toto supprimé"
elif [ -f toto ]; then
	echo "Le fichier existe encore"
else
	echo "Erreur : Le fichier n'existe pas ou ne peut pas être supprimé"
fi # signale la fin de la condition
```


**Boucle While**
```bash
while finger | grep -i boin > /dev/null; do 
	echo "L'utilisateur est connecté" 
	sleep 5 # en secondes
done
```


**Boucle For**
```bash
for medecin in $(cat liste_medecins.txt); do 
	echo "Envoi d'un email au Dr $medecin ..."
	sleep 10 # en secondes
	echo "Email envoyé au Dr $medecin" 
done
```


****
## Expressions conditionnelles

`[[` est une commande interne utilisée avec `if` pour évaluer des **expressions conditionnelles** sur les chaînes de caractères.

**Exemple :**
```bash
if [[ -x test ]]; then 
	echo "Le fichier est exécutable" 
else 
	echo "Pas exécutable" 
fi
```
**Note :** Les espaces autour des crochets sont obligatoires.


`((` est utilisé pour les **opérations arithmétiques**.

**Exemple :**
```bash
if (( valeur > 3 )); then 
	echo "$valeur est plus grand que 3" 
fi
```


`==`, `!=`, `>=`, `<=` permettent de comparer des chaînes de caractères ou des valeurs.


****
## Exécution du script

Si on est déja dans un shell bash, on se contente d'exécuter le script ainsi : 
`./script.sh`
Sinon, on le donne en argument au bin bash :
`bash ./script.sh`

Comme mentionné dans [[02 - Utilisateurs et permissions|le cours sur les utilisateurs et permissions]], il faut que la personne qui veuille lancer un script ait les droits d'exécution dessus.
	*La commande `chmod u+x script.sh` ajoute les droits d'exécutions au propriétaire du script
	La commande `chmod +x script.sh` ajoute les droits d'exécutions à tout le monde*
