[[Java]]
****
**Note:** Even if generics are related to lambdas, most signatures will be incorrect due to missing wildcards (those will be mentioned in a dedicated class)
****
**Table of Contents**
```table-of-contents
```

****
## Refresher

In OOP, **universality** is the ability of an object to **represent both data and code**. 
Viewing a function as an object gives room for abstraction by separating:
- The generic and reusable part of an algorithm
- And its specific, non-reusable part

A good example of this is: `List.sort(comparator)`.
- The sorting algorithm itself (re-usable)
- The **comparator** (passed as an argument), which specifies how it should be ordered

All languages have a way to abstract code. In Java, we use **Functional Interfaces**.


****
## Functional Interface

A functional interface is simply **an interface with a single abstract method**.
	*Its here to represent a type of function, when you think about it... Take a look at the functional interfaces provided by Java and see for yourself*

A Functional interface can have as much default, static or private methods as necessary, the only requirement to respect is the one about the unique abstract method...

Example:
```java
@FunctionalInterface
public interface BiFunction<T, U, V> {
	public V apply(T t, U u); // unique abstract function
}

// Represents the type (T, U) -> V
// So it takes two generics as parameter, and return a generic
BiFunction<String, Integer, Double> fun = (String s, Integer i) -> s.length() + i + 5.0;
```

So, each time we want a type of function, we have to define the according functional interface
	*Kind of annoying, but fortunately the JDK provides us some (for the most common function types).
	**Problem**: You have to learn them (by heart... especially the name of their abstract method)*

Those native functional interfaces are in the `java.util.function` package.

### `Runnable`

Equivalent to `() -> void`. 

```java
Runnable code = () -> { System.out.println("hello"); }
code.run();
```
> [!info]
> We take nothing, we return nothing.

### `Supplier`

`Suplier<T>` is equivalent to `() -> T`

```java
Supplier<String> factory = () -> "hello";
System.out.println(factory.get());
```
> [!info]
> We take nothing, and return an object of any kind

There are equivalent for primitives (`[Int|Long|Double]Supplier`):
```java
IntSupplier factory = () -> 42;
System.out.println(factory.getAsInt());
```

### `Consumer`

`Consumer<T>` is equivalent to `(T) -> void`

```java
Consumer<String> printer = s -> System.out.println(s);
printer.accept("hello");
```
> [!info]
> We take a generic, and return nothing... we devour it.........

Same as `Supplier`, equivalent for primitives exists.

### `Predicate`

`Predicate<T>` is equivalent to `(T) -> boolean`

```java
Predicate<String> isSmall = s -> s.length() < 5;
System.out.println(isSmall.test("hello"));

// For primitives
LongPredicate isPositive = v -> v >= 0;
System.out.println(isPositive.test(42L));
```
> [!example]
> This is very useful for filters

### `BiPredicate`

`BiPredicate<T, U>` is equivalent to `(T, U) -> boolean`
	*Same as `Predicate`, but takes two parameters*

### `Function`

`Function<T, U>` is equivalent to `(T) -> U`

