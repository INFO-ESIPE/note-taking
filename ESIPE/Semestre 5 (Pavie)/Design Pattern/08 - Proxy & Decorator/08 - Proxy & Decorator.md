[[Design Pattern]]
****
**Table of Contents**
```table-of-contents
```

****
## Concept

Those two patterns allows to add features to a class without opening it and adding responsibilities to it. We use **delegation**.
	*So we respect both the Open-Close principle and the Single Responsibility principle*


****
## Decorator

A decorator is a simple way to a dynamically enhance an existing behaviour using composition.
The decorator will implement an interface `A`, and take an element of `A` at initialisation step (in the constructor).
	*The logic is: Our decorator will **implement the same methods as the underlying class it wraps**. When one of these methods is called on the decorator, it will delegate the call to the wrapped object’s method. 
	However, the decorator can also perform **additional actions before and after delegating the call**, allowing us to enhance or modify the behaviour of the original method.*

![[decorator.png]]

### Example 1: Ducks

Let's take this as our example:
```java
public interface Duck {
    void quack();
}

public class RegularDuck implements Duck {
    @Override
    public void quack() {
        System.out.println("Quack !");
    }
}

public class AtomicDuck implements Duck{
    @Override
    public void quack() {
        System.out.println("Atomic Quack");
    }
}
```
> We would like to be able to have the possibility to add a log when calling `quack()`.

We can have a **decorator** class called `LoggerDuck`, which implements `Duck` and will take any `Duck` when constructed:
```java
public class LoggedDuck implements Duck {
    private static final Logger LOGGER =
    		 Logger.getLogger(LoggedDuck.class.getClass().getName());

    private final Duck duck; // Delegate

    public LoggedDuck(Duck duck) {
        this.duck = duck;
    }

    @Override
    public void quack() {
        LOGGER.log(Level.INFO,"Call to Quack!");
        duck.quack();
    }
}

// Usage
public class Ducks {
    public static void main(String[] args) {
        Duck duck1 = new RegularDuck();
        Duck duck2 = new LoggedDuck(new RegularDuck());
        Duck duck3 = new LoggedDuck(new AtomicDuck());
        duck1.quack();
        duck2.quack();
        duck3.quack();
    }
}
```
> `LoggedDuck` acts kind of like a wrapper around the concrete `Duck` we want to log.

### Example 2: Coffee

We have coffees:
```java
interface Coffee {
  long cost();
  String ingredients();
}

record SimpleCoffee(long cost) implements Coffee {
  @Override
  public String ingredients() {
    return "Coffee";
  }
}
```

We now want to represent coffees with more ingredients. A clever way of doing this is by taking any coffee, increase its price and add the new ingredient, like so:
```java
// Example with a coffeee adding milk to a normal coffee
record WithMilk(Coffee coffee) implements Coffee {
  @Override
  public long cost() {
    return coffee.cost() + 50;
  }
  
  @Override
  public String ingredients() {
    return coffee.ingredients() + ", Milk";
  }
}
```
> `WithMilk` is a coffee, but with milk. In itself, it remains a coffee (which is why it implements the interface), and it adds a layer of milk *to any coffee* (it takes as parameter). 

If we want to go further, we could have a sealed hierarchy of coffees so the user can only use ours:
```java
// Declare
sealed interface Coffee permits SimpleCoffee, WithMilk, WithSprinkles {
  long cost();
  String ingredients();

  static Coffee simple(long cost) {
    return new SimpleCoffee(cost);
  }
  default Coffee withMilk() {
    return new WithMilk(this);
  }
  default Coffee withSprinkles() {
    return new WithSprinkles(this);
  }
}

// Usage
Coffee coffeeWithMilkAndSprinkles = Coffee.simple(100)
    .withMilk()
    .withSprinkles();
```


****
## Proxy

A proxy is similar to a decorator, it follows the same logic.
The only difference is that the proxy is **supposed to be hidden from the user, while the decorator is clearly visible.**
	*User only sees the interface.*
