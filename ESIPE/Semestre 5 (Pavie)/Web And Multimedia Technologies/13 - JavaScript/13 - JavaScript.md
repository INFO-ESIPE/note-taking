[[Web And Multimedia Technologies]]
05/12/2024
****
**Note:** This note focuses on the language mechanisms. Interactions with HTML or usage of AJAX will be for next lessons. 
****
**Table of Contents**
```table-of-contents
```

****
## JavaScript?

JavaScript is a high-level, dynamic, untyped, and interpreted programming language. 
	*JavaScript is prototype-based with first-class functions, making it a multi-paradigm language, supporting object-oriented, imperative, and functional programming styles.*

> [!note]
> The standardised name of JavaScript is ECMAScript (as it is developed by Ecma International).

it's main purpose is to integrates with HTML to provide interactive and dynamic functionalities:
- **Direct Inclusion**, with `<script>` tags:
```html
<!DOCTYPE html>
<html>
	<body>
		<script>
			<!-- JavaScript code -->
		</script>
	</body>
</html>
```

- **External files**:
```html
<!DOCTYPE html>
<html>
	<body>
		<script type="text/javascript" src="path-to-javascript-file.js"></script>
	</body>
</html>
```

We can observe the behaviour (and errors) of our JavaScript in the **console**, which is included in all modern web browser's devtools.


****
## Fundamentals

Each line ends with a `;`

### Variables

