[[Axe 2 - SQL]]
05/01/2023
****

Une opération ensembliste permet de **combiner deux requêtes SELECT **

| Opération   | Résultat                                        |
| ----------- | ----------------------------------------------- |
| `UNION`     | Lignes renvoyées soit par Q1, soit par Q2 ( ∪ ) |
| `INTERSECT` | Lignes renvoyées par Q1 et Q2 ( ∩ )             |
| `EXCEPT`    | Lignes renvoyées par Q1 mais pas Q2 ( \ )       |

Dès qu'une opération ensembliste est effectuée, les doublons sont systématiquement supprimés
	*comme ci un `DISTINCT` était spécifié de manière tacite*


Le mot clef `ALL`, spécifié après l'opération ensembliste, permet de conserver les doublons
	*Cette opération est moins efficace et plus coûteuse en performances*

| Opération       | Résultat                                        |
| --------------- | ----------------------------------------------- |
| `UNION ALL`     | Lignes renvoyées soit par Q1, soit par Q2 ( ∪ ) |
| `INTERSECT ALL` | Lignes renvoyées par Q1 et Q2 ( ∩ )             |
| `EXCEPT ALL`    | Lignes renvoyées par Q1 mais pas Q2 ( \ )       |
