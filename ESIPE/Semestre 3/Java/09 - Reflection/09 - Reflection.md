[[Java]]
****
**Table of Contents**
```table-of-contents
```

****
## Concept

As explained in previous lessons, types are for the compiler. At runtime, we only manipulate values.
	*Either a primitive value, or a reference (address)*

A CPU only "understands" four kinds of values: `u32`, `u64`, `f32`, `f64`
	*Where `u` stands for unsigned, like in Rust. This is not exactly true, a CPU doesn't "understand" things, but those are the formats he can use.*
So, all the values are represented by one of those.
	*boolean/char/short/int → `u32`
	float → `f32`
	long → `u64`
	double → `f64`
	reference -> either a `u32` or a `u64` (compressed pointer)*

Like we have seen in assembly, a local variable is an **offset relative to the base address of a function call on the stack**.
An array element is located at the address: `array_address + index * element_size` 
	where `element_size` is either 8, 16, 32, or 64 bytes depending on the element type

A field is an offset relative to the address of its object in the **heap**.
A **static field** is an **offset** relative to the address of the class in the **metaspace**.


As the Garbage Collector needs to know if a memory cell (in the heap or the stack) is a reference or not. To achieve this (and more), the JVM adds metadata to things:
- For a method, we pre-compute the maximal size of the heap and remember the layout of local variables for each call.
- For an object, the associated class describes the type of each field

### Class

At runtime, a class is a structure describing itself, the interfaces, fields, methods...
So, an object knows its class. 
We can get the class of an object in two ways:
- `Class<?> object.getClass()`
- or `.class` (the "dot class" notation, it does the same thing as the `getClass()` method)
```java
// They works if the instance extends Object, which should be the case ...
Object objet = new Car();
Class c = object.getClass(); // Car.class

Object object = "hello";
Class c = object.getClass(); // String.class;
```
> [!info]
> The class at runtime is the one indicated next to `new`, as we know

Unlike `getClass()` (and `.class`, but the confusion is mostly with the method), `instanceof` asks if an object is an subtype of blabla:
```java
Object o = new Car();

o instanceof Car // true
o instanceof Vehicle // true if Car implements Vehicle

o.getClass() == Car.class // true
o.getClass() == Vehicle.class // false
```

### Array

Arrays in Java are objects, so they belong to a class.
`Class.isArray()` will return true on an array, and `Class.getComponentType()` will return the type of elements it stores.

**Array classes has no methods nor fields**

### Record

There is an `isRecord()` method we can call on a class.
We can also call the `getRecordComponents()` method on a record, which returns an array of `RecordComponent` objects.
	*Various methods can be called on each (`getName()`, `getType()`...)*

### Enum

Similar to what's above, an `isEnum()` method exists.
The `getEnumConstants()` method returns an array containing all the possible values of the enumeration.

### Inner/Local Class

Calling the `getClasses()` method on a class allows to retrieve an array containing the public inner classes.
The `isMemberClass()` allows to ensure a class is an inner class of another. 


****
## Reflection API

Since Java is a dynamic language, we know the class of objects at runtime
Methods provided in `java.lang.Class` allows to both find methods of a class, and call them with arguments.

### Methods

We can retrieve **public methods of a class' object** like so:
```java
// Returns an array of java.lang.reflect.Method containing all methods
// Does not include static methods
Method[] methods = String.class.getMethods();

// Find a public method by its name and parameters
// If the method is not found, throws a NoSuchMethodException
Method concat = String.class.getMethod("concat", String.class);
```

Now that we have a `Method` object (which represents an instance method of the class we queried), we can obtain more information about it:
- `getName()`: Returns the name of the method
- `getReturnType()`: Indicates what the method is supposed to return by forwarding it as a `Class` object
- `getParameterTypes()`: Returns an array of `Class` objects, one per parameter

### `AccessFlag`

`AccessFlag` is an Enum containing modifiers
	*`PRIVATE`, `ABSTRACT`, `VOLATILE`...*