A local variable is declared with `let`. It can be initialised at the same time as the declaration, or later:
```js
let a = 7;
let b = 42;  // 42

let sum;
sum = a + b; // 42
```
> [!info] 
> The default value of an non-initialised variable is `undefined`. More details on that in the part dedicated to [[13 - JavaScript#Types|types]].

It's scope will be the enclosing block of code. A variable cannot be declared two times, nor cannot be queried if not defined:
```js
console.log(a); // ReferenceError: a is not defined

let x;
let x;          // SyntaxError: redeclaration of let x
```

We can declare constants with the `const` keyword:
```js
const number = 42;

try {
  number = 99;   // Will throw an error. The value is final...
} catch (err) {
  console.log(err); // TypeError: invalid assignment to const 'number'
}
```
> [!info]
> As a constant is final, it needs to be initialised when declared (unlike a `let` which can be initialised later).

Unlike the two previous examples, we can also declare a global variable which will be accessible in the entire file (and not the enclosing block only):
```js
myvariable = 'Hello' // No keyword at the beginning
```

*A fourth way of declaring variables is with the `var` keyword, but var isn't [[13 - JavaScript#Scope|scopped correctly]], so we avoid using it.*

### Types

Three main types: `boolean`, `string` and `number`.
We can retrieve a value's type with the `typeof` operator:
```js
typeof true  // boolean
typeof false // boolean
typeof 747   // number
typeof -34.6 // number
typeof 'foo' // string
typeof "bar" // string
```
> [!note]
> Other types for arrays, objects and functions, but they will be detailed in a dedicated part

### Operators

Classic operators `+`, `-`, `/` and `%` works as expected on numbers:
```js
let a = 13;
let b = 4;
a + b        // 17
a - b        // 9
a * b        // 52
a / b        // 3.25
a % b        // 1
```

Numbers are always **floats**. If the number has no decimal, `.0` is simply not printed.
	*So, the value of `a` is actually `13.0`. There is no separation between float/double and int/long like in decent languages*

Calculations on floating values is secured (no errors):
```js
3 / 0          // infinity
-3 / 0         // -infinity
0 / 0          // NaN (Not a Number)

let x = 0 / 0; // NaN
x == x         // false (!)
```

Test operators (`==`, `!=`, `<`, `<=`, `>`, `>=`, `&&`, `||`) works as expected too. Calculations are rounded to the nearest float:
```js
let a = 0.1;
let b = 0.2;

a + b == 0.3 // false (!)
a + b        // 0.30000000000000004 (!)
```
> [!caution]
> We never use `==` nor `!=` on floating values

`==` and `!=` are defined for all types, they work more or less as expected depending on types:
```js
'hello' == "hello" // true
true != false      // true

4 == "4"           // true
4 != "4"           // false

4 === "4"          // false
4 !== "4"          // true
```
> [!tip]
> Triple `=` also checks if the type is the same. For instance, `4` is a number while `"4"` is a string. `===` will not consider them equal as there is a type mismatch.

The `+` operator can also be used for string concatenation **if one of the operand is a string** (left or right value):
```js
'hello ' + 'Bob' // hello Bob

'hello' + 3      // hello3
3 + 'hello'      // 3hello

'hello' + true   // hellotrue
false + 'hello'  // falsehello
```

### Template Strings

Back quotes (\`) allows to define a string containing expressions.
An expression is introduced by `${...}`:
```js
let name = 'Bob';
`Hello ${name}`       // hello Bob

let x = 21;
`result = ${ x * 2 }` // result = 42
```

### Instruction Flow control

We have the classic `if` and `else`, but we can also use `? :` to simulate an `if` between conditions (ternary operation):
```js
let age = 13;
let text = age < 18 ? 'too young !' : 'go get your beer'; // ifTrue : ifFalse
console.log(text); // too young !
```

`&&` and `||` are lazy.
	*If left part of an `&&` is false, right part won't even be checked
	If left part of an `||` is true, right part won't even be checked*

We have classic `while` and `for` loops as well.
```js
for(let i = 0; i < 4; i = i + 1) { // "i" can be initialised in the loop itself
  console.log(i);
}
```
> [!info] 
> We can exit a loop with `break` and `return`, and skip a loop with `continue`

### Increment

JavaScript borrow C's syntax for this.
	*`i++` = `i = i + 1`
	`i--` = `i = i - 1`
	`i += 2` = `i = i + 2`
	`i -= 2` = `i = i - 2`
	`i++` (resp. `i--`) returns `i` and increments afterwards
	`++i` (resp. `--i`) increments first and then returns the new value of `i`*


****
## Functions

To avoid code duplication (and re-use fragments of code we often call), we declare functions. It can return a value:
```js
function hello(name) {
  return `hello ${name} !`;
}

console.log(hello("Bob"));   // hello Bob !
console.log(hello("Kathy")); // hello Kathy !
```
> [!info]
> If we don't return, `undefined` will be returned.

If too many arguments are fed to a function, they will be ignored. If too few are given, missing ones will be `undefined`:
```js
function fun(x, y) {
  console.log(`${x} and ${y}`);
}

fun('1', '2', '3');    // '3' is ignored
fun('1');              // y is undefined inside the function
```

We can have a (sketchy) optional argument system this way:
```js
function compute(value, optMultiplier) {
	// If optMultiplier is not filled, it defaults to 1
	let multiplier = optMultiplier === undefined ? 1 : optMultiplier;
	return value * multiplier;
}

console.log(compute(64));
console.log(compute(32, 2));
```

### Anonymous functions and Closure

If a function is part of an expression, it doesn't need a name:
```js
let a = function(x) {
  return x + 1;
};
console.log(a); // function(x) { return x + 1; }

function(x) {   // SyntaxError: function statement requires a name
  return x + 1;
}
```

Anonymous functions are represented as "arrow functions":
	(arguments) => expression
	(arguments) => { instructions }
```js
let f = () => 7;
let g = x => x + 7;
let h = (u, v) => u * v + 7;

console.log(f() + g(1) + h(2, 3)); // 28
```
> Parenthesis aren't mandatory if only one argument is passed

Functions can be nested. A function can access variables of the enclosing function:
```js
function printHello(s) {
  function hello() {
    return `Hello ${s} !`;
  }
  console.log(hello()); // can access "s"
}

printHello("Bob"); // Hello Bob !
```
> [!important]
> A nested function using variables from the enclosing function is called a **Closure**.

A function can take another function as argument, and return a function:
```js
function print(allowPrinting) {
  if (allowPrinting) {
    return v => console.log(v);
  } else {
    return v => {};
  }
}

print(true)(42);  // 42
print(false)(42);
```

### Pre-defined functions

`parseInt()` and `parseFloat()` allows to convert a string into a number, and `toString()` is allowing the opposite.

`Math` contains functions for calculation on floating values:
```js
Math.max(3, 7)        // 7
Math.min(3, 7)        // 3
Math.floor(3.6)       // 3
Math.abs(-7)          // 7

Math.PI               // 3.141592653589793
Math.cos(Math.PI)     // -1
Math.sin(Math.PI / 2) // 1

// Get a pseudo-random value
Math.random()         // e.g. 0.19577859792418983
Math.random()         // e.g. 0.43371751314719564
Math.random() * 10    // e.g. 2.084181752170248
Math.floor(Math.random() * 10) // e.g. 9
```


****
## Scope

As briefly mentioned [[13 - JavaScript#Variables|at the beginning]], usage of `var` is independent from the block it is declared in (the `{ }`).
In fact, vars are declared at the beginning of the function.

This means that we can also access the loop variable outside of the loop:
```js
for(var index = 0; index < 2; index++) {
  console.log("loop", index); // loop 0; loop 1
}
console.log("after", index); // after 2
```

`let` fixes this issue:
```js
for(let index = 0; index < 2; index++) {
  console.log("loop", index);
}  // <-- index dies here

console.log("after", index); // ReferenceError: index is not defined
```


****
## Arrays

JavaScript arrays are **heterogeneous**, which means they can contain elements of different types (two strings, a number, a boolean...). They are declared in between `[ ]`, elements are separated by `,`.
```js
[1, 2, 3]
['a', 'b', 'c']

[true, 'hello', 4.5] // heterogeneous
[]                   // empty array
```

We can access elements of an array by its index (starting at 0). if we specify an index which is out of bounds (< 0, > size), `undefined` is returned.

We can go through the array with a classic `for` loop, or with a `for..of`:
```js
// for
function printAll(array) {
  for(let i = 0; i < array.length; i++) {
    let element = array[i];
    console.log(element);
  }
}
printAll([42, true]);

// for..of
function contains(array, value) {
  for(let element of array) {
    if (element == value) {
      return true;
    }
  }
  return false;
}

console.log(contains(['h', 'e', 'l', 'l', 'o'], 'e')); // pass an array litteral
```

If the last argument of a function is declared with `...`, arguments are placed in an array:
```js
function all(text, ...args) {
  for(let arg of args) {
    console.log(text, arg);
  }
}
all("val ", 1, 4); // val 1; val 4
all("empty");
```

The opposite can be done as well: If we call a function and use the `...` (**spread**), values of the array are transferred as separated arguments (in index order):
```js
function minOf3(a, b, c) {
  return Math.min(a, b, c);
}
let array = [40, 2];
let min = minOf3(7, ...array); // a = 7, b = array[0] = 40, c = array[1] = 2
console.log(min); // 2

var array = [1, 2, 3, 4];
let array2 = [0, ...array]; // Also works for creating new arrays
```

### Methods on arrays

An array is considered an "object", as we explain [[13 - JavaScript#Dictionary (Object)|later on]]. So, they come with their own functions and properties that helps us manipulating the data inside it.

The `length` property allow us to know the size of the array:
```js
let array = [1, 2, 4];
array.length;    // property
array.slice(1);  // function (method)
```
> [!note]
> Mind the usage of `()`. More details on the next chapter

`indexOf()` uses `==` to find the first occurrence of an element, and returns its index. `lastIndexOf()` does the same but on the last occurrence.

`join(delimiter)` allows to create a string by separating all elements of the array with a delimiter:
```js
let array = [1, 2, 5, 8]; // [1,2,5,8]
array.join(","); // 1,2,5,8
array.join(" "); // 1 2 5 8
```

`string.split(delimiter)` does the opposite: it is called on a string, and returns an array by splitting it by a delimiter:
```js
let text = "1, 2, 5, 8";
text.split(",");  // [1, 2, 5, 8]
text.split(", "); // [1,2,5,8]
```

`push()` allows to place an element at the end of the array.
	*There is also a `pop` method which removes the last element, allowing us to simulate a stack...*

`shift()` pads all elements to the left, `unshift()` to the right.

`slice(start, end)` allows to copy a subpart of the array:
```js
let array = ['a', 'b', 'c', 'd', 'e']; // [a,b,c,d,e]
array.slice();                         // [a,b,c,d,e]
array.slice(1);                        // [b,c,d,e]
array.slice(2, 4);                     // [c,d]
```
> Unlike previous methods, this does not modify the array we call it on. It returns a new array.

`sort()` allows to sort the array. The behaviour can be weird when our array contains numbers, as the algorithm was designed to sort string elements:
```js
let array = [1, 10, 2, 100]; // [1,10,2,100]

array.sort();                // [1,10,100,2], not really what we want...
```
To fix this, we can pass a comparison function to the `sort()` method:
```js
/*
* Comparison function must be C compliant:
* returns 0 if the numbers are the same, a negative value if the first value is smaller, and a positive value in the opposite situation.
*/
function cmp(v1, v2) {
  return v1 - v2;
}

let array = [1, 10, 2, 100];
array.sort(cmp);
console.log(array);
```
> Like a comparator in Java, kind of...


****
## Dictionary (Object)

An associative array (A.K.A. map, symbol table, or dictionary) is an abstract data type composed of a collection of `key, value` pairs, such that each possible key appears just once in the collection.

In JavaScript, we declare it with `{ }`, where the key and the value are separated by a `:`, and couples are separated by `,`:
```js
{'key': 'value' } 
{ wheels: 4, seats: 2 }

{ color: 'red', align: true }
{} // empty
```
> [!info]
> Keys must be strings

We retrieve the value associated to a key via the `.` (dot):
```js
let car = { wheels: 4, seats: 2 };
car.wheels     // 4
car.seats      // 2
car["seats"]   // Works as well, syntactic sugar

car.seats = 4; // We can change values as well

car.foo        // undefined 
```
> As shown above, `undefined` is returned if we query a key that doesn't exist in the map

As explained above, keys must be unique. If we define a same key two times, only the later is kept:
```js
let dict = { dog: 3, dog: 4 }; // {"dog":4}
```

In JavaScript, dictionaries are called objects (which might be confusing when you come from the OOP world).
	*Try to see each couple as an object's field, more like a C struct in fact...*
`toString` on objects aren't providing much information, we prefer to use `JSON.stringify()` instead:
```js
let obj = { peer: 3 }; // {"peer":3}
obj.toString()         // [object Object] (not very useful)

JSON.stringify(obj)    // {"peer":3}
```
> JSON is the JavaScript Object Notation. Its mostly used to represent objects in a transportable and standardised format, like XML but less verbose and with lack of verification mechanisms.

As mentioned in [[13 - JavaScript#Types|the type chapter]], "objects" are types, but they aren't primitive types.
So, `==` compares in memory address, not values (like in Java).
If we want to compare two objects, we must compare all fields:
```js
let p1 = { name: "Bob", age: 14 };
let p2 = { name: "Bob", age: 14 };
let p3 = p1;

p1 == p2                               // false (not same address)
p1 == p3                               // true
p1.name == p2.name && p1.age == p2.age // true
```

`Object.keys` returns all the keys of the object (equivalent to `keySet` on Java maps):
```js
let letters = { 'a': 1, 'b': 2 }; // {"a":1,"b":2}
Object.keys(letters);             // [a,b], returns an array filled with the keys
```


****
## Classes

In object-oriented programming, a class is an extensible program-code-template for creating objects, providing initial values for state (member variables) and implementations of behaviour (member functions or methods).

So, a class defines a template for object creation. Like in Java, we create a new object with the `new` keyword:
```js
class Car { 
	// ...
}

let car = new Car();
console.log(car);    // {}
```

A class can contain functions associated to each object. In this context, they are called **methods**:
```js
class Hello {
  say(person) {
    console.log(`Hello ${person}`);
  }
}

let hello = new Hello();
hello.say("Bob");
```
> As we can see, those methods depends on the object. But how ?

In fact, each method has an implicit `this` parameter, which corresponds to the object on which we call the method:
```js
class Person {
  hello(/* this */) {
    console.log(this.name);
  }
}

let person = new Person();
person.name = "Bob";
person.hello(); // Bob
```
> We have the same thing in Java. this is always the first argument of methods, but we tend to forget it because this step is done for us.

In general, we avoid initialising fields one by one from the outside when we build an object. We'd rather use a mechanism called the **constructor**, which is a method dedicated to object creation:
```js
class Person {
  constructor(name) {
    this.name = name;
  }
  hello() {
    console.log(this.name);
  }
}

let person = new Person("Bob");
// person.name = "Bob";
person.hello(); // Bob
```

Note that `this` value is `{}` at the beginning of the constructor, as the object isn't instantiated yet (it will evolve as we set the fields):
```js
class Point {
  constructor(x, y) {
    console.log(this); // {}
    this.x = x;
    console.log(this); // {"x":1}
    this.y = y;
    console.log(this); // {"x":1,"y":3}
  }
}

new Point(1, 3);
```

If we want to access the fields of an object, we have to explicitly use `this` (unlike Java and C++ where this is done implicitly):
```js
class Car {
  constructor(driver) {
    this.driver = driver;
  }
  driverName() {
    return driver;  // oops, should be this.driver
  }
}

let car = new Car("Bob");
console.log(car.driverName()); // ReferenceError: driver is not defined
```

### Deeper dive - Prototypes
*This is mostly useful for JavaScript 6 and below*

How does the `class` keyword really works ?
Well in fact, a JavaScript class is a function:
```js
class Car {
  constructor(driver) {
    this.driver = driver;
  }
}

console.log(typeof Car);              // function
console.log(Car instanceof Function); // true
```

And a method is a function stored in an object where its name serves as the key:
```js
const car = {
  driver: "Bob",
  driverName: function() { return this.driver; }
};
console.log(car);              // {"driver":"Bob"}
console.log(car.driverName()); // Bob
```
> [!info] 
> We call those "object literals". Here, our "car" object has one data property (`driver`), and a method (`driverName`).

`Object.create(prototype)` allows to create an object via a prototype, which is another object in which to look for if data is missing in the first object:
```js
let oldPets = {};
oldPets.snoopy = "dog";
oldPets.garfield = "cat";
let newPets = Object.create(oldPets); 
newPets.goose = "flerken";
console.log(newPets.goose);  // flerken
console.log(newPets.snoopy); // dog (found in the prototype)
```

We can retrieve the value of the prototype with the `__proto__` key:
```js
let oldPets = {};
oldPets.snoopy = "dog";
oldPets.garfield = "cat";
let newPets = Object.create(oldPets);
newPets.goose = "flerken";
console.log(newPets);           // {"goose":"flerken"}
console.log(newPets.__proto__); // {"snoopy":"dog","garfield":"cat"}
```
> [!tip]
> It is recommended to try all of this (and experiment) in a devtool console. Try this on a website's console. If you add the dot (`.`) right after the object name, all possible options will be listed. You will see that `__proto__` is one of them, but a lot more exists !

If we want to share methods, we put them in the prototype we use, so we can access them:
```js
let methods = {
  driverName: function() { return this.driver; }
};

let car1 = Object.create(methods);
car1.driver = "Bob";
let car2 = Object.create(methods);
car2.driver = "Ana";
console.log(car1.driverName()); // Bob
console.log(car2.driverName()); // Ana
```

For old versions of Js, we used to have a function acting as the constructor, and the prototype of this function was containing all the methods:
```js
function Car(driver) { this.driver = driver; }

// Put this method in the prototype, which acts as a container of methods
Car.prototype.driverName = function driverName() {
  return this.driver;
};

let car = new Car ("Bob");
console.log(car.driverName()); // Bob
```

==This prototype feature is kind of unique to JavaScript. The `class` notation is way more popular (in Java, Rust, C++, Swift ...), which is why we better understand the later...==


****
## JavaScript and Functional Programming

Functional programming is a programming paradigm that treats computation as the evaluation of mathematical functions and avoids changing-state and mutable data (side effects).

Js allows two syntax for anonymous functions:
```js
let f = function(v1, v2) {
  return v1 + v2;
};
console.log(f);  // function(v1, v2) { return v1 + v2; }

let f2 = (v1, v2) => v1 + v2; // Similar to lambda syntax :)
console.log(f2); // (v1, v2) => v1 + v2
```

`forEach(function)` allows to apply a function to all elements of a collection:
```js
let array = [1, 2, 5, 6, 7];
array.forEach(v => console.log(v)); // 1\n2\n5\n6\n7
```

`map(function)` returns an array containing the result of applying the function to each element of the collection:
```js
let array = [1, 2, 5, 6, 7];

 // v is the current element, it becomes doubled in the new array
let array2 = array.map(v => 2 * v);

console.log(array2); // [2,4,10,12,14]
```

`filter(function)` creates an array where we only keeps elements where the function returns true on them:
```js
let array = [1, 2, 5, 6, 7];
let array2 = array.filter(v => v % 2 == 0); // Only true for even numbers
console.log(array2); // [2,6]
```

`some` returns true if any property is satisfied, and `every` if all of them satisfies:
```js
function greaterThan(max) {
  return v => v >= max;
}

let array = [12, 32, 4];
console.log(array.some(greaterThan(10)));  // true
console.log(array.every(greaterThan(10))); // false, because of the last element
```
