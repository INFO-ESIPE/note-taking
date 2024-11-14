[[Design Pattern]]
****
**Table of Contents**
```table-of-contents
```

****
## UML

Unified Modelling Language (UML) is a standard that allows representation of classes (designed for OOP).
	*Since it is a standard, it does not depend on any language. If a language implies some mechanics that aren't represented in UML, it is up to the developers to create their own. **What matters is that everyone understand***

This topic is vast (and boring), so we will only focus on **class diagrams** here...

Visibility:
- + for public
- - for private
- = for default
We indicate properties and methods. For each, we give it's name and type (return type in the case of methods):
![[class.png]]


Some Java features has no UML representation (e.g., indicating that an entity is an interface or an enumeration). We won't hesitate to **adopt our own contentions, the standard isn't rigid at all and we are free to do so as long as it remains fairly comprehensible**
![[convention.png]]


****
## Relations

The UML diagram is here to **represent** classes, but especially the **relations between those classes**.

A dotted arrow indicates that a class **uses another one**.
The class pointed by the arrow is the one being used:
![[relation.png]]

However, this is a very generic representation. There are more conventions to give a better representation of how classes are linked with each other...


**Inheritance (`extends`)**

![[extends.png]]


**Interface implementation (`implements`)**

![[implements.png]]


****
## Aggregation and Composition

They both indicates that a class **contains** one or several instances of another class.
The difference is the strength of the link between those two classes:
- **Aggregation:** Destruction of the containing class does NOT implies the destruction of the contained class
- **Composition:** Opposite, if we destroy the containing class, the contained one is destroyed too

**Example:**

![[dentist.png]]

The dentist has:
- An **aggregation relation with his teeth** (he can live without them)
- A **composition relation with his heart** (he will perish!!)

*Mind the `0..32` and  `0..1` that indicates the number of instances*


This is very useful to **represent mappings of classes in the database**:
![[student.png]]

Deleting a student in the database does not destroy the lectures he attended nor the university he registered in.
However, it should destroy the remarks we gave him.

*Note that, for french people (and perhaps some other nationalities as well), we have our unique model to represent elements in the database (Modèle Conceptuel de Données) that shares a few similarities, especially the cardinalities*
