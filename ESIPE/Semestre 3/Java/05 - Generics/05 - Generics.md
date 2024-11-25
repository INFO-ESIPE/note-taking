[[Java]]
****
**Table of Contents**
```table-of-contents
```

****
## Concept

Generics were introduced in Java 5. Before that, we had to use unsafe casts:
```java
ArrayList list = new ArrayList();
list.add("hello");
list.add(new URI(...)); // ooooooops, it compiles but shouldn't!

String s = (String) list.get(0); // Unsafe ! Might crash at runtime if we get(1)
```
> The problem is that we only detect the problem at runtime when we cast. We should detect the problem when we add the `URI`...


The concept of generic is the following: **Instances of classes are parameterised**.
The compiler:
- Ensures that we insert/manipulate the correct type
- Returns Objects with the correct type
```java
ArrayList<String> list = new ArrayList<String>();
list.add("hello");
list.add(new URI(...)); // does not compile, good!

String s = list.get(0); // no need for an ugly and unsafe cast anymore, good!
```


### Parameterised Type

We introduce a parameterised type by declaring a type variable:
```java
//                 ↓declare↓
public class ArrayList<E> {
	//            ↓use↓
	public void add(E element) { }
	//   ↓use↓ 
	public E get(int index) { }
}
```
> At compile time, if we use an `ArrayList<String>`, the compiler replaces `E` by `String`.
> So, we obtain the methods `add(String)` and `String get(int)`.

We can specify several of them, by separating the declaration with commas:
```java
public class HashMap<K, V> {
	public void put(K key, V value) { }
	public V get(Object key) { }
}
```


Since this is purely related to an instance, we can not access them with a static method:
```java
class Foo<E> {
	// Works
	E e;
	List<E> list;
	E foo(E e) { return e; }
	void foo2() { E e; }

	// Does not work
	static E e;
	static E bar(E e) { return e; }
	static void bar2() { E e; }
}
```


**Primitives cannot be used in a diamond syntax**. We must use the Wrappers:
```java
ArrayList<int> list = // ... does not compile

ArrayList<Integer> list = // …
list.add(1); // convert int → Integer (boxing)
int value = list.get(0); // convert Integer → int (unboxing)
```
> Generics only accepts reference types...


### Parameterised Method

We would like the same feature for methods:
```java
// Here, it would be nice to specify that types in src and dst should match 
public static void copy(List dst, List src) {
	// ...
}
```

Well, just like we did above for the class, we **declare** a type variable for the method.
	*The declaration must be done after the visibility and before the return type*
```java
//        ↓declare↓            ↓use↓        ↓use↓
public static <T> void copy(List<T> dst, List<T> src) {
	// ↓use↓
	for(T element: src) {
		dst.add(element);
	}
}
```


Declaring a generic in a class or a method **does not exactly have the same meaning**:
```java
// For a class, there exists a T such that
class Stack<T> {
	void push(T t) { }
	void T pop() { }
}

// For a method, for any T such that
class Utils {
	<T> void transfer(Stack<T> s1, Stack<T> s2) { … }
}
```


****
## Erasure

Unlike Java's generics, C++ has a "Template" system.
	*The compiler generates as much classes on disk as there are instances using genericity. So it creates a `HashMap<String, String>`, `HashMap<String, URI>` etc...
	This generates too much code though*

In Java, **generics are** only presents during compilation, but **absent during runtime**.

So, the compiler uses the generic mechanism to ensure the types, but after that, this code is replaced:
- Replaces diamond syntax with a **raw type** alternative (no `< >`)
- Replaces type variables with their **upper bound**
- Inserts casts where they belong
> We end up with a single .class file for all the instantiations.

Example:
```java
// Source code               With Erasure
class Stack<T> {             class Stack {
	void push(T t) { … }         void push(Object t) { … }
	void T pop() { … }           void Object pop() { … }
}                            }

// Usage - Source code
Stack<String> stack = new Stack<String>();
stack.push("hello");
String s = stack.pop();

// Usage - With Erasure
Stack stack = new Stack();
stack.push("hello");
String s = (String) stack.pop();
```


Since generics are gone at runtime, the following operations won't compile:
- `instanceof T` or `instanceof Foo<String>`
- `new T`
- `new T[]` or `new Foo<String>[]`
- `class Foo<T> extends Throwable { … }`
	*Because `catch(Foo<String>)` is impossible...*
*A cast into a generic produces a warning (more on that later)*

Erasure can cause conflicts at compile time if two methods share the same signature besides their use of generics:
```java
// Does not work...
class Foo {
	void bar(List<String> list) { }
	void bar(List<Integer> list) { }
}

// Same if we implement two times a same interface using generics
class Foo implements Bar<String>, Bar<Integer> {
}
```


****
## Generics for Arrays

Due to erasure, it seems we cannot have arrays using generics.
	*But we need those, else how could `ArrayList` and `HashMap` exist?*

We have to use an unsafe cast!
```java
public class ArrayList<E> {
	private E[] elements;
	
	public ArrayList() {
		// We make an array of objects, and then cast it to an array of E
		elements = (E[]) new Object[16]; // Unsafe, compiler emits a warning
	}
	// ...
}
```
> Works during execution since the cast does not appear in the bytecode (`E[]`'s erasure is `Object[]`).

Since we know our cast is safe in this context, we can tell the compiler to not emit a warning, via one `SuppressWarning`:
```java
public class ArrayList<E> {
	private E[] elements;

	@SuppressWarnings("unchecked") // If specified here, it affects the entire function
	public ArrayList() {
		@SuppressWarnings("unchecked") // If specified here, it only affects the line under (better)
		E[] elements = (E[]) new Object[16];
		this.elements = elements;
	}
	// ...
}
```
> Do not abuse this feature. This `SuppressWarning` should only be used in this context, and not to hide problems in your code...


This fixes our issue, but what about Varargs ? Those asks the compiler to create an array.
We can avoid a warning if we use `@SafeVarargs`:
```java
@SafeVarargs
static <T> List<T> asList(T… elements) { /* ... */ }
```
> As it relies on implementation, we can only specify this on methods that cannot be re-defined (static, private, final, or the constructor)


****
## PECS

We can (and should) use **wildcards**.
Let's say we have the following code:
```java
void printAll(List<Object> list) {
	list.forEach(System.out::println);
}

List<String> list = List.of("foo");
printAll(list); // does not compile
```
> Unlike arrays which are covariant, generics are **invariant**...

Here, we would like a way to specify that we want to accept any object that is a subtype of the bound we specify.
We can do this with `? extends`, where `?` indicates "any type", and `extends` indicate that we aim at a **subtype**.
	*In this context, the `extends` keyword is not related to inheritance*
```java
// This code accepts CharSequence or any subtype (e.g. String)
void foo(List<? extends CharSequence> list) {
	// ...
}
```

We can also do the opposite with `? super`, where `super` indicates a super-type of the bound.

A third type of wildcard is the **unbounded wildcard**: `?`, which allows any type.
	*In fact, `?` is an equivalent to `? extends Object`, as any type is a subtype of `Object`*


If we have a public method that uses generics, we follow the **PECS** rules:
- If the type acts as a data **P**roducer, it must be declared with **E**xtends
- If the type acts as a data **C**onsumer, it must be declared with **S**uper
> If the type acts as both (or is a return type), we don't use wildcards
```java
interface Collection<E> {
	public boolean addAll(Collection<? extends E> cs);
}

interface Stream<E> {
	public Stream<E> filter(Predicate<? super E> predicate);
}

public static <E> void copy(List<? super E> dst, List<? extends E> src)
```
