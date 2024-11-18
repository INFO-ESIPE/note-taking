[[Web And Multimedia Technologies]]
11/11/2024
****
**Table of Contents**
```table-of-contents
```

****
## XML

EXtensible Markup Language (XML) is used to describe data.
	*Even though it was first designed to overcome some HTML limitations. That's why it appears with a similar `<tag>...</tag>` syntax.
	XML is a simplified version of SGML (which is too precise and complex for our needs)*

XML is actually a **meta-language** (a "skeleton" language used to design other languages). Real languages can implement XML rules.
	*So, saying a document is written in XML is not completely true, we use a language that implements XML rules*

Characteristics of XML:
1. Used to store data in a text file
2. Even though (languages implementing) XML looks like HTML, the purposes are totally different
3. XML isn't made to be read by humans
4. XML is a family of technologies
5. XML is verbose and old
6. XML is well-supported and platform-free

It looks like this:
```xml
<?XML version="1.0" encoding="ISO-8859-1"?> <!-- More on this line later -->
<message>
	<to>Luca</to>
	<from>Marco</from>
	<subject>Important information</subject>
	<body>
		Don’t forget our meeting tomorrow morning!
	</body>
</message>
```
*Note that the comments and overall syntax are similar to HTML*


****
## Main Features

As said above, XML allows to exchange data between different systems under a standardised format.
	*XML and JSON are the two main ways of describing data in a text file*

Since XML is a meta-language, it has no tags by itself
	*The languages implementing it defines the tags*

A tag can have any kind of name, but must respect some conditions:
- Doesn't start with a digit, punctuation, or "XML" (any case)
- Contains no space

An opening tag must always have a corresponding closing tag
	*Self-closing tags like `<blabla/>` are allowed (like in HTML)*

XML is case-sensitive (it is recommended to stick to lowercase, like with HTML)
```xml
<message> <!-- different --> <Message>
```

Spaces are not ignored (as our language needs to describe data, it has to keep its shape)
```xml
one two  three   four <!-- different --> one two three four
```

Attributes must be specified in between quotes:
```xml
<tag attribute="value"> <!-- correct -->
<tag attribute='value'> <!-- correct -->
<tag attribute=value> <!-- wrong -->
```


Another example:
```xml
<book>
	<title>Web Technologies</title>
	<classific id="inf-437" media="paper"/>
	<chapter>The HTML Language
		<paragraph>What is HTML?</paragraph>
		<paragraph>Structure of an HTML document</paragraph>
	</chapter>
	<chapter>Scripting languages
		<paragraph>Introduction to JavaScript</paragraph>
		<paragraph>Basic elements of the language</paragraph>
		<paragraph>Form validation</paragraph>
	</chapter>
</book>
```

*Note: XML tags can also be called "elements"*

### Elements and Attributes

When should I use elements or attributes ?
Let's say I want to describe the following information:
	`A male called Giuseppe Verdi` (good composer)

I can describe it in three ways:
```xml
<!-- 1 -->
<person>
	<gender>male</gender>
	<name>Giuseppe</name>
	<surname>Verdi</surname>
</person>

<!-- 2 -->
<person gender="male">
	<name>Giuseppe</name>
	<surname>Verdi</surname>
</person>

<!-- 3 -->
<person gender="male" name="Giuseppe" surname="Verdi"/>
```

It is better to use **elements for important properties** and **attributes for "accessory"
properties**.
Also, **if a property can assume several values, use an element**
	*As an attribute can only carry a single value...*

All the solutions above are alright in our case. However, if we want to add the notion of "job", they have to be defined by elements:
```xml
<person gender="male" name="Giuseppe" surname="Verdi">
	<job>Composer</job>
	<job>Musician</job>
	<job>...</job>
</person>
```


****
## Namespaces

It is possible to encounter a name conflict/collision in between tags:
```xml
<table> <!-- html table -->
	<tr>
		<td>Element 1</td>
		<td>Element 2</td>
	</tr>
</table>

<!-- blabla ... -->

<table> <!-- custom data table -->
	<name>Baroque table</name>
	<dim>100x200x120</dim>
</table>
```

Here our "table" tag is used twice, but describes different things.
	*Since XML is a meta-language, it is up to implementations to define the tags. But what if we have to deal with two data sources, where both uses the "table" tag but in a different way ?*

We can solve this behaviour with **namespaces**:
```xml
<h:table xmlns:h="https://www.w3.org/TR/html4/">
	<h:tr>
		<h:td>Element 1</h:td>
		<h:td>Element 2</h:td>
	</h:tr>
</h:table>

<f:table xmlns:f="https://www.tables.com">
	<f:name>
		<f:name>Baroque table</f:name>
		<f:dim>100x200x120</f:dim>
	</f:name>
</f:table>
```
We need to specify a **variable** before the conflicting tag name. They are both separated by `:`
> Here we named the variable "h", but it can be any sequence of character


****
## XML Validation

A **Well-formed** document follows syntactic rules of XML.
A **Valid** document follows **our Document Type Definition (DTD) or XML Schema**
	*Both of those documents serves the same purpose: check for XML validity
	The only difference is that an XML Schema is more sophisticated than a DTD*

So, defining a DTD/XML Schema allows us to detect anomalies in XML files we send/receives.

### Document Type Definition (DTD)

We can describe the DTD in the XML file itself, but it is considered a better practice to save it as an external file.

Here is our XML containing data:
```xml
<?xml version="1.0"?>
<!DOCTYPE message SYSTEM "mess.dtd"> <!-- Includes the DTD -->
<message>
	<to>Luca</to>
	<from>Marco</from>
	<subject>
		Un’informazione importante
	</subject>
	<body>Ricordati…</body>
</message>
```

And here is the DTD:
```dtd
<!-- Message tag must contain one and only one of each tags specified -->
<!ELEMENT message (to,from,subject,body)>
<!-- "#PCDATA" means it accepts a String (normal text) -->
<!ELEMENT to (#PCDATA)> 
<!ELEMENT from (#PCDATA)>
<!ELEMENT subject (#PCDATA)>
<!ELEMENT body (#PCDATA)>
```

**Element declaration:**
`<!ELEMENT element-name (element-content)>` if the element contains other elements
`<!ELEMENT element-name EMPTY>` if the element is empty
	*E.g., `<br/>`*
`<!ELEMENT element-name ANY>` if any content is allowed

**Data Types:**
- **\#PCDATA** (**P**arsed **C**haracter **Data**): A text that will be analysed by a parser
- **\#CDATA** (**C**haracter **Data**): A text that will not be analysed 

**Child elements occurrences:**
```xml
<!-- one occurrence of <title> -->
<!ELEMENT book (title)>

<!-- one or more occurrences of <title> -->
<!ELEMENT book (title+)>

<!-- zero or more occurrences of <title> -->
<!ELEMENT book (title*)>

 <!-- zero or one occurrence of <title> -->
<!ELEMENT book (title?)>

<!-- one occurrence of <title> and one of <chapter> or <paragraph> -->
<!ELEMENT book (title,(chapter|paragraph))>

<!-- generic (parsed) text and any number (also zero) of <title> and
<chapter> elements -->
<!ELEMENT book (#PCDATA|title|chapter)*> 
```
*As we can see, we can not be precise on the number of occurrences (e.g., allowing 5 children), we can only specify a \*. If we want to be specific, we have to use an XML Schema*

**Attribute declaration:**

Follows this syntax:
`<!ATTLIST element-name attribute-name attribute-type default-value>`

Example:
```dtd
<!-- accepts any type of data -->
<!ATTLIST classific id CDATA>

<!-- two possible values, and default one is paper -->
<!ATTLIST classific media (paper|CD) "paper">
```


### XML Schema

Better alternative to DTD. Not necessarily more complex, but more sophisticated and verbose (as it allows a more granular control).

Let's make both a DTD and an XML Schema for the following:
```xml
<?xml version="1.0"?>
<note>
	<to>Luca</to>
	<from>Marco</from>
	<heading>Reminder</heading>
	<body>
		Don't forget me this weekend!
	</body>
</note>
```

DTD:
```dtd
<!ELEMENT note (to, from, heading, body)>
<!ELEMENT to (#PCDATA)>
<!ELEMENT from (#PCDATA)>
<!ELEMENT heading (#PCDATA)>
<!ELEMENT body (#PCDATA)>
```

XML Schema:
```xml
<?xml version="1.0"?>
<xs:schema xmlns:xs="https://www.w3.org/2001/XMLSchema" targetNamespace="http://www.w3schools.com"
xmlns="http://www.w3schools.com"
elementFormDefault="qualified">
	<xs:element name="note">
		<xs:complexType>
			<xs:sequence>
				<xs:element name="to" type="xs:string"/>
				<xs:element name="from" type="xs:string"/>
				<xs:element name="heading" type="xs:string"/>
				<xs:element name="body" type="xs:string"/>
			</xs:sequence>
		</xs:complexType>
	</xs:element>
</xs:schema>
```


****
## XSL
