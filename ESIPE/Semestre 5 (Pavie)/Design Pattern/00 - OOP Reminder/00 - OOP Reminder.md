[[Design Pattern]]
****
**Table of Contents**
```table-of-contents
```

****
## Encapsulation

Fields represents the state of an object
Public methods are the object contract, those are the only methods external code will interact with... hence why it needs to be protected (set correct visibility, pre-conditions, defensive copies...)

Java (unlike python, for instance) comes with several visibility levels:
- `private`: Only visible inside the class itself
- `public`: Everyone can access it
- `default`/`package`: Only visible by classes belonging to the same package
- `protected`: Only visible inside the class and any classes that inherit from it (children). Since inheritance is a bad practice, we never really use this...
*Visibility guides code users, even though it is possible to bypass it via the Reflexion API*


So, an object defines:
- What's inside = its state = his (private) fields
- What's outside = other objects
His public methods are his contract regarding those other objects (outside world). Those objects are his clients and his public methods are the services he provides to them.
	*Clients should only mind about the contract, not the implementation (private stuff that defines how things works). So, it doesn't really matter when we change the implementation of an object, but **it has a considerable impact on clients when we modify it's contract***


The constructor is the entry point for the object's creation
- It only makes initialisations (creation of **state**) and verifications (checks coherence)
- If we need to make complex calculations to make the object, we'd rather **delegate the work to a dedicated factory method OR another object** (cf. `Builder` and `Factory` design patterns)

Public methods must maintain a coherent state of the object (beware of getters and setters, do not make them like a retard at the creation of the class if it's not needed)
	*The object itself is the only one responsible of his state's coherence*


****
## Immutability

We call an object immutable if it's state can not change after it was initialised
	*There are Records since Java 19 to do this*

**If we can make an object immutable, it's better!** (No concurrency issues, no incoherent state, cool !!!)
	*Obviously, if it has to be done, then you do it... There is still a performance issue if you re-create an object each time you want to change one of its value instead of making it mutable*


****
## SOLID

A concept = a class
A class = a single role