[[Axe 4 - Manipulation de tables]]
05/01/2023
****

Une **contrainte** peut porter sur une colonne précise, ou sur l'ensemble de la table.
On spécifie les contraintes affectant la totalité de la table, de manière générale, après avoir déclaré chaque colonne
	*Sinon on ne peut pas utiliser certains champs comme arguments de la contrainte, car ceux-ci ne sont pas encore déclarés*


****
## Types de données

| Type                                | Catégorie   | Rôle                                                                                      |
| ----------------------------------- | ----------- | ----------------------------------------------------------------------------------------- |
| integer, int, int4                  | Entiers     | Entier signé (4 octets)                                                                   |
| bigint, int8                        | Entiers     | Entier signé (8 octets)                                                                   |
| serial                              | Entiers     | Entier auto-incrémenté                                                                    |
| char(n), character(n)               | Str         | Chaîne de longueur fixe                                                                   |
| varchar(n) character varying(n)     | Str         | Chaîne de longueur au plus n                                                              |
| text                                | Str         | Chaîne de longueur arbitraire                                                             |
| real                                | Non-Entiers | Flottant simple précision                                                                 |
| float, double precision             | Non-Entiers | Flottant double précisio                                                                  |
| numeric, decimal                    | Non-Entiers | Décimaux en précision arbitraire                                                          |
| numeric(p,s)                        | Non-Entiers | Décimaux sur p chiffres, dont s après la virgule                                          |
| boolean                             | Autre       | Donnée booléenne                                                                          |
| Date / Timestamp \[with time zone\] | Autre       |                                                                                           |
| time \[with time zone\] / interval  | Autre       |                                                                                           |
| bytes                               | Autre       | Données binaires brutes (Rare, utilisé pour stocker des fichiers, mais on préfère éviter) |


****
## Contraintes

| Contrainte                   | Sens                                                                                                                   |
| ---------------------------- | ---------------------------------------------------------------------------------------------------------------------- |
| `PRIMARY KEY`                | Définit la clef primaire                                                                                               |
| `FOREIGN KEY x REFERENCES y` | Définit une clef étrangère                                                                                             |
| `UNIQUE`                     | La valeur d'une colonne (ou la combinaison de plusieurs colonnes) doit être unique sur cette même colonne              |
| `NOT NULL`                   | La valeur ne peut pas être nulle                                                                                       |
| `DEFAULT x`                  | Définit une valeur par défaut si aucune valeur n'est spécifiée                                                         |
| `CHECK`                      | Vérifie une condition qui doit être satisfaite lors de l'insertion de données CONSTRAINT nom CHECK (poids <= poidsMax) |

Spécifier 
```sql
CONSTRAINT nom [PRIMARY KEY]
```
permet de nommer la contrainte.
	*Cette opération n'est pas obligatoire, et permet uniquement de spécifier quelle contrainte à été enfreinte dans un message d'erreur
	Si la contrainte n'est pas nommée, le système en choisit un*


***
## Actions sur clef étrangère

La modification ou la suppression d'une table/colonne peut poser problèmes si une clef étrangère y fait référence.  
Pour gérer le comportement du SGBD vis-à-vis de cette situation, des actions peuvent être précisées afin de lui indiquer le comportement à suivre : 

| Action                   | Comportement                                                                                                                                                                                |
| ------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `NO ACTION` / `RESTRICT` | Annule la modification et renvoie une erreur                                                                                                                                                |
| `CASCADE`                | Propage la modification                                                                                                                                                                     |
| `SET NULL`               | Les valeurs des colonnes concernées sont remises à `NULL`                                                                                                                                   |
| `SET DEFAULT`            | Les valeurs des colonnes concernées sont remises à leur valeur par défaut.<br>*Dangereux si la valeur par défaut fait référence à une clef primaire qui n'existe plus dans une autre table* |

On précise sur quel type de comportement on veut appliquer l'action. 

Pour ce faire, on le spécifie avant le type d'action
```sql
ON UPDATE CASCADE -- ...
ON DELETE RESTRICT -- ... 
ON DELETE SET NULL
```

