[[Design Pattern]]
****

An object-oriented architecture must be easy to maintain, test, re-use and upgrade
- **A class must represent a well-identified concept** (otherwise no tests and no re-usability)
- **Limit dependencies in between classes** (otherwise no maintainability and boilerplate)

For the new millennium, Uncle Bob defines 5 guidelines that we must follow to obtain a good object-oriented architecture:
- **S**ingle-responsibility principle
- **O**pen-closed principle
- **L**iskov substitution principle
- **I**nterface segregation principle
- **D**ependency inversion principle
	*He does not explain how to implement those guidelines though...*

**Design Patterns, by respecting those principles, provides a basic structure that helps solving classic OOP issues.**


****
## **S**ingle-responsibility principle

A Class must only have one responsibility, like so:
```java
public class AWTColorConverter {
	public static Color convert (CanvasColor c) {
        return switch (c) {
            case BLACK -> Color.BLACK;
            case WHITE -> Color.WHITE;
            case ORANGE -> Color.ORANGE;
        };
    }
}
```


**Example 1: Client**

```java
class Client {
	private String firstName;
	private String lastName;
	private int streetNumber;
	private String streetName;
	private int zipCode;
	private String city;
	// ...

	public String getName()  {...};
	public String printAddress()  {...};
}
```

How to properly manage addresses when we start managing clients from foreign countries ?
Solution: **Composition + Delegation**
```java
public class Address {
    private int streetNumber;
    private String streetName;
    private int zipCode;
    private String city;

    public String print() {...};
}

public class Client {
    private String firstName;
    private String lastName;
    private Address address; // Composition
    // ...

    public String getName() {...};
    public String printAddress(){
         return address.print(); // Delegation
    }
}
```

**Composition** is simply when an object contains another:
![[composition.png]]

**Delegation** is when an object assign some work to another one. In our code, delegation happens with the printing (we call the `print` method of the contained class. It is its job to print things)

We can even have a better result by making `Address` an interface (in the case where some countries have strict rules or a different way to print things, we can do it):
```java
public class Client {
    private String firstName;
    private String lastName;
    private Address address;
    // ...

    public String getName() {...};
    public String printAddress(){
        return address.print();
    }
}

public interface Address {
    public String print();
}

public class FrenchAddress implements Address {
     // ...
}

public class BritishAddress implements Address {
     // ...
}
```
![[address.png]]


**Example 2: Uber Client**

We begin with the following class:
```java
class UberClient {
    private String firstName;
    private String lastName;
    private long uid;
    private Email email;
    private PhoneNumber phoneNumber;
    private double score;
    private ArrayList<Double> grades; 
    // ...

    public String toString() {...};
    public String toJSON() {...};
}
```

As our project evolves, our class will end up like this in a few months:
```java
class UberClient {
    private String firstName;
    private String lastName;
    private long uid;
    private Email email;
    private PhoneNumber phoneNumber;
    private double score;
    private ArrayList<Double> ratings; 
    // ...

    public String toString() {...};
    public String toJSON() {...};
    public String toHTML() {...};
    public String toHTMLFull() {...};
    public String toXML() {...};
    public String toXMLBackEnd() {...};
}
```

Our class clearly went from a simple representation of a client to a way of exporting its data. Each time we need to export data in a different format, a developer adds a method. Those methods probably introduce a lot of **duplicated code**, and they might even **display information that should not be displayed** (like the `uid` or the list).


The good solution is the following:
```java
public class UberClient {
    // ...
    private UberClientInfo infos() {...};

    public String export(UberClientFormatter formatter) {
        return formatter.fomat(getInfos());
    };
}
```
- We introduce a transparent class `UberClientInfo` (usually a record) that **contains exportable information (only)** of an `UberClient`.
- The `UberClient` class offers no public nor package method to calculate the `UberClientInfo`. It only gives a public method that accepts a formatter (`UberClientFormatter` interface) and returns the result.


****
## **O**pen-closed principle

A class (package or library) should be **closed to modifications** but **opened to extensions**.

Once a class is tested and versioned, we close it and avoid touching the code until we bring new features to it. However, we would like to **extend its capabilities without touching a single line of its code**.


Extensions will be brought by **adding new classes** instead of modifying current ones (e.g, adding a new exportation format for an `UberClient` by adding a new class implementing the `UberClientFormatter` interface, like a `JsonUberClientFormatter` and an `XMLBackUberClientFormatter`).


We keep our `UberClient` class, but now we would like to find a good way to calculate their score. 
	*Note that the algorithm might change in the future (average of all ratings, average of all 100 last ratings...)*

Putting a getter on the `ratings` list would break the encapsulation. The best way to go is to externalise the calculation algorithm.
	*`UberClient` class still owns the data and does not expose them. But when it is time to calculate the score, it will **take the algorithm as a parameter**.*

```java
public class UberClient {
	// ...
	// private double score; (we don't need this anymore)
    private ArrayList<Double> grades;
    
    double getScore(ScoreComputationStrategy strategy){
    	return strategy.computeScore(grades);
    }
}
```

We can remove the `score` property, as it will now be calculated depending on the strategy provided instead of being stored.

