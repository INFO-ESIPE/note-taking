[[Axe 5 - Requêtes de sélection]]
11/01/2023
****
**Table of Contents**
```table-of-contents
```

****
## Objectif

Le but principal d'une sous requête est de **mettre en œuvre une requête trop complexe pour être faite simplement**
	En essence, **décomposer une requête complexe en de plus petites requêtes** de tailles intermédiaires. 

*Par exemple, pour savoir si un élève x a une moyenne supérieure à la moyenne de la classe, il est nécessaire de récupérer cette moyenne dans une sous requête*


Une sous-requête calcule une table, afin qu'elle puisse être utilisée comme une table dans la clause `FROM`, mais aussi dans la clause `WITH`, afin de définir une table auxiliaire
On peut, de plus, l'utiliser dans la clause `WHERE`, avec les opérateurs `IN`, `EXISTS` et `ANY`/`ALL`. Si la table **n'a qu'une seule valeur**, elle peut être utilisée comme une valeur simple. 

Une sous-requête peut utiliser des attributs de ses parents, ce qui fait d'elle une **requête corrélée**...


****
## `FROM` et `WITH`

Une sous-requête dans la clause `FROM` doit avoir un alias (`AS`), car elle sera utilisée comme toute autre table 
La sous-requête dans `FROM` peut être corrélée et récursive, ce qui n'est pas le cas pour `WITH`. 

Une sous-requête dans la clause `WITH` est similaire à celle pour `FROM`, cela change essentiellement la syntaxe. Cependant, celle-ci doit être **déclarée avant la requête**. Cela va créer une table temporaire qui sera supprimée après la requête


****
## `IN` et `NOT IN`

Permet de tester l'appartenance (ou non-appartenance), tout simplement...


****
## `ANY` et `ALL`

Permet de tester des conditions sur les valeurs renvoyées par la sous-requête :
- `ANY` est vrai si au moins une valeur respecte la comparaison 
- `ALL` est vrai si chaque valeur respecte la comparaison 

**La sous-requête ne doit renvoyer qu'une seule colonne**


****
## `EXISTS` et `NOT EXISTS`

`EXISTS` permet de vérifier que la sous-requête **renvoie au moins une valeur** (peu importe la valeur)
	*C'est utile pour vérifier si la sous-requête est un minimum correcte dans sa logique, ou de faire une comparaison entre deux valeurs d'une nouvelle façon*

`NON EXISTS` vérifie que rien n'est renvoyé.


****
## Sous-requête comme valeur

Si la sous-requête renvoie **une seule ligne et une seule colonne**, alors elle peut être **utilisée comme une valeur** (un int, un varchar…).


Cette fonctionnalité est pratique si l'on souhaite comparer une valeur dont on est sur qu'elle est unique et sur une colonne 
	*Calcul de moyenne avec avg(), maximum avec max() ou autres agrégats ...* 

Attention toutefois lorsque l'on décide d'utiliser des conditions `WHERE` dans la sous-requête. Si deux valeurs ont le même résultat, alors les deux seront renvoyés, ce qui va causer des erreurs dans le cas ou on utilise cette sous-requête comme une valeur...


Pour éviter ce problème (un cas ou deux lignes peuvent être possiblement renvoyées), on spécifie avant la comparaison `ANY` ou `ALL`.


**Exemple :**
```sql
-- étudiants qui ont eu une meilleur note que Dupuis dans le cours 25
SELECT numEtud
FROM examen AS ex1
WHERE codeCours = 25
AND note >
	(
		SELECT examen.note
		FROM examen NATURAL JOIN etudiant
		WHERE examen.codeCours = 25
		AND nom = 'Dupuis'
	);
```
Le problème majeur que peut causer cette sous-requête comme valeur (que l'on veut utiliser de part leur aspect pratique) est qu'une mauvaise implémentation **ne créée pas d'erreur en runtime**. Une erreur ne sera présente que dans le cas ou deux résultats sont, un jour, sélectionnés depuis la table ciblée (ce qui peut arriver aléatoirement à n'importe quel moment).


