[[Axe 2 - SQL]]
05/01/2023
****
**Table of Contents**
```table-of-contents
```

****
## Tri

Par défaut, aucun ordre de tri précis n'est définit lors d'une sélection. 
	*Cela dépend de l'algorithme de tri utilisé par la requête, qui peut changer en fonction du contenu de la requête afin d'optimiser les performances*

La clause `ORDER BY` permet de spécifier un ordre de tri précis selon une ou plusieurs colonnes
	*Tri par défaut en ordre croissant (`ASC`), sinon décroissant (`DESC`)*

```sql
SELECT * 
FROM film 
ORDER BY titre DESC;
```


****
## Résultat

| Opération | Résultat                                   |
| --------- | ------------------------------------------ |
| LIMIT x   | Limite le nombre de résultats renvoyés à x |
| OFFSET x  | Ignore les x premières lignes              |

Cela permet — entre autres — de la pagination de résultats sur un site web (faire des tableaux dynamiques et à résultats limités) : 
	page 1: De 0 à LIMIT (x) 
	page 2: De OFFSET (x) à LIMIT (x*2) 

**Attention** : Un `ORDER BY` est obligatoirement à spécifier dans ce cas, pour s'assurer que l'ordre de résultats soit toujours le même