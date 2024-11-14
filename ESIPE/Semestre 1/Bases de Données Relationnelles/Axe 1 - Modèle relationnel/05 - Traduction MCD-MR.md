[[Axe 1 - Modèle relationnel]]
07/12/2022
****
**Table of Contents**
```table-of-contents
```

****
## Caractéristiques

Chaque type d'entité devient une relation
Chaque propriété d'une entité devient un attribut de la relation
La propriété identifiante devient la clef primaire de la relation


****
## Traduction des relations

**1 vers 1**
Si les cardinalités maximales de l'association sont 1 (donc il peut y avoir un 0,1): 
- L'association est intégrée à l'une ou l'autre des deux entités
- L'identifiant de l'entité 2 est ajouté en clef étrangère à l'entité 1 
- Les attributs de l'association sont ajoutées à ceux de l'entité 1 


**1 vers n** 
Si les cardinalités maximales de l'association sont 1 vers n  (0,1 // 1,1 vers 0,n // 1,n): 
- L'association est intégrée à l'entité de cardinalité 1 
- L'identifiant de l'entité de cardinalité n est ajouté en clef étrangère à l'entité de cardinalité 1 
- Les attributs de l'association sont ajoutées à ceux de l'entité de cardinalité 1 


**n vers n**
Si les cardinalités maximales de l'association sont n vers n (0,n // 1,n vers 0,n // 1,n): 
- L'association devient une nouvelle relation 
- Sa clef primaire se compose des identifiants des deux entités (Clef primaire composée) 
- Les attributs de l'association sont ajoutés à cette nouvelle relation. 

Une nouvelle relation "nomRel (idRel1, idRel2, …)" sera à ajouter au MR. 
Les deux clefs étrangères seront également à préciser dans le MR. 


**Arités >= 3 (n)** 
Si l'association est d'arité >= 3 (sans aucune cardinalité avec un maximum de 1), on procède comme dans le cas n vers n: 
- L'association devient une nouvelle relation 
- Sa clef primaire se compose des identifiants des entités concernées

*Note: Une ternaire ou les 3 cardinalités n'ont pas une valeur maximale de n, la ternaire ne fait pas sens et est surement fausse*


**Associations récursives**
- On procède comme dans les cas précédents, suivant les cardinalités 
- Il est généralement nécessaire de renommer l'un des attributs 
- Ce renommage se reporte dans la description des clefs étrangères


****
## Exemple de traduction

```
Etudiant (numEtud, nom, prénom, idGroupe) 
Groupe (idGroupe) 
Passe_examen(numEtud, idCours, note) 
Enseigne (idGroupe, numProf, idCours, heures) 
Cours (idCours, intitulé, numProf) 
Professeur (numProf, nom, prénom) 
Requiert (idCours, idCoursRequis, note) 

FK : Etudiant (idGroupe) fait référence à Groupe (idGroupe) 

FK : Passe_examen (numEtud) fait référence à Etudiant (numEtud) 
FK : Passe_examen (idCours) fait référence à Cours (idCours) 

FK : Enseigne (idGroupe) fait référence à Groupe (idGroupe) 
FK : Enseigne (numProf) fait référence à Professeur (numProf) 
FK : Enseigne (idCours) fait référence à Cours (idCours) 

FK : Cours (numProf) fait référence à Professeur (numProf) 

FK : Requiert (idCoursRequis) fait référence à Cours (idCours)
```