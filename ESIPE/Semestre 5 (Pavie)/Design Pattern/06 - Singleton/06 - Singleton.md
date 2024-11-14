[[Design Pattern]]
****
**Table of Contents**
```table-of-contents
```

****
## Singleton

This pattern allows to create a **class you can only instantiate once**
	*In practice, this is used to represent a centralised resource (Database connection, object representing the player in a video game...*

The implementation in Java is quite easy:
```java
public class DB {
    private static final DB INSTANCE = new DB();
    // fields

    private DB(){
        // initialise fields
    }


    public static DB getInstance(){
        return INSTANCE;
    }
}
```

The basic logic is straightforward: 
- We make the constructor private so we can not instantiate from this class
- A private static and final field operates the one and only instantiation from the inside of the class
- The `getInstance()` is the only way to retrieve our object from outside.

**Note:** This implementation is thread-safe (as long as the class stays immutable). Java's memory model gives visibility of static variables to all threads once those variables are fully constructed.


****
## Eager vs Lazy

**Lazy** means the instance of the singleton is only generated at first use. 
	*This is what Java is doing. In Java, classes are loaded during their first usage*
**Eager** means the instance of the singleton is generated when the program starts.
	*This is what C++ is doing, it is loaded at the very beginning of the program, way before we even access it*

Eager has the advantage that the instance is already ready when needed, but might waste memory if we never access the instance.
Lazy is more performant as it only allocates the memory for the instance when we access it first. If we never access it, nothing happens.


****
## Singleton and SOLID

**Singleton** is one of those patterns who **completely breaks the SOLID principles**:
- **Dependency Inversion**: Singletons are accessed globally, it creates tight coupling
- **Open/Closed**: Singleton is by default closed to extensions as it requires a private constructor. If the constructor is opened, then it can not work as a singleton anymore


**A Singleton is NEARLY NEVER THE GOOD SOLUTION**
	*We mostly talk about it here as it is widely used in frameworks (especially J2EE) along with dependence injection*