We now define the strategies:
```java
@FunctionalInterface // Only one method, so we can make it a FInterface
public interface ScoreComputationStrategy {
    public double computeScore(List<Double> grades);
}

// Implementation 1
public class AverageStrategy implements ScoreComputationStrategy{
     public double computeScore(List<Double> grades){
         return grades.stream()
                      .mapToDouble(x->x)
                      .average()
                      .orElse(5);
     }
}

// Implementation 2
public class AverageBoundedStrategy implements ScoreComputationStrategy{
     private final int maxNbGrades;

     public AverageBoundedStrategy(int maxNbGrades){
         this.maxNbGrades = maxNbGrades;
     }

     public double computeScore(List<Double> grades){
         return grades.stream()
                      .limit(maxNbGrades)
                      .mapToDouble(x->x)
                      .average()
                      .orElse(5);
     }
}
```

We use it like so:
```java
public static void main(String[] args){
	 var ClientUber client = // ...
	 System.out.println(client.getScore(new AverageStrategy()));
	 System.out.println(client.getScore(new AverageBoundedStrategy(100)));
}
```

This idea of externalising the algorithm is called a **Strategy Design Pattern** (hence the naming...)


****
## **L**iskov substitution principle
*This part is about subtypes (which mostly concerns inheritance). This helps to understand why it is a bad habit that must be avoided by explaining what must be guaranteed when using inheritance*

**Children classes must be usable instead of their parent class without breaking the program.**
	*In other words, when we inherit from a class, we must preserve the contract with it*


**Example 1: Employee**

```java
public class Employee {
     private String Name;
     private int salary;
     public String getName() {...};
     public void setSalary(int salary) {...};
     public int getSalary() {...};
}
```

We want to add interns that are paid 0,00001 pesos:
```java
public class Intern extends Employee {
	 @Override	 
     public void getSalary() { return 0; };
}
```

But this can happen:
```java
Employee e = ...;
e.setSalary(500);
assert(e.getSalary() == 500); // NO! We don't respect the contract
```

It will not return what we want in the case of an Intern... We can set the salary to 500, and it ignore it and return 0.
	*As a result, the behaviour of `getSalary()` is inconsistent across `Employee` and `Intern`, breaking the expected contract where `getSalary()` should reflect the value set by `setSalary()`. This inconsistency makes the class hierarchy unreliable and confusing, especially when working with `Employee` references.*


**Example 2: Shapes**

We have the following:
```java
public class Rectangle implements Shape{
    public Rectangle(int upperLeftX, int upperLeftY, int width, int height) {...};

    public void setWidth(int width) {...};
    public int getWidth() {...};
    public void setHeight(int height) {...};
    public int getHeight() {...};

    @Override
    public double distance2(int x, int y) {...};

    @Override
    public void draw(Graphics2D graphics2D) {...}

}
```

We now want to add a `Square` to the `Shape` interface.
Can we make `Square` inherit from `Rectangle` ? 
Can `Rectangle` inherit from `Square` ????

The problem is that `Square` should have the same width and height, but the `Rectangle` class (we are supposed to inherit from) doesn't guarantee us this behaviour. If `Square` inherits from `Rectangle`, enforcing equal width and height could break the expected behaviour of a `Rectangle`, so we are trapped.


Inheritance introduces too many issues, and rarely ends up providing reusable code
	*The child class depends way too much on the parent class. It reduces the amount of code, but the code becomes impossible to re-use...*

**[[02 - SOLID#Historical use of Inheritance|Almost]] no design pattern uses inheritance.**


*Inheritance can be (cautiously) used for code refactoring if **concrete classes inherits from package-visibility abstract classes**. But again, Interfaces are here to give the common type, an abstract class SHOULD NEVER GIVE THE COMMON TYPE as it involves an improper use of inheritance*


****
## **I**nterface Segregation Principle

No clusterfuck interfaces. A straightforward interface with a specific purpose makes the code both easier to understand and maintain.
	*Remain moderate though, else you end up with an interface per method or silly things like this*


****
## **D**ependency inversion principle

Your code should depend on **abstractions** instead of **concrete classes**.
	*So, your classes should implement interfaces, not extend classes*

This is overall better for polymorphism and reduces dependency on concrete classes.
	`Drawing` depends on `Shape`; Not `Line`, `Rectangle`...


****
## Historical use of Inheritance

`HttpServlet` is a J2EE class allowing description of code to execute when a webserver receive an HTTP request:
```java
public abstract class HttpServlet {
	// ...
	public abstract void doGet(HttpServletRequest req,HttpServelResponse resp);
	public abstract void doPost(HttpServletRequest req,HttpServelResponse resp);
}
```

To define your own servlet, you **had** to inherit from `HttpServlet` and re-define the methods (`doGet()`, `doPost()`...)

This **was** one of the only use case of inheritance in design pattern, and even this got **replaced by composition**:
```java
public class HttpServlet{
	private final HttpProcessor get;
	private final HttpProcessor post;

	public HttpServlet(HTTPProcessor get,HttpProcessor post){...}

	public void doGet(HttpServletRequest req,HttpServelResponse resp){
		get.process(req,resp);
	}

	public void doPost(HttpServletRequest req,HttpServelResponse resp){
		post.process(req,resp);
	}
}

@FunctionalInterface
public interface HTTPProcessor{
	public void process(HttpServletRequest req,HttpServelResponse resp);
}
```