[[Java]]
****
**Table of Contents**
```table-of-contents
```

****
## Concept

The concept of inner classes is making encapsulation more refined.

In Java 1.0, a public class had to be defined in a file of the same name
Since Java 1.1, a class can also be defined:
- Inside a class, as an **inner class**
- Inside a method, as a **local class** or an **anonymous class**

```java
class Foo {
	class Bar { /* ... */ } // Inner class
}

void method() {
	class Bar { /* ... */ } // Local class
}

void method() {
	Bar anon = new Bar() { /* ... */ } // Anonymous class
}
```


****
## Inner Class

So, an Inner Class is placed inside another.
- It crosses the `.java <=> one class` rule
- It impacts visibility
```java
package fr.umlv.example;

// fr.umlv.example.Foo
class Foo {
	// fr.umlv.example.Foo.Bar
	class Bar { /* ... */ } // only visible if Foo is visible
}
```

Just like a method or a field, an **inner class is a member of the enclosing class**
	*Therefore, it can accept visibility modifiers.*
```java
public class Foo {
	private class Bar { // Foo is visible from the outside, but Bar isn't
		/* ... */
	}
}
```

`private` is **extended to inner classes**
	*The enclosing class can see the inner class' private attributes
	The inner class can see the enclosing class' private attributes*
```java
class Foo {
	private int a;
	void m(Bar bar) { bar.b /* can see "b" field of Bar */ }
	
	class Bar {
		private int b;
		void m(Foo foo) { foo.a /* can see "a" field of Foo */ }
	}
}
```

### JVM

The JVM does not have this notion of inner classes.
	*On disk, they are both separate files*

So, the compiler must do the following to indicate they belong in the same **nest**:
- In the enclosing class: add a `NestMembers` field containing a list of all its inner classes
- In an inner class: add a `NestHost` field containing the enclosing class' name

So, if we have the following:
```java
class Foo {
	class Bar { }
}
```

We will obtain two files:
- Foo.class
- Foo$Bar.class

### Static Inner Class

We can do static inner classes.
In this context, static means **independent from the enclosing class**.
	*So, unlike in C where a struct inside another struct shares the same lifespan, in Java they are independent (if inner is static...)*

```java
class Foo {
	static class Bar { }
}

Foo.Bar bar = new Foo.Bar();
```
> [!info]
> Here, `Foo` is **not loaded into memory**.

You probably have guessed it already, but since they are independent, they cannot access each other's private properties anymore **unless they are static**.
```java
class Foo {
	int a;
	static int b;
	
	static class Bar {
		public void m() {
			/* ... */ a // Doesn't compile
			/* ... */ b // Works because it's static, it's Foo.b
		}
	}
}
```

### Non-Static Inner Class

In fact, a standard inner class possess an hidden reference pointing on the enclosing class.
```java
class Foo {
	int a;
	static int b;
	
	class Bar { // non static
		public void m() {
			a  // <- there is something magic here
			b  // Foo.b
		}
	}
}

// Reality
class Foo {
	int a;
	static int b;
	
	class Bar {
		Foo Foo.this; // hidden field !
		public void m() {
			a  // <- Foo.this.a
			b 
		}
	}
}
```
> [!info]
> As explained before, the inner class depends on the enclosing class (when not static)

For this reason, an instance of the enclosing class must be there so we can instantiate the inner class on it:
```java
class Foo {
	class Bar { /* ... */ }
	
	public static void main(String[] args) {
		// Does not work...
		Foo.Bar bar = new Foo.Bar();

		// Works
		Foo foo = new Foo();
		Bar bar = foo.new Bar();
	}
}
```

An inner class declared inside an interface is always (implicitly) static
An inner record/enum is always (implicitly) static


****
## Local Class

A local class is defined inside a method. It is only visible from the inside of the method.

A local class has access to all the elements of its enclosing class:
```java
class Foo {
	int a;
	void m() {
		class Bar {
			void hello() { … a … } // Foo.this.a
		}
		new Bar().hello(); // this.new Bar().hello()
	}
}
```

