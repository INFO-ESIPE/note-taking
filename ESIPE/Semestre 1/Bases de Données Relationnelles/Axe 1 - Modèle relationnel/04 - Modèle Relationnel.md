[[Axe 1 - Modèle relationnel]]
07/12/2022
****
**Table of Contents**
```table-of-contents
```

****
## Modèle Relationnel

Le Modèle Relationnel est un Modèle logique de données (MLD). 
Le Modèle Relationnel est basé sur la notion mathématique d'ensemble et de relation. 

Il permet de définir le schéma logique de bases de données. 

De plus, il permet de préciser les spécifications des opérations sur les données (calcul relationnel).


****
## Informations

Informations structurelles: 
- Nom de colonnes 
- Nombre de colonnes 
- Types de données prises en charge pour la colonne 
    
Données: 
- Le reste


****
## Domaines

Un domaine est un ensemble de valeurs qui sont d'un type identique
	*E.g: Les entiers, les réels, les mots de trois lettres, un ensemble (blanc, rouge, bleu…)*

Le produit cartésien des domaines *D1, D2 … Dn*, noté *D1 x D2 x … x DN* est l'ensemble de toutes les combinaisons de domaines.


****
## Relations

Une relation est un sous-ensemble du produit cartésien d'une liste de domaines 

Cette relation est représentée sous la forme d'un tableau à double entier: 
- Colonne: Les attributs, définies par un nom 
- Ligne: Les enregistrements


****
## Schéma d'une relation

Données concernant le nom d'une relation, de ses attributs (et domaines) et de la clef primaire (soulignée) 

E.g: `films(idF, titre, annee, idDir)`


****
## Schéma d'une base de données

Ensemble des schémas des relations de la BDD 
Données des clefs primaires et étrangères de chaque relation 

**Exemple :** 
```
films (idF, titre, annee, idDir) 

realisateurs (idDir, nom, prenom, dateNaiss) 

FK : film (idDir) fait référence à realisateur (idDir)
```
