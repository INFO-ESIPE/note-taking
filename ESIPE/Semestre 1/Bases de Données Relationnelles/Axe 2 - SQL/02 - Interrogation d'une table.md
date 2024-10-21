[[Axe 2 - SQL]]
07/12/2022
****

Une requête `SELECT` permet d'interroger la BDD. Cette requête va créer une table temporaire (la réponse) à partir d'une ou plusieurs autres tables au sein de la BDD questionnée.

On spécifie les lignes (sélections) et colonnes (projections) à extraire.

**Exemples :**
```sql
-- Sélectionne le nom et prénom de chaque réalisateur
SELECT nom, prenom FROM realisateur; 

-- Sélectionne toutes les propriétés de chaque film
SELECT * FROM film; 

-- Sélectionne toutes les propriétés de tous les réalisateurs ayant pour
-- prénom "David" et nom "Fincher" (le réal de merde hein)
SELECT * FROM realisateur WHERE prenom = 'David' AND nom = 'Fincher'; 

-- Sélectionne le nom et prénom de chaque réalisateur né après 1979
SELECT nom, prenom FROM realisateur WHERE anneeNaiss >= 1980;
```

Clause `SELECT`: 
- Choix des colonnes à retourner 
- L'opérateur wildcard "\*" permet de sélectionner toutes les colonnes 

Clause `FROM`: 
- Choix de la tables à manipuler

Clause `WHERE`: 
- Spécifier des conditions


****
## Conditions

| Condition         | Représentation                                                                  |
| ----------------- | ------------------------------------------------------------------------------- |
| Egalité           | ==                                                                              |
| Différence        | <>  <br><br>(!= marche sur certains SGBDR mais non normalisé)                   |
| Arithmétiques     | <, <=, >=, >                                                                    |
| Encadrement       | BETWEEN x AND y                                                                 |
| Filtre de chaînes | LIKE "blabla" <br>Un caractère manquant: _ <br>Plusieurs caractères manquant: % |
| Appartenance      | IN (x, y); NOT IN (x, y)                                                        |
| Booléennes        | AND; OR; NOT                                                                    |


****
## `NULL`

`NULL` représente une donnée manquante ou inconnue.
	*Sous PostgreSQL, les NULL sont représentés par des cases vides*

`NULL` ne vaut pas 0, ni une chaine vide, ni un espace blanc

Les comparaisons et opérations avec `NULL` renvoient systématiquement `NULL` : 
- `NULL + 1` vaut `NULL` 
- `NULL LIKE "John%"` vaut `NULL`


`IS NULL` et `IS NOT NULL` permet de tester si une valeur est nulle
**Attention, x = NULL et x <> NULL ne fonctionne PAS**, car cela teste l'égalité de valeur (qui renvoie systématiquement NULL)


****
