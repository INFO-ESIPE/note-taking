[[Web And Multimedia Technologies]]
26/11/2024
****
**Table of Contents**
```table-of-contents
```

****
## Concept

Semantic Web is often called **Web of data**.
Most of the data we encounter is not actually part of the web. They are jealously held and kept by the applications themselves.
	*Web does not have a specific standard to describe data, its just a mass of data where the structure is chosen by each individual developer. Furthermore, those applications does not necessarily share the data.*

We needed a way to create relationship between those pieces of data.
**Semantic Web is here to describe the relation between portions of information coming from diverse sources**.

A basic architecture:
![[archi.png]]
> Rdf is the document that describes the relation between links and pages

A more complete architecture:
![[archi-complete.png]]
> Semantic web was built to be a **"Web of trust"**, which is not the case for the web we currently use.


****
## URI/IRI

A **Uniform Resource Locator (URL)** is a unique address that identifies and locates a specific resource on the web, such as a web page or a file.
	*We identify—and access—anything that exists the web*

A **Uniform Resource Identifier (URI)**, on the other hand, is a broader concept used to refer to any entity, not just those accessible on the web.
	*A URI is a way to identify anything and optionally represent it on the web.*
This URI can **take two forms:**
1. A **URL**: A reference that provides a way to access a resource on the web.
2. A **Uniform Resource Name (URN)\***: A persistent, location-independent identifier for a resource.

\*Example of a hypothetical **URN**:  
`urn:def://blue_laser`
> Here, `def://` could refer to an agency or directory of online glossaries and dictionaries, while `blue_laser` is the resource's persistent name.*


****
## RDF

Resource Description Framework (RDF) is a **data model** used to **describe and interlink resources** (such as documents, pages, or ideas) identified by **URIs**. 
It provides a structured way to represent information using **triples** (subject, predicate, and object).

### RDF Triples

RDF organises information into **triples**, each consisting of:
1. **Subject**: The resource being described (e.g., a document, page, or idea).
2. **Predicate**: The property or characteristic of the resource (e.g., author, theme).
3. **Object**: The value of the property (e.g., a person, topic, or resource).
```
↓subject↓  ↓predicate↓     ↓object↓
doc.html   has for author  Marco Porta
doc.html   has for theme   JavaScript
```
> This describes the relationships of the resource `doc.html`.

### RDF Syntax

RDF documents follows an XML syntax:
```xml
<rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
xmlns:inria="http://inria.fr/schema#">
	<rdf:Description rdf:about="http://inria.fr/rr/doc.html">
		<inria:author rdf:resource="http://ns.inria.fr/fabien.gandon#me"/>
		<inria:theme>Music</inria:theme>
	</rdf:Description>
</rdf:RDF>
```
- **`rdf:RDF`**: The root element defines the RDF document.
- **`xmlns:rdf`**: Declares the RDF namespace for standard RDF syntax.
- **`rdf:about`**: Identifies the resource (`doc.html`) being described.
- **`inria:author`**: A custom predicate (from the `inria` namespace) linking to the author's URI.
- **`inria:theme`**: Describes the resource's theme ("Music")


****
## RDFS

**RDF Schema (RDFS)** is a semantic extension of RDF that allows us to define **classes**, **properties**, and their relationships. It provides a basic vocabulary to structure RDF data, enabling the organisation of resources into a hierarchy.
	*It provides a simple way to define and organise classes and relationships in a hierarchy. Suitable for basic ontologies.*

It is composed of:
- **Classes**: Define categories of resources.
    *For example, `Person` and `Book` can be classes.*
- **Subclasses**: Create hierarchies by making one class a subclass of another.
    *For example, `Student` can be a subclass of `Person`*
- **Properties**: Define relationships between resources or between a resource and a value.
    *For example, `hasAuthor` can be a property linking a `Book` to a `Person`.*
- **Domain and Range**
    - **Domain** specifies which class a property applies to.
    - **Range** specifies the type of value a property can have.
	    *Example: For `hasAuthor`, the domain could be `Book` and the range could be `Person`.*
```xml
<rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
         xmlns:rdfs="http://www.w3.org/2000/01/rdf-schema#">
    <rdfs:Class rdf:about="http://example.org/Person">
        <rdfs:label>Person</rdfs:label>
        <rdfs:comment>A class for all people.</rdfs:comment>
    </rdfs:Class>

    <rdfs:Class rdf:about="http://example.org/Student">
        <rdfs:subClassOf rdf:resource="http://example.org/Person"/>
        <rdfs:label>Student</rdfs:label>
        <rdfs:comment>A subclass of Person for students.</rdfs:comment>
    </rdfs:Class>
</rdf:RDF>
```


****
## OWL

**Ontology Web Language (OWL)** builds on RDF and RDFS to provide a more advanced framework for defining and reasoning about **complex relationships and constraints** between data.
	*It extends RDFS with more expressive tools for defining relationships and reasoning, ideal for complex and detailed knowledge structures.*

```xml
<rdf:RDF xmlns:rdf="http://www.w3.org/1999/02/22-rdf-syntax-ns#"
xmlns:rdfs="http://www.w3.org/2000/01/rdf-schema#"
xmlns:owl="http://www.w3.org/2002/07/owl#"
xmlns:dc="http://purl.org/dc/elements/1.1/">
	<!-- OWL Header Example -->
	<owl:Ontology rdf:about="http://www.linkeddatatools.com/plants">
		<dc:title>An Example Ontology</dc:title>
		<dc:description>An example ontology to show its basic syntax</dc:description>
	</owl:Ontology>
	
	<!-- OWL Class Definition Example -->
	<owl:Class rdf:about="http://www.linkeddatatools.com/plants#planttype">
		<rdfs:label>The plant type</rdfs:label>
		<rdfs:comment>The class of plant types.</rdfs:comment>
	</owl:Class>
</rdf:RDF>
```


****
## Applications

Semantic Web is aimed at:
- Web of trust
- Service discovery, since—in theory—the semantic web describes everything with great precision
- IoT/Home automation

A global semantic web obviously does not exist, but some local implementations exists.