[[Axe 4 - Manipulation de tables]]
05/01/2023
****
## Suppression de table

```sql
DROP TABLE IF EXISTS nom_table 
CASCADE;
```
permet de supprimer la table.
	*`IF EXISTS` est optionnel et permet, si besoin, d'éviter le renvoi d'une erreur si la table que l'on souhaite supprimer n'existe pas
	L'option `CASCADE` peut être spécifié à la fin si l'on souhaite supprimer les contraintes de clefs étrangères, de vues …, mais pas les entitées auxquelles ont fait référence*


```sql
TRUNCATE TABLE nom_table1, nom_table2 
RESTART IDENTITY;
```
permet de vider le contenu de la table, sans altérer sa structure.
	L'option `RESTART IDENTITY` permet de réinitialiser les Serial (Incrémentation). Si cette option n'est pas appelée, l'option `CONTINUE IDENTITY` est appliquée tacitement par le SGBDR
	L'option `CASCADE` peut être spécifiée. Elle est cependant assez dangereuse car **elle va vider le contenu d'autres tables** (celles qui font partie de la relation de dépendance de la clef étrangère).


****
## Insertion de valeurs

```sql
INSERT INTO x(col, col2) 
VALUES (y, z) 
RETURNING col2
```
permet l'insertion de valeurs dans la table x.
	L'option `RETURNING` permet de retourner une valeur après l'insertion, à savoir une valeur attribuée à la colonne qui vient d'être insérée
	Cela permet de récupérer la clef primaire auto-incrémentée par exemple, afin de s'assurer que l'insertion se passe bien, ou encore de récupérer la valeur d'un champ, dans le cas ou des contraintes DEFAULT s'applique*


****
## Mise à jour de valeurs

```sql
UPDATE nomTable 
SET col = val 
WHERE -- conditions
```

permet de modifier le contenu d'une table

Cette clause modifie **CHAQUE LIGNE qui va satisfaire la condition**, elle va donc modifier plusieurs lignes **si l'on ne précise pas une clef primaire dans la clause `WHERE`**.


****
## Suppression de valeurs

```sql
DELETE FROM x 
WHERE -- conditions 
```
permet de supprimer des enregistrements d'une table. 

Comme pour la requête de modification, cette clause modifie CHAQUE LIGNE qui va satisfaire la condition.