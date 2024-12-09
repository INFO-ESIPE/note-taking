[[Java]]
****
**Table of Contents**
```table-of-contents
```

****
## CBSE
*Note: Everything described here is language agnostic. We will focus on how it is related to Java later on, but it could—conceptually—work with several languages (especially compiled ones implementing correct encapsulation mechanisms, unlike Python...)*

To crystallise the idea of softwares built from prefabricated components, Component-Based Software Engineering (CBSE) was introduced.
	*Purpose was to have faster development cycles (faster Time-to-market), smaller costs, easy to manage system construction...*

> In this context, software products where now seen as "Product families" sharing a same infrastructure and applying the same standards. This allowed programmers to focus on developing and testing new features only.

### Software Component

A **Software Component** is a **modular**, **reusable**, and **encapsulated** unit of software that **provides a specific functionality** or set of functionalities. 
It is designed to **interact with other components via well-defined interfaces** and can
be independently developed, deployed, and maintained.

A software component must respect a few key characteristics:
1. **Encapsulation:** As we know very well, the implementation is hidden and developers only interact with a backward-compatible API
2. **Reusability:** Component is generic enough to be re-used in various scenarios
3. **Modularity:** Component represents a specific feature that can be deployed independently (They are **loosely-coupled** and as they don't rely on other components to work)
4. **Composability:** Component can be combined with others (easy to integrate and "extend")
5. **Interchangeability:** Component can be replaced by another component which implements the same interface (flexible system evolution)

### Component Model

To emphasise with the aforementioned pre-requisites, we have a **component model** which sumarizes everything a component needs to usable in a CBSE context:
- **Component Interface**: The API, defines methods that the component implements or that other components can leverage
- **Composition Mechanism:** A protocol or procedure allowing two components to work together
- **Component Platform:** A platform for executing the components

Popular component models:
- **Jakarta Enterprise Beans** (formerly Enterprise JavaBeans), server-side component model used in JEE
	==*Not to be confused with "JavaBeans"*==
- **Spring Beans**, which encapsulates business logic in the Spring Framework. Supports dependency injection, lifecycle and dependencies are managed by the Inversion of Control (IoC) container.
- **JavaBeans**, reusable component model allowing developers to create objects (beans) that can be manipulated in a visual development environment (NetBeans or Eclipse)
- **.NET Component Model**, packages components as `.dll` files which can be shared across various type of applications.

Most frameworks incorporates a component-based logic:
- Spring (dependency injection and spring beans)
- Vue.js, React, Angular (UI elements broken into small reusable components)
- Apache Struts (MVC model for JEE)

A dedicated JEE class will be organised during fifth semester.


****
## Framework

A Framework is a **collection of common code providing generic functionality** that can be selectively overridden or specialised by user code. Frameworks provide a **standardised way to build and organise code**, often offering pre-built functionality for tasks like user interface design, database interaction, and input/output management.
	*Frameworks exists for most modern languages. Major ones for Java are Spring, Helidon and Quarkus (JavaFX and Swing for GUI).*

### Inversion of Control

Even though frameworks comes with reusable abstractions of code wrapped in a well-defined API (like libraries), they differ in the way that frameworks also comes with **Inversion of Control**.
	*The program flow is more in control of the framework itself than the caller, unlike libraries*

This happens because the application architecture is often fixed and determined by the Framework.
	*Execution flow spends more time in "Framework Code" than in "Application Code".
	It lets developers focus more on business logic while the framework manages lower-level operations like object creation and lifecycle management.*



