[[Axe 5 - Requêtes de sélection]]
11/01/2023
****

Une vue est une structure de table qui est créée par une requête. Cela signifie que **la vue ne stocke aucune donnée**. La structure de la table de retour de la vue sera inchangée, mais les valeurs renvoyées seront différentes en fonction du contenu de la BDD. 

On utilise généralement les vues pour des requêtes `SELECT` (éviter de le faire pour des `UPDATE`, `DELETE` …). 


Le premier objectif est d'**adapter le schéma relationnel** pour une application spécifique 
	*E.g: Une vue qui seléctionne les 5 films les plus récents sur letterboxd*

Le second objectif est de **restreindre l'accès à certaines données pour certains utilisateurs** 
Le but annexe est aussi d'utiliser la vue comme un **raccourci pour des requêtes complexes**, mais ça ne doit pas être l'objectif principal...


****
## Mise en place de Vue

Le comportement est relativement similaire à la création d'une table :
```sql
-- Les étudiants en L2 qui passent en L3
CREATE VIEW nouveauL3 AS (
	SELECT numEtud, nom
	FROM etudiant NATURAL JOIN examen NATURAL JOIN cours
	WHERE note >= 10 AND numLic = 2
	GROUP BY numEtud, nom
	HAVING sum(ects) >= 60
);
```


Utilisation de la vue :
```sql
-- Les futurs L3 qui ont suivi le cours de BDD
SELECT * FROM nouveauL3 NATURAL JOIN examen NATURAL JOIN cours
WHERE intitule = 'Base de données';
```


Suppression de la vue :
```sql
DROP VIEW nouveauL3;
```


****
## Vue matérialisée

Une vue matérialisée est l'**inverse d'une vue** : Le contenu y est ici stocké, afin de mémoriser le résultat de requêtes.
	*Cas d'utilisation: Charts 2022 sur un site de musique type RYM. Le calcul peut être très couteux, et on ne voudrait pas avoir à le faire à chaque fois qu'un utilisateur charge la page d'accueil. De ce fait, on conserve un temps les résultats sur une vue matérialisée, et on la met à jour à un rythme précis.*


Création de la vue matérialisée et choix des attributs :
```sql
CREATE MATERIALIZED VIEW nomVue (arg1, ..., argN)
AS (requête);
```

Mise à jour de la vue matérialisée :
```sql
REFRESH MATERIALIZED VIEW nomVue;
```

Suppression de la vue :
```sql
DROP MATERIALIZED VIEW nomVue;
```

