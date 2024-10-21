[[Axe 1 - Modèle relationnel]]
30/11/2022
****
## Base de Données

Une base de données représente une collection d'information structurée sur des entités et leurs relations dans un contexte applicatif particulier. 
La BDD a donc un rôle précis, qui rend service à une autre entité (application web/mobile, Réseau social…)

Des enjeux principaux accompagnent la mise en place d'une BDD, elle doit satisfaire l'ensemble de ces conditions: 
- Volume 
- Performance 
- Fiabilité (Synchronisation des données, modification globale d'une simple information…) 
- Sécurité 
- Partage (Données distribuées, Gestion d'accès concurrents) 
- Indépendance (Implémentation physique séparée du schéma logique)

Une BDD admet: 
- **Une indépendance physique des données :** La manière dont les données sont réellement stockées sur le disque (HDD, Utilisation de la RAM…) 

- **Une indépendance logique des données :** Possibilité de maintenir le fonctionnement de la BDD tout en en modifiant la structure ou l'infrastructure.


****
## Système de Gestion de Base de Données (SGBD)

Un SGBD est un ensemble de logiciels permettant la création, l'utilisation et la maintenance de Bases de Données.
	*Dans ce cours, nous utiliserons PostgreSQL*


****
## Utilisation

Utilisateur: 
- Se contente d'utiliser la bases de données
- On n'attend pas de lui qu'il sache comment cela fonctionne réellement, ou qu'il connaisse le schéma de la BDD

Développeur:
- Personne qui traduit les besoins en un schéma conceptuel de données
- Il conçoit et implémente les applications qui utilisent cette BDD

Administrateur: 
- Celui qui gère le schéma physique (interaction avec disque dur)
- Responsable du chargement, fiabilité et sécurité des données
- Optimisation et maintient des performances


****
## Langages

**Langages de conception :** 
Entité-Association 
- UML

**Langages de requête :** 
Langages déclaratifs; Spécifier le résultat attendu, mais pas la manière de l'obtenir
	*Comme les Streams en Java*
- SQL 
- XQuery 
- Cypher 

**Langages de programmation :** 
Langages impératifs; Préciser pas à pas la manière de procéder pour obtenir un résultat 
- PL/SQL 
- Java 
- PHP


****
## Etapes de développement de la BDD

1. **Cahier des charges** - Résumer les fonctionnalités attentes, besoins des utilisateurs… 
2. **Modélisation** - Modéliser les points mentionnés dans le cahier des charges 
	*Schéma relationnel // Entité-Association*
3. **Implémentation** - Optimisation, Création et insertion des tables/données…

