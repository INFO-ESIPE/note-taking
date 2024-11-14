[[Axe 5 - Requêtes de sélection]]
05/01/2023
****
**Table of Contents**
```table-of-contents
```

****
## Objectifs

Le but d'un agrégat est de regrouper les informations de plusieurs lignes d'une même table
La requête `SELECT` extrait des données lignes par lignes, ce qui ne permet pas de répondre à des questions du type : 
- Moyenne d'un étudiant
- Moyenne de la classe sur un contrôle
- ...

L'agrégat permet donc d'agréger les valeurs de plusieurs lignes, et donc d'**obtenir une donnée calculée plutôt que stockée** 

Les clauses d'agrégats sont à préciser juste après le `SELECT`
```sql
SELECT sum(prix) AS x FROM y -- ... 
```
*Il est recommendé de renommer la colonne d'agrégats avec `AS`*


| Agrégat             | Action                                         |
| ------------------- | ---------------------------------------------- |
| sum                 | Somme des lignes                               |
| avg                 | Moyenne des lignes                             |
| max                 | Valeur maximale entre toutes les lignes        |
| min                 | Valeur minimale entre toutes les lignes        |
| count(x)            | Compte le nombre de lignes retournées          |
| count(col)          | Le nombre de valeurs de la colonne sauf `NULL` |
| count(DISTINCT col) | Le nombre de valeurs uniques de la colonne     |


****
## Groupement `GROUP BY`

La clause `GROUP BY` permet de choisir les attributs sur lesquels regrouper les lignes 
Les agrégats sont calculés séparément sur les sous-ensembles de lignes dont les valeurs de groupement sont égales.
	*Cela a pour conséquence de produire plusieurs lignes, une pour chaque valeur de groupemenet*


****
## Post-groupement `HAVING`

La clause `HAVING` permet d'appliquer des **conditions sur les lignes après le groupement**, et donc, après le calcul des agrégats
	*C'est donc l'inverse d'une clause `WHERE`, qui elle ne sélectionne QUE les éléments qui respectent certaines règles, la ou `HAVING` va épurer les résultats après coup, en supprimant ceux qui ne respectent pas certaines clauses (équivalent du `filter()` sur un stream java)*

Cela peut influer sur des calculs de moyennes, par exemple, ou l'on va complètement ignorer des moyennes avec un `WHERE`, ce qui fausse le calcul. Cependant, avec un `HAVING`, ils vont être pris en compte pour le calcul, mais ignorés dans la réponse.
