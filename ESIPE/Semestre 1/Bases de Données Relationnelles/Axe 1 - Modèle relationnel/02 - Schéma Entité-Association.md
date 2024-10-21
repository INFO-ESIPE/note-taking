[[Axe 1 - Modèle relationnel]]
30/11/2022
****

Le schéma **EA (Entity-Relationship Diagram)** est un **modèle conceptuel de données (MCD)** qui va décrire le schéma d'organisation d'une BDD. 

Elle admet trois constituantes: 
- Entités (Tables)
- Attributs/Propriétés (Colonnes d'une table)
- Associations des entités (Clefs étrangères)


Chaque entité doit admettre un attribut qui permet d'**identifier chaque donnée de manière unique (Clef primaire)**.
	*Dans un schéma EA, il ne doit y avoir qu'un seul Identifiant à souligner (même si la clef primaire est composée)*


****
## Association et cardinalités

Une association est souvent binaire (fait le lien entre deux entités), mais elle peut dépasser cette valeur.
	*Une association ternaire (voire plus) est donc possible
	Une association récursive est également possible*


Les cardinalités indiquent **combien de fois l'entité est autorisée à participer à une association**. 
Elle admet un nombre minimal et maximal (`0,n` ; `1,n` ; `0,1`…).
	*Etant donné que les associations et leur cardinalité font le lien entre les entités, **on ne spécifie jamais de clefs étrangères dans le schéma EA***


****
## Construire le schéma Entité-Association

Déterminer les entités et pour chacune :
- Etablir la liste de ses attributs
- Déterminer un identifiant (clef primaire unique) 
    
Déterminer les associations entre entités et pour chacune :
- Etablir la liste des attributs propres à l'association
- Vérifier la dimension de l'association (binaire, ternaire, n-aire…)
- Définir les cardinalités de l'association 

Vérifier le schéma obtenu :
- Supprimer les éventuelles transitivités et redondances  
	*ne spécifier l'information qu'a un seul endroit afin de garantir le caractère synchrone de l'information*
- Comparer au cahier des charges 
	*Vérifier que tout est traité, et uniquement ce qui concerne directement la BDD, on ignore ce qui concerne l'apparence du site web, ou toute autre information superflue*

