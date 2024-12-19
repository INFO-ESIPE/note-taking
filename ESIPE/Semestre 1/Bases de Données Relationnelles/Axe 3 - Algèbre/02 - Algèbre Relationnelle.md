[[Axe 3 - Algèbre]]
14/12/2022
****
**Table of Contents**
```table-of-contents
```

****
## Concept

Une formule d’algèbre relationnelle est une expression composée des relations du schéma relationnel connectées par des opérateurs définis: 

| Opérateur | Nom               | Type                       |
| --------- | ----------------- | -------------------------- |
| π         | Projection        | Unaire                     |
| σ         | Sélection         | Unaire                     |
| ρ         | Renommage         | Unaire                     |
| ×         | Produit Cartésien | Binaire                    |
| ∪         | Union             | Binaire                    |
| ∩         | Intersection      | Binaire                    |
| −         | Différence        | Binaire                    |
| ⋊         | Jointure          | Opérateur dérivé (binaire) |
| ÷         | Division          | Opérateur dérivé (binaire) |


****
## Définition et propriétés des opérateurs:

**La projection ( π )**
Choix des colonnes de sortie
La projection n'est pas commutative 

**La Sélection ( σ )**
Choix des lignes de sortie 
Les conditions sont des combinaisons booléennes (ET, OU, NON, <, >, <=….) d'attributs
La sélection est commutative


**Exercice :**
Dire si les expressions suivantes sont bien formées et pour celles qui le sont calculer leur résultat sur la relation ci-contre

![[ex1table.png]]
![[ex1formulas.png]]


1. 

| Nom   |
| ----- |
| Poire |
| kiwi  |
| pomme |
| figue |
| prune |

2. Non
3. 

| Id  | Nom   |
| --- | ----- |
| 1   | Poire |
| 2   | Kiwi  |
| 3   | Poire |
| ... | ...   |
| 9   | Prune |

4. Non
5. Même résultat que la 1
6. Non
7. 

| Id  | Nom   | Couleur | p   |
| --- | ----- | ------- | --- |
| 2   | Kiwi  | vert    | 100 |
| 4   | Pomme | vert    | 140 |
| 7   | Figue | vert    | 50  |
| 9   | Prune | vert    | 30  |

8. 

| Id  | Nom   | Couleur | p   |
| --- | ----- | ------- | --- |
| 2   | Kiwi  | vert    | 100 |
| 4   | Pomme | vert    | 140 |
| 5   | Pomme | marron  | 150 |
| 6   | Pomme | rouge   | 120 |

9. Mếme résultat que la 8
10. 

| Id  | Nom | Couleur | p   |
| --- | --- | ------- | --- |
|     |     |         |     |
*Le résultat est donc une table vide*

11. Non. `p` n'est pas un attribut de (πnom F)
12. 

| Nom   |
| ----- |
| Poire |
| Pomme |

13. 

| Nom   | p   |
| ----- | --- |
| Poire | 120 |
| Pomme | 140 |
| Pomme | 150 |
| Pomme | 120 |
****

**Le renommage ( ρ )**
Pas essentiel à connaitre syntaxiquement 

**Produit cartésien ( × )**
Combine l'ensemble des possibilités d'une relations de plusieurs tables
![[cart.png]]
*Cela revient à faire:*
```sql
SELECT * FROM fruit, préparation;
```

**Un produit cartésien est couteux**. En général on préfère effectuer une jointure (un **produit cartésien filtré**)

**Union ( ∪ ), Intersection ( ∩ ), Différence ( − )**
Dans les trois cas, R et S doivent être compatibles (avoir le même schéma). 
Union et Intersection sont commutatives (R ∪ S = S ∪ R) 
La différence n'est pas commutative, l'ordre a une importance. 

**Jointure ( ⋊ )**
La jointure est une opération dérivée (une opération qui a déjà une représentation possible, mais assez longue a exprimer alors on a mis en place une opération dédiée). 
La jointure est simplement un produit cartésien filtré (par une sélection). 
La jointure est la forme la plus courante de produit cartésien.


**Exercice :**
On considère le schéma relationnel suivant :
![[schema.png]]

Proposer une expression d’algèbre relationnelle pour obtenir les informations suivantes :  
1. Les noms des projets ayant un budget de plus de 225k euros 
	πnom (σbudjet > 225000 projet) 

2. Les noms des projets sur lesquels travaille l’employé numéro 1* 
	πnomP (σnomE = 1 (travaille) ⋊ projet) 

3. Les employés qui travaillent sur des projets à Paris et à Dublin 
	Il est nécessaire de consulter deux fois la table, car deux pays (deux projets ayant un pays différent) sont attendus. On retourne le numéro d'employé afin d'avoir un identifiant unique 
	πnumE (σville=Paris (projet) ⋊ travaille) ∩ πnumE (σville=Dublin (projet) ⋊ travaille).

****

**La division ( ÷ )**
Pas essentiel à connaitre syntaxiquement 
Permet d'obtenir un "pour tout".


**Exercice :**
On considère le schéma relationnel simple suivant :
![[scheme2.png]]

Ces expressions sont-elles bien formées ? Si oui, que calculent elles ?
![[expr.png]]

1. Non (Travaille a plus d'attributs qu'employé)
2. Oui. Elle retourne les employés qui travaillent dans un projet
3. Oui. Elle retourne les employés qui travaillent sur tous les projets
4. Non