```java
Function<String, String> fun = s -> "hello " + s;
System.out.println(fun.apply("function"));
```
> [!info]
> Take a type, return a type. Both objects can be of the same type, but it is better to use an [[02 - Lambdas#`UnaryOperator`|UnaryOperator]] in that situation.

There are two sets for primitives (mind the difference):
- `[Int|Long|Double]Function<T>`, equivalent to `(int|long|double) -> T`
```java
IntFunction<String[]> arrayCreator = size -> new String[size];
System.out.println(arrayCreator.apply(5).length)
```

- `To[Int|Long|Double]Function<T>`, equivalent to `(T) -> int|long|double`
	*It's the opposite of the other one*
```java
ToIntFunction<String> stringLength = s -> s.length();
System.out.println(stringLength.applyAsInt("hello"));
```

### `BiFunction`

`BiFunction<T, U, V>` is equivalent to `(T, U) -> V`
	*Same as `Function`, but takes two parameters*

### `UnaryOperator`

`UnaryOperator<T>` is equivalent to `(T) -> T`

```java
UnaryOperator<String> op = s -> "hello " + s;
System.out.println(op.apply("unary operator"));

// For primitives
IntUnaryOperator negate = x -> - x;
System.out.println(negate.applyAsInt(7));
```
> [!info]
> We take a type, and return the same type. Usually this is used when we apply modifications on an object

### `BinaryOperator`

`BinaryOperator<T>` is equivalent to `(T, T) -> T`
	*It's the same as `UnaryOperator` but it takes two parameters instead of one.*


****
## Method Reference

What we call a method reference is this: `::`
It allows to view a method as an instance from a Functional Interface.
![[reference.png]]

The compiler, in order to find the correct method, will use parameters types and return type of the abstract method to understand which method is referenced by `::`.
![[find.png]]

There are four different usages of the `::` operator.

### Instance Method

`::` sees methods as functions. So, one must not forgot the `this` implicit first argument.
	*`this` is automatically passed by the compiler, that's why we don't have to do it manually*
```java
record Person(String name, int age) {
	boolean isOlderThan(Person person) {
		return age > person.age;
	}
	
	// is the same as
	boolean isOlderThan(Person this, Person person) {
		return this.age > person.age;
	}
}

// Two Person, the first one is the type of "this"
BiPredicate<Person, Person> older = Person::isOlderThan;
```

### Static Method

`A::method` does not mean that the method is static (nor an instance method, actually). The only information it provides is that `method()` is part of the `A` Class.

This can results in conflicts if both a static and an instance method shares a similar signature:
```java
class A {
	void m(A this) { /* ... */ }
	static void m(A a) { /* ... */ }
}

Consumer<A> consumer = A::m; // refuses to compile! both methods are valid, and 
							 // we cannot make a choice
```

### Instance method + attached receiver (bound)

`::` allows to specify a value (like an object) as the receiver for an instance method (the value of `this` in the method).

```java
record Person(String name, int age) {
	boolean isOlderThan(Person person) {
		return age > person.age;
	}
}

var john = new Person("John", 37);
Predicate<Person> isYoungerThanJohn = john::isOlderThan;

// evaluates
john.isOlderThan(somePerson);
```

Here, `john::isOlderThan` is a reference that binds the object `john` to the `isOlderThan` method of the `Person` class.
When this reference is used (e.g., in a `Predicate`), it behaves as if you're calling `john.isOlderThan(person)` for any given `person`.

> [!info] 
> This is similar to Haskell's partial application, where you "attach" one argument of a function (in this case, `john` as the receiver) and leave others open to be provided later (the `person` argument).

### Constructor and Array constructor

It works like this:
```java
// Constructor
record Person(String name) { }
Function<String, Person> factory = Person::new;

// Array constructor
IntFunction<String[]> arrayFactory = String[]::new;
```

### Conversions

`::` can do the following:
- Contravariance of Parameters
```java
class Printer {
    void print(Object obj) {
        System.out.println(obj);
    }
}

// Contravariance: Object parameter is accepted, even if Consumer<String> is expected
Consumer<String> stringPrinter = new Printer()::print;

// Works because String is a subtype of Object
stringPrinter.accept("Hello, World!"); 
```

- Covariance of return type
```java
class Factory {
    Number produceNumber() {
        return 42;
    }
}

// Integer is more specific than Number
Supplier<Integer> numberSupplier = () -> 42;
Supplier<Number> factorySupplier = new Factory()::produceNumber;

System.out.println(numberSupplier.get()); // Prints 42
System.out.println(factorySupplier.get()); // Prints 42
```

- Conversion of primitive types
```java
@FunctionalInterface
interface IntProcessor {
    void process(int value);
}

class Calculator {
    void square(double value) {
        System.out.println(value * value);
    }
}

// Method reference converts int to double automatically
IntProcessor processor = new Calculator()::square;
processor.process(5); // will print 25.0
```

- Boxing/Unboxing (works with varargs)
```java
ToDoubleFunction<String> fun = Integer::parseInt;
Function<String, Integer> fun = Integer::parseInt;
```


****
## Lambda

So, the `::` operator forces us to define an according named method in the class:
```java
class Utils {
	static boolean startsWithAStar(String s) {
		return s.startsWith("*");
	}
}

Predicate<String> predicate = Utils::startsWithAStar;
```

Lambdas allows to **declare anonymous methods**:
```java
Predicate<String> startsWithAStar = (String s) -> s.startsWith("*");
```


The compiler will transform this lambda into a normal static method with a generated name. Our lambda above will be compiled into:
```java
static boolean lambda$1(String s) {
	return s.startsWith("*");
}
Predicate<String> predicate = Utils::lambda$1;
```


> [!tip] 
> It is not mandatory to specify the parameters' type (like for `::`, the types are already specified in the functional interface)
```java
@FunctionalInterface
interface Comparator<T> {
	int compare(T t, T t2);
}

// Redundant declaration of type
Comparator<String> c = (String s1, String s2) -> s1.compareIgnoreCase(s2);

// Better
Comparator<String> c = (s1, s2) -> s1.compareIgnoreCase(s2);
```


### Syntax

We can declare it as a **lambda block**
```java
Comparator<String> c = (s1, s2) -> {
	// Allow for several lines, it's a block...
	return s1.compareIgnoreCase(s2);
};
```

or a **lambda expression**
```java
// The return is implicit as its a one-liner 
Comparator<String> c = (s1, s2) -> s1.compareIgnoreCase(s2);
```


The syntax is different depending on the number of parameters our lambda takes:
```java
// No parameter, we specify an empty parenthesis
() -> 42

// One paramter, no need for parenthesis (optimised declaration, like Scala,
// C# and JavaScript)
s -> s.length()

// More than one parameter, we chain them in the parenthesis
(s1, s2) -> s1.compareTo(s2)
```


### Capture

Variables captured by a lambda **must be assigned only once (effectively `final`)**:
```java
// Does not work
static void foo() {
    for (var loop = 0; loop < 5; loop++) { // loop++, oops
        Runnable runnable = () -> {
	         // Does not compile, because "loop" is modified
            System.out.print(loop);
        };
    }
}

// Works
static void foo() {
    for (var loop = 0; loop < 5; loop++) {
	    var copyOfLoop = loop; // good !
        Runnable runnable = () -> {
            System.out.print(copyOfLoop);
        };
    }
}
```
> [!info]
> C# and JavaScript (with `let`) are smarter and have a specific scenario for this, but in Java we have to do it manually ...

We can also implicitly capture `this`:
```java
record Person(String name, int age) {
	Runnable printer() {
		return () -> {
			// System.out.println(name + " " + age);
			System.out.println(this.name + " " + this.age);
		};
	}
}
```


****
## Stream

Introduced in Java 8, it works in three steps:
1. Create the stream
2. Intermediate operations on data
3. Retrieve output

Stream operations are **lazy** (they aren't executed if not needed)
	*If I apply intermediate operations but never ask for an output, no computation is made*

Streams allows declarative programming in Java, which is a better approach in a lot of situations:
```java
record Employee(String name, int age) { }

// Poop! Code is difficult to read
List<String> findEmployeeNameOlderThan(List<Employee> employees, int age) {
	var list = new ArrayList<String>();
	for(var employee: employees) {
		if (employee.age() >= age) {
			list.add(employee.name());
		}
	}
	return list;
}

// Readable, elegant, easy to change if needed. We directly indicate what we want
List<String> findEmployeeNameOlderThan(List<Employee> employees, int age) {
	return employees.stream()
		.filter(e -> e.age() >= age) // Predicate :)
		.map(Employee::name) // Function :)
		.toList();
}
```


So, we got SQL-like operations:
```sql
SELECT name 
FROM users
WHERE country = ‘France’ ORDER BY id
```
```java
users.stream() // Step 1: Declare stream
	.filter(user -> user.country().equals("France")) // Step 2: Operation
	.sorted(Comparator.comparing(User::id)) // Step 2
	.map(User::name) // Step 2
	.toList(); // Step 3: Retrieve
```


As mentioned above, a stream is made in three parts. This is called the **pipeline**.

### Pipeline - Source (creation)

We can create a stream:
- **From a collection**: `collection.stream()`
- **From values**: `Stream.of(1, 2, 3, ...)`
- **From an array**: `Arrays.stream(array)`

There are three streams for primitive types (avoid boxing)
`IntStream, LongStream, DoubleStream`

### Pipeline - Intermediate operations
(Those operations are applied to every objects in the collection)

`filter(Predicate<E>)`: An object is not propagated if the predicate returns false on it.
![[predicate.png]]

`map(Function<E,R>)`: Transforms each element by calling the function
```java
stream.map(Person::name)
```
![[map.png]]


`flatMap(Function<E, Stream<R>>)`: Transforms each element by flattening a stream
	*This method returns a stream, so each element's output stream can contain from 0 to n elements*
```java
stream.flatMap(p - > p.children.stream())
```
![[flatmap.png]]

`mapMulti(BiConsumer<E,Consumer<R>>)`: For each object, can propagate from 0 to n objects
```java
stream.mapMulti((Person p, Consumer<Person> consumer) - > {
	for (var child : p.children) {
		consumer.accept(child);
	}
})
```
![[mapmulti.png]]

Other convenient operations:
- `.limit(n)`: Filter that only allows `n` elements to pass
- `.skip(n)`: Skips the `n` first elements
- `.distinct()`: Filter that does not allow a similar element to pass two times
- `.sorted(comparator)`: Sorts elements depending on the given comparator

Map operations (`map`, `flatMap` and `mapMulti`) have dedicated alternatives for primitives (avoid boxing):
```java
stream.map(Person::age) → Stream<Integer>
stream.mapToInt(Person::age) → IntStream // Better
```

### Pipeline - Terminal operation (retrieve)

Classic ways of retrieving elements from the stream:
- `.toList()`: Stores elements in an immutable list
	`stream.toList();`
- `.toArray(IntFunction<E[]>)`: Stores elements in an array
	`stream.toArray(Person[]::new);`
- `findFirst()`: Returns the first element as an **OPTIONAL** (as the stream might be empty)
	`stream.findFirst()`

Check elements:
- `.allMatch(Predicate<E>)`: If they ALL validates the predicate
- `.anyMatch(Predicate<E>)`: if AT LEAST ONE validates the predicate
- `.noneMatch(Predicate<E>)`: if NOBODY validates

Aggregate into a single value:
- `.count()`: Returns the number of elements
- `.reduce(T initial, BinaryOperatot<T> combiner)`: Combine elements in pair
```java
stream.mapToInt(Person::salary)
	.reduce(0, Integer::sum);
```

Min/Max on a stream:
- `.min(comparator)`
	`stream.min((p1, p2) - > Integer.compare(p1.age(), p2.age()));`
- `.max(comparator)`
	`stream.max(Comparator.comparingInt(Person::age));`

Additional operations for primitive streams:
- `.min()`, `.max()`, `.sum()`, `.average()`
- `.boxed()`

There is also a `forEach(Consumer<E>)` method that calls the consumer on each element.
1. It follows the order of the collection
2. **DO NOT MAKE A STREAM FOR THE SOLE PURPOSE OF USING FOREACH**
	*There is already a `.forEach` method on collections, use this directly...*

The last way we can do it is via the `.collect(Collector)` method...


****
## Collectors

A collector is able to create a mutable object (a collection, often) and add elements of the stream inside it, and turn it into a non-mutable object.
	*Example 1: Elements will be placed in an `ArrayList` (mutable), and then `List.copyOf()` is called when everything is done.
	Example 2: Strings are linked via a `StringJoiner`, which is turned into a normal `String` at the end.*

The `Collectors` class (with an **s**) contains static methods returning pre-defined Collector.
- `joining(delimiter, prefix, suffix)`: Allows to join strings on a stream with a delimiter (and optionally a prefix and suffix)
```java
stream.map(Person::toString)
	.collect(Collectors.joining(", "))
```
> This only works on streams of subtypes of **`CharSequence`** (the common interface for `String` and `StringBuilder`

- `toList()`, `toUnmodifiableList()`: Stores elements in a mutable / non-mutable list
```java
stream.collect(Collectors.toList());
stream.collect(Collectors.toUnmodifiableList())
```
>`Collectors.toUnmodifiableList()` is very similar to `stream.toList()`, the major difference is that this collector does not accept null values, unlike `stream.toList`.

- `toMap(), toUnmodifiableMap()`: Stores in a mutable / non-mutable map. Each method takes two `Function` as parameters:
```java
// Map<String, Integer>
stream.collect(Collectors.toMap(Person::name, Person::age));
```
> Those methods does not accept two elements returning the same key. Be careful...

- `groupingBy(Function, Collector)`: Groups elements in a map where the key is given by the `Function`. Elements with a similar key are passed to the given collector.
```java
// Map<Integer, List<Person>>
stream.collect(Collectors.groupingBy(Person::age, Collectors.toList()))

// Map<Integer, Long>
stream.collect(Collectors.groupingBy(Person::age, Collectors.counting()))
```