We can retrieve the access flags of a `Class` object or a `Method` object via the `accessFlags()` method:
```java
var filesClass = Files.class;
var classFlags = filesClass.accessFlags(); // Returns a Set<AccessFlag>

var filesMethods = filesClass.getMethods();
var fMethodFlags = filesMethods[0].accessFlags(); // Get flags of first method

// Checks if the first method is private
fMethodFlags.contains(AccessFlags.PRIVATE);

// Checks several properties 
fMethodFlags.containsAll(Set.of(AccessFlags.PUBLIC, AccessFlags.STATIC));
```

### Invoke

We can call a method on an object with the `Object invoke(Object receiver, Object… args)` method:
```java
var method = String.class.getMethod("concat", String.class);
Object result = method.invoke("hello", "o"); // calling the concat method on
											 // a String object containing "o"
System.out.println(result);
```
> [!info]
> If we want to call a static method, we pass `null`

### Constructor

`class.getConstructors()` allows to retrieve all public constructors of a class.

If we need to retrieve a specific public constructor, we can use `class.getConstructor(...parameterClasses)` where we provide the types taken as parameter:
```java
// Constructor used for: new Point(int x, int y);
Constructor<Point> constructor = java.awt.Point.class.getConstructor(int.class, int.class);
```

Those methods return a (or several) `Constructor` object(s).
It has several methods attached to it (`accessFlags()`, `getParameterTypes()`), but the most interesting one is `newInstance(...args)`, which allows us to **call the constructor**:
```java
Constructor<Point> constructor = java.awt.Point.class.getConstructor(int.class, int.class);
Object result = constructor.newInstance(2, 3);
```

### Fields

We can retrieve public fields of an object with:
- `class.getFields()`
- `class.getField(name)`
```java
Field field = java.awt.Point.class.getField("x");
```

Those methods return a (or several) `Field` object(s).
It has several methods attached to it (`accessFlags()`, `getName()`, `getType()`...)


****
## Exceptions

When using the reflection API, we are outside of the typing system.
This means that the compiler does not control:
- If the method exists
- If the method isn't abstract
- If the method is called with the correct object
- If we are allowed to access it
- ...

