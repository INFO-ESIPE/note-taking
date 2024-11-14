[[Axe 2 - SQL]]
14/12/2022
****
**Table of Contents**
```table-of-contents
```

****
## Problème

Il est possible de spécifier plusieurs tables dans la clause `FROM` 
La requête est effectuée sur le produit cartésien des tables en entrée, ce qui donne généralement un résultat non souhaité...

Afin de régler ce problème, il faut spécifier dans la clause `WHERE` que la clef primaire de la table T' doit être identique à la clef étrangère de la table T. 

On appelle cette opération une **Jointure**.


****
## Jointure basique

Deux syntaxes existent: 
```sql
-- 1
SELECT colonne 
FROM tab1, tab2 
WHERE tab1.idTab2 = tab2.id AND -- … 

-- 2
SELECT colonne 
FROM tab1 
JOIN tab2 ON tab1.idTab2 = tab2.id 
WHERE -- …
```


****
## Jointure Naturelle (`NATURAL JOIN`)

Une **jointure naturelle** est une jointure qui se base sur l’égalité des colonnes portant le **même nom dans les deux tables** impliquées. Elle fusionne automatiquement les colonnes qui sont communes aux deux tables et ne conserve qu’une seule occurrence de chaque colonne partagée.


```sql
-- Table `film` : 
+---------+-------------+----------------+ 
| film_id | titre       | realisateur_id | 
+---------+-------------+----------------+ 
| 1       | Hana-Bi     | 11             | 
| 2       | Hard Boiled | 12             |
| 3       | Sans Soleil | 10             |
| 4       | Sonatine    | 11             |
+---------+-------------+----------------+ 

-- Table `realisateur` : 
+----------------+----------------+------------------+ 
| realisateur_id | nom            | nationalite      | 
+----------------+----------------+------------------+ 
| 10             | Chris Marker   | FR               | 
| 11             | Takeshi Kitano | JP               | 
| 12             | John Woo       | HK               |
+----------------+----------------+------------------+


-- Requête avec jointure naturelle (marche car les deux colonnes ont le même nom)
SELECT * FROM film NATURAL JOIN realisateur;
```


****
## Auto-Jointure (Self-join)

Une **auto-jointure** consiste à réaliser une jointure d'une table avec elle-même. Elle est utile pour **comparer des données** au sein de la **même table**. L'auto-jointure permet de traiter des relations entre lignes différentes d'une même table.

Comme il s'agit de la même table, il est nécessaire d'utiliser des **alias** (avec la clause `AS`) pour distinguer les différentes occurrences de la table dans la requête.

### Exemple

Supposons que nous avons une table **`film`** contenant des informations sur plusieurs films et leurs réalisateurs, mais nous voulons comparer les films réalisés par **le même réalisateur** pour voir par exemple, quel film a été tourné après un autre :

```sql
SELECT f1.titre AS film1, f2.titre AS film2, f1.realisateur_id 
FROM film AS f1 
JOIN film AS f2 
	ON f1.realisateur_id = f2.realisateur_id 
	AND f1.film_id < f2.film_id;
```


****
## Jointures Externes

La clause `JOIN x ON <…>` ne contient pas les lignes de la première table qui n'ont pas de correspondance dans la seconde table (et inversement)

A l'inverse, la jointure externe conserve ces valeurs, et les complète donc avec des `NULL`

| Jointure   | Action                                                                                |
| ---------- | ------------------------------------------------------------------------------------- |
| LEFT JOIN  | Conserve les lignes de tab1 (celle déclarée à gauche, donc en premier)                |
| RIGHT JOIN | Conserve les lignes de tab2 (celle déclarée à droite de la condition, donc en second) |
| FULL JOIN  | Conserve les lignes des deux tables                                                   |
*Le mot clef `OUTER` est spécifiable, mais optionnel (`LEFT OUTER JOIN`, `RIGHT OUTER JOIN`, `FULL OUTER JOIN`)*


**`JOIN`**
![[join.png]]


**`LEFT JOIN`**
![[leftjoin.png]]


**`RIGHT JOIN`**
![[rightjoin.png]]


**`FULL JOIN`**
![[fulljoin.png]]