A local class can also capture local variables if they are effectively final (like lambdas):
```java
interface Expr {
	int result();
	public static Expr add(Expr left, Expr right) {
		class Add implements Expr {
			public int result() {
				return left.result() + right.result(); // captures left & right
			}
		}
		return new Add();
	}
}
```

A local class cannot implement a sealed interface :(
A local class cannot be static (for now, might be added in a future version of Java)


****
## Anonymous Class

An anonymous class is a **local class without a name**.
It has a special syntax: `new Class() { ... }`

An anonymous class has access to all attributes of the enclosing class:
```java
class Foo {
	int a;
	void m() {
		new Object() {
			void hello() { … a … } // Foo.this.a
		}.hello();
	}
}
```

Since the class has no name, **we cannot reference it's type. We must use `var`**:
```java
class Foo {
	void m() {
		Object o = new Object() { void hello() { ... } };
		o.hello(); // does not work
		
		var o2 = new Object() { void hello() { ... } };
		o2.hello(); // ok !
	}
}
```
> [!info]
> The logic is that this anonymous class will implement/extend the given interface/class

Again, an anonymous class can capture effectively final variables (like a local class).


> [!tip]
> We can make a combo of lambda + var + anonymous class to declare a mutable field:
> ```java
> void foo() {
> 	var value = 3;
> 	IntSupplier supplier = () - > value++; // does not compile
> }
> 
> void foo() {
> 	var box = new Object() {
> 		private int value = 3;
> 	};
> 	IntSupplier supplier = () - > box.value++; // ok !
> }
> ```


****
## Enum

An enum lists all of its instances by their name:
```java
enum Pet { DOG, CAT, LION }
```
> [!info]
> DOG, CAT and LION are **instances of Pet** (the `new` is implicit).

In fact, an enum is simply a class that—implicitly—inherits from `java.lang.Enum`.
For this reason:
1. It can have methods:
```java
public enum Pet {
	DOG, CAT, LION;
	
	public boolean isDangerous() { return this == LION; } // can use "==" !
}
```

2. It is a **closed type**. We cannot add instances to the already existing ones.
3. It cannot be extended or implemented (implicitly `final/sealed`)
4. It cannot take part in inheritance
	*we don't care about that much, since we don't use it :)*

`java.lang.Enum` defines two fields:
- `int ordinal`, which indicates the number of the instance (starting from 0)
- `String name`, which indicates the name of the constant
```java
enum Pet { DOG, CAT, LION }
assert Pet.DOG.ordinal() == 0;
assert Pet.CAT.name().equals("CAT");
```

We can retrieve the constants:
```java
enum Pet { DOG, CAT, LION }

// Retrieve all constants in an array
// Obviously, a defensive copy is made. Mind the performance issue...
Pet[] pets = Pet.values();

// Retrieve a constant by its name
Pet dog = Pet.valueOf("DOG");
```
> [!info]
> Those methods are automatically added by the compiler

### Mutable Enum

An enum can have a private/package constructor (not public!):
```java
public enum Coin {
	DIME(10), QUARTER(50);
	private final int value; // final!
	
	private Coin(int value) { // private constructor
		this.value = value;
	}
}
```
> [!caution]
> It's a bad idea, avoid using this feature

### Enum and Anonymous Class syntax

A constant can use the syntax of anonymous classes:
```java
public enum Pet {
	DOG {
		public void sound() { System.out.println("bark"); }
	}, CAT {
		public void sound() { System.out.println("meow"); }
	}, LION {
		public void sound() { System.out.println("roar"); }
	};
	
	public abstract void sound();
}
```
> [!info]
> If this feature is used, those constants are considered as inner classes of the Enum.
> In this case, the enum is `sealed` instead of `final`.

This works, but a more elegant approach of doing this is to use delegation:
```java
public enum Pet {
	DOG(() -> { System.out.println("bark"); }),
	CAT(() -> { System.out.println("meow"); }),
	LION(() -> { System.out.println("roar"); });
	
	private final Runnable action;
	private Pet(Runnable action) {
		this.action = action;
	}
	
	public void sound() { action.run(); }
}
```
