[[Java]]
****
**Table of Contents**
```table-of-contents
```

****
## Collection API

The API was introduced with Java 1.2 (in 1998)
	*Before that, we could only use Vector, Stack and Hashtable (most are deprecated now)*

### Interfaces and implementation

The (basic) API looks like this:
![[ESIPE/Semestre 3/Java/06 - Collections/api.png]]

We have two kinds of implementation:
- **Named**: `ArrayList`, `HashSet`, `ArrayDeque`, `HashMap`
- **Anonymous**: `List.of()`, `Arrays.asList()`, `List.subList()`, `Map.copyOf()`

Those interfaces are parameterised (generics).
	`List<E>`, `Set<E>`, `Deque<E>` and `Map<K,V>`
> Each element E/V or key K must implement the following methods:
> - `equals(Object)`
> - `hashCode()`
> - `toString()`

Here is an example of what happens if we don't define `hashCode`:
```java
class Person {
	private int age;
	public Person(int age) { this.age = age; }
	
	public boolean equals(Object o) {
		return o instanceof Person(int age) && age == this.age;
	}
}

var person = new Person(32);
var set = Set.of(person);
System.out.println(set.contains(new Person(32))); // false...
```

As explained [[ESIPE/Semestre 3/Java/01 - Fundamentals/01 - Fundamentals#Encapsulation|here]], we take into account **design by contract**. Once an element is inserted in the collection, we can not modify it anymore:
```java
class Person {
	// ...
	public int hashCode() { return age; }
}

var person = new Person(32);
var set = Set.of(person);
person.age = 23; // mutation, not good
System.out.println(set.contains(person)); // false
```


Should we manipulate a collection by its implementation or its interface ?
```java
// by interface
List<String> list = new ArrayList<>();

// by implementation
ArrayList<String> list = new ArrayList<>()
```

We have a problem for each choice:
- If a method takes an implementation as parameter, we cannot give another
```java
public void m(ArrayList<String> list) { } // Can't give another type of List
```

- An operation on an interface will have a **varying complexity** depending on the underlying implementation
```java
public void m(Set<String> list, String value) {
	set.contains(value);  // HashSet or Set.of(): O(1)
						  // TreeSet: O(ln n)
}
```