In this case, the API will throw runtime exceptions (`ReflectionOperationException`).
- [[09 - Reflection#Methods|getMethod]] throws `NoSuchMethodException`
- [[09 - Reflection#Invoke|invoke]] throws 
	- an `IllegalAccessException` if the class containing the method cannot be accessed **from the current location**
	- an `IllegalArgumentException` if the receiver object is not of the correct class
	- an `InvocationTargetException` if an `Exception` was thrown in the called method. We can retrieve the real `Exception` behind this by calling the `getCause()` method. **The cause must be propagated:**
```java
try {
	...
} catch(InvocationTargetException e) {
	var cause = e.getCause();
	if (cause instanceof RuntimeException rte) {
		throw rte; // propagate runtime exceptions
	}
	if (cause instanceof Error error) {
		throw error; // propage errors
	}

	throw new UndeclaredThrowableException(cause);
}
```

- [[09 - Reflection#Constructor|getConstructor()]] throws a `NoSuchMethodException` if the specified constructor doesn't exist (similar to what happens with `getMethod`)
- [[09 - Reflection#Constructor|newInstance()]] throws
	- an `IllegalAccessException` if the constructor is not visible
	- an `IllegalArgumentException` if the receiver object is not of the correct class
	- an `InvocationTargetException` if an `Exception` was thrown in the constructor (similar to `invoke()`.

- [[09 - Reflection#Fields|getField()]] throws a `NoSuchFieldException` if the field does not exist


****
## Security

When calling `invoke()` and `newInstance()`, the default behaviour is to check the security context.
We can bypass the security by calling `setAccessible(true)`.

The method can, however, raise an `InaccessibleObjectException` if it is impossible to bypass the security. This typically happens when a closed module exists.
A module is a `module-info.java` file, more information in [[10 - Packaging#Encapsulation|the next class]].
	*Java classes (along with Spring/JEE Frameworks classes) are defined in closed module. If we do declare a module, it is closed by default.*


****
## Annotations

We can use pre-defined annotations, or define ours!
To use an annotation, we indicate the name of the class after a `@`, followed by parenthesis containing key-value pairs:
```java
// Using a custom "Author" annotation
@Author(name = "John Doe",
		company = "DoesSoftware")
public class Foo { /* ... */ }
```
> [!info]
> The values specified in the annotation **must be constants**.
> 
> *Accepted values: primitives, `String`, `Class`, `Enum` (of any kind), `Annotation`\*, and a parameterised array (`ElementType[]`)*
> 
> \*because Annotations does not support Inheritance, we use delegation instead (so, using another Annotation).

In fact, an annotation is *some kind of special interface* which contains:
- **parameter-less methods**, which returns an immutable type (since we feed them with constant values). Those methods can also return a default value.
- **meta-annotations**, indicating if the annotation is visible at runtime, and where we can use
```java
@Target(ElementType.METHOD)         // Can only be applied to methods
@Retention(RetentionPolicy.RUNTIME) // Available at runtime for reflection
public @interface Author {          // declare the "Author" annotation
	String name();
	String company();
	int year() default 0; // Optional attribute with default value
						  // If not filled, "0" will be used.
```
> [!info] 
> We use `@interface` to declare an annotation

### Meta-annotations

`@Target(ElementType…)` indicates **where the annotation can be used**
	*CONSTRUCTOR, METHOD, FIELD, LOCAL_VARIABLE, RECORD_COMPONENT...
	Several can be applied, mind the "..."*

`@Retention(RetentionPolicy)` indicates for **how long the annotation should be retained by the compiler**
	*SOURCE, CLASS, RUNTIME
	If not specified, CLASS retention is applied by default*

`@Inherited` indicates that an annotation should be inherited by a subtype if applied to its supertype

`@Documented` indicates if it should appear in the Javadoc

`@Deprecated` indicates that the annotation should no longer be used

`@Repeatable` indicates that a same target can apply it several time

### Kinds

There are three main types of annotations:
- **Marker Annotation**, with no methods
```java
public @interface Version0 { } // Declare

@Version0
public class Foo { }           // Usage
```

- **Single-value Annotation**, with one method which must be called `value`:
```java
public @interface Version1 {   // Declare
	String value()
}

@Version1("14.0")
public record Foo() { }        // Usage
```

- **Full Annotations**, with several methods
```java
public @interface Version2 {   // Declare
	int maj();
	int min();
}

@Version2({maj = 14, min = 7})  // Usage
public enum Foo { }
```

### Repetition

By default, an annotation is non-repeatable. But we can define an annotation container and mark it with the `@Repeatable(class)` meta-annotation:
```java
public @interface Hello() { }

@Repeatable(Hello.class)
public @interface HelloContainer {
	Hello[] container(); // Hence why we can use parameterised arrays :)
}
```
```java
@Hello @Hello
public record Foo() { }
```


****
## Annotations and Reflection

As we have mentioned [[09 - Reflection#Annotations|earlier]], an annotation can be accessed through reflection if it is declared with the `@Retention` meta-annotation set to `RentionPolicy.RUNTIME`.

The type of the annotation will be **an interface inheriting from `java.lang.annotation.Annotation`**.

An annotation is, in fact, an instance of a class—created by the JDK—which implements the interface that corresponds to the annotation.


The `AnnotatedElement` interface provides some methods to query annotations with reflection:
- `boolean isAnnotationPresent(Class<? extends Annotation> annotationType)` checks if the annotation is present
- `<A extends Annotation> A getAnnotation(Class<A> annotationType)` returns the value of the annotation, or null if not found
```java
@Retention(RetentionPolicy.RUNTIME)
public @interface Say {
	String message();
}

@Say(message="hello")
public class Foo { }

Say say = Foo.class.getAnnotation(Say.class);
System.out.println(say.message()); // call the interface's method
```
