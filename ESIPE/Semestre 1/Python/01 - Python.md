[[Python]]
***
**Note:** We'll consider you know the basics (OOP, Concurrent programming and programming overall, what's an interpreter, how to use a command line...)
It is recommended to use PyCharm community edition for this class as it is beginner friendly. Feel free to keep using neovim though. 
***
**Table of Contents**
```table-of-contents
```

****
## Fundamentals

Multi-paradigm (OOP, functional), interpreted, dynamically typed (resolved at runtime).
Python scripts execute top-to-bottom. Code blocks are defined by indentation (4 spaces PEP 8 standard).
```python
user = "John"            # Variables are dynamically typed
print(f"Hello {user}!")  # f-strings format variables directly
```

Python variables are **references** to objects. Assigning `a = b` does not copy the object; both point to the same data. We use `copy()` or `deepcopy()` for explicit copies.

An interactive python interpreter can be summoned from the terminal, usually stored in `/usr/local/bin` or `/usr/bin`:
```bash
$ python3.13
Python 3.13 (default, April 4 2023, 09:25:04)
[GCC 10.2.0] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> the_world_is_flat = True
>>> if the_world_is_flat:
...    print("Be careful not to fall off!")
...
Be careful not to fall off!
```

Source code files can begin with an UNIX shebang line to explicitly specify which interpreter to use, instead of relying on the default choice.
	*It also allows to run the script as an executable without typing `python3` explicitly*

### Variables and types

As mentioned above, it's dynamically typed. However, it is possible to use **type hints** (PEP 484), which are optional static typing via annotations.
```python
def multiply(a: int, b: int) -> int:
    return a * b

# Union types (Python 3.10+), value accepts both int and strings
def show_value(value: int | str) -> None:
    print(value)

# Type aliases, defined with the "type" keyword
type Vector = list[float]

def scale(scalar: float, vector: Vector) -> Vector:
    return [scalar * num for num in vector]
```

We can cast types, note that it doesn't mutate the entity:
```python
age = 15
name = ""
bool(name)     # False, useful to quickly test for empty strings
age = str(age) # '15', it is now a string
age += 1       # TypeError, we can't concat an integer to a string
age += "1"     # '151'
```

### Flow control

The `for` statement is different from what we know in C. Here, it iterates over a sequence of items, like a foreach.
Like for Java, we need to be cautious when traversing a collection we modify.
```python
# Dictionary
users = {'Hans': 'active', 'Éléonore': 'inactive', '景太郎': 'active'}

# Iterate over a copy
for user, status in users.copy().items():
    if status == 'inactive':
        del users[user]
```

`for` can leverage the `range()` function if we want to iterate over a sequence of number like we used to do:
```python
for i in range(5):
    print(i)                # 0, 1, 2, 3, 4

list(range(5, 10))          # [5, 6, 7, 8, 9], in boundary included and out excluded, like in Java
list(range(0, 10, 3))       # [0, 3, 6, 9], third parameter used to specify an increment of 3 instead of 1 (default)
list(range(-10, -100, -30)) # [-10, -40, -70]

a = ['Mary', 'had', 'a', 'little', 'lamb']
for i in range(len(a)):     # Common way to iterate over a list
    print(i, a[i])
```

There is a `pass` statements that does nothing (lol). It's used for active waiting or placeholders I guess
```python
def hola(*args):
    pass   # Like a todo
```

The switch case is called `match` here, and it supports pattern matching (like new switch in Java):
```python
def http_error(status):
    match status:
        case 400:
            return "Bad request"
        case 401 | 403:
		    return "Not allowed"
        case 404:
            return "Not found"
        case 418:
            return "I'm a teapot"
        case _:
            return "Something's wrong with the internet"
```

Membership operators (`in`, `not in`) can be used to check if a value is found in a sequence:
```python
students = {"Bobberto", "Goobert"}
student = input("Enter name of new student: ")
if student in students:
	print(f"{student} was found")
else:
	print(f"{student} was not found")
```

### Functions

Declared with the `def` keyword. There is no procedure in python, as all functions returns something.
	If no return specified, returns `None`
```python
def add(a: int, b: int) -> int:  
    """  
    Adds two integers and returns the result.  

    Args:  
        a (int): The first integer.  
        b (int): The second integer.  

    Returns:  
        int: The sum of `a` and `b`.  
    """ # Documentation string is declared inside the function, like so
    return a + b 

def ask_ok(prompt, retries=4, reminder='Please try again!'): # Support default values, like PHP
    while True: # Booleans begins with a capslock  
        reply = input(prompt)
        if reply in {'y', 'ye', 'yes'}:
            return True
        if reply in {'n', 'no', 'nop', 'nope'}:
            return False
        retries = retries - 1
        if retries < 0:
            raise ValueError('invalid user response')
        print(reminder)

# Function can be called with positional arguments, or keyword arguments
ask_ok("Hello")
ask_ok("Hello", retries=5)
ask_ok("Hello", 12, "Try again please")
```
> [!info]
> Default arguments are evaluated **once** when the function is defined. 
> For mutable defaults (e.g., `def f(a=[])`), this can cause unexpected behaviour. It is better to use `None` for mutable defaults along with a safe initialisation:
> ```python
> def f(a=None):
>     a = a or []  # Safe initialization
> ```

Coming from the functional world, python has lambdas:
```python
add = lambda x, y: x + y  
print(add(2, 3))  # 5

numbers = [1, 2, 3]  
squared = [x ** 2 for x in numbers]
print(squared)  # [1, 4, 9]

even_numbers = list(filter(lambda x: x % 2 == 0, [1, 2, 3, 4]))  
print(even_numbers)  # [2, 4]
```

### Data structures

Classic list, which comes with util methods (`append`, `extend`...).
```python
fruits = ['orange', 'apple', 'pear', 'banana', 'kiwi', 'apple', 'banana']
fruits.count('apple')  # 2
fruits.index('banana') # 3
fruits.index('banana', 4)  # 6, it finds next banana starting at position 4
fruits.reverse()           # Modifies the list, fruits ['banana', 'apple', 'kiwi', 'banana', 'pear', 'apple', 'orange']
fruits.append('grape')     # fruits ['banana', 'apple', 'kiwi', 'banana', 'pear', 'apple', 'orange', 'grape']

# Lists can also be used as a stack, but it's "append" instead of "push"
fruits.pop()               # 'pear'
stack.append('strawberry')

# Lists can also be used as a deque (similar to java), but through the "collections" library ......
from collections import deque
queue = deque(["Eric", "John", "Michael"])
queue.append("Terry")           # Terry added
queue.append("Graham")          # Graham added
queue.popleft()                 # Eric removed
queue                           # deque(['John', 'Michael', 'Terry', 'Graham'])

# Nested lists
matrix = [
    [1, 2, 3, 4],
    [5, 6, 7, 8],
    [9, 10, 11, 12],
]
```

Python has tuples, an immutable collection composed of several values separated by commas:
```python
t = 12345, 54321, 'hello!'
t[0]                   # 12345
t                      # (12345, 54321, 'hello!')

u = t, (1, 2, 3, 4, 5) # Nested tuples
u                      # ((12345, 54321, 'hello!'), (1, 2, 3, 4, 5))

t[0] = 88888           # Immutable !!! But objects it contains can be mutable
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: 'tuple' object does not support item assignment

empty = ()             # Empty tuple
singleton = 'hello',   # Tuple with single value, must end with a disgusting trailing comma
```
> [!info]
> In general, we use tuples in an heterogeneous manner (store objects of the same type).
> We can do **sequence unpacking** on the tuples
> ```python
> x, y, z = t
>```

Python has sets, useful to avoid storing duplicates.
An important structure is the dictionary (it's a map):
```python
tel = {'jack': 4098, 'sape': 4139}
tel['guido'] = 4127
tel # {'jack': 4098, 'sape': 4139, 'guido': 4127}


for k, v in knights.items(): # Same as foreach($k => $v) in php
    print(k, v)
```

### IO

Several ways to format the strings:
```python
year = 2016
event = 'Referendum'
f'Results of the {year} {event}' # Formatted string litterals, 'Results of the 2016 Referendum'
print(f'The value of pi is approximately {math.pi:.3f}.') # 'The value of pi is approximately 3.142.'

yes_votes = 42_572_654
total_votes = 85_705_149
percentage = yes_votes / total_votes
'{:-9} YES votes  {:2.2%}'.format(yes_votes, percentage) # format method, ' 42572654 YES votes  49.67%'

s = 'Hello, world.'
str(s)   # Typecasting that converts anything to string; very useful for quick debug, 'Hello, world.'
str(1/7) # '0.14285714285714285'
```

 Basics with standard IO:
```python
print("Hello")
print("Hello", "how", "are", "you") # Separates each with a space, like a collector in Java
print("No linejump", end="")        # Print does a line jump by default, this allows to replace the end character

age = int(input("How old are you? ")) 
age += 1 # Typecast to int necessary, anything from stdin is considered a string
print(f"You are {age} years old.")
```

Basic operations to read/write files:
```python
f = open('workfile', 'w', encoding="utf-8")   # Returns a file object
with open('workfile', encoding="utf-8") as f: # Safer method, it auto-closes the file (like a try-with-resources)
    f.read()     # 'This is\nthe entire file.\n'
    f.read()     # '', exhausted
    f.seek(0)    # Go to the 0th byte in the file
    f.readLine() # 'This is\n'
    f.readLine() # 'the entire file.\n'
    f.write('This is a test\n') # 15 (the number of characters added)

f.closed # True, it did close the file for us
```
> [!info]
> Can do a basic try-finally that closes the resource too + catches any error:
> ```python
> try:
>     f = open("file.txt")
> except IOError:
>     print("File error")
> finally:
>     f.close()  # Always executed
> ``` 

### Errors & Exceptions

Similar mechanism to `try-catch` in most languages:
```python
while True:
    try:
        x = int(input("Please enter a number: "))
        break
    except (ValueError, TypeError) as e: # The second can't happen, its just to show we can chain them
        print("Oops!  That was no valid number.  Try again...")
        print(type(inst))    # the exception type
	else: # Optional else clause, this code is executed if no exception is raised
		print("Ok!")
```

`BaseException` is the base class for exceptions. The `Exception` subclass is dedicated to non-fatal exceptions.
If an exception is not a subclass of the later, it should not be handled and we should terminate the program.
	*Like in Java, we can catch `Exception` as a wildcard for all subclasses*

We can manually raise exceptions if needed, and even chain them:
```python
def func():
    raise ConnectionError('SQL Error 19292918339xdDd')

try:
    func()
except ConnectionError as exc:
    raise RuntimeError('Failed to open database') from exc
```

A good way to handle—potentially unrelated—Exceptions is to use the `ExceptionGroup`, which wraps a list of exception instances so that they can be raised together.
```python
def f():
    raise ExceptionGroup(
        "group1",
        [
            OSError(1),
            SystemError(2),
            ExceptionGroup(
                "group2",
                [
                    OSError(3),
                    RecursionError(4)
                ]
            )
        ]
    )

try:
    f()
except* OSError as e:
    print("There were OSErrors")
except* SystemError as e:
    print("There were SystemErrors")
```
> [!info]
> Here, `except*` (instead of `except`) allows to selectively handle only the exceptions in the group that match a certain type (it extracts them correctly).


***
## Modules
### Concept

Modules are Python files containing reusable code (functions, classes, variables) that can be imported into other scripts using the `import` statement. They help organise code, promote re-usability, and avoid repetition.

```python
import math                     # Import the entire module  
import math as m                # Import with an alias
from math import pi as p        # Import only `pi` from the module (alias works here too)  
from math import *              # Import everything (not recommended)  

print(math.pi)  # Output: 3.141592653589793
```
> [!tip]
> `help("modules")` lists all modules natively included in the standard library.

### `__name__ == __main__`

For your file to be imported or run standalone, it is necessary for functions and classes to be re-used without necessarily entering the `main` function.
	*If we are developing a library, we expect other pieces of code to import our script but only uses specific functions we provide, nothing more. The problem is that if we have some code wandering around, it will automatically executes when we import our file*

Python files comes with several special variables. One of the most important being **the `__name__` variable**.
- When a file is **run directly**, `__name__` is set to `'__main__'`
- When a file is **imported as a module**, `__name__` is set to the module's name (e.g., `'math'`)

A common practice is to use the following block to ensure that certain code (e.g., tests, main logic) only runs when the script is executed directly, not when it is imported as a module.
```python
# script.py
def greet(name: str) -> str:  
    return f"Hello, {name}!"  

def main():  
    print(greet("Alice"))  

if __name__ == '__main__':  
    main()

# script2.py
import script  

print(script.greet("Bob"))
```
```bash
$ python3 script.py  # Executed directly
Hello, Alice!

$ python3 script2.py # Executed indirectly
Hello, Bob!
```
> [!info]
> The `main()` function is **not naturally executed** when the script is imported.

The `dir()` function lists all names (functions, variables, modules...) a module defines. Without argument, it simply lists all names in the current scope, including special attributes like `__name__`.
```python
import sys

print(dir())     # Lists all names in the current scope  
print(dir(sys))  # Lists all names defined by the "sys" module
print(__name__)  # Output: '__main__' when run directly
```

Overall, this forces you to put your code into *reusable functions*. 

### Packages

Packages are a way of structuring Python’s module namespace by using “dotted module names” (similar to Java).

> [!example]- Directory structure
> ```bash
> sound/                          # Top-level package
>       __init__.py               # Initialize the sound package
>       formats/                  # Subpackage for file format conversions
>               __init__.py
>               wavread.py
>               wavwrite.py
>               aiffread.py
>               aiffwrite.py
>               auread.py
>               auwrite.py
>               ...
>       effects/                  # Subpackage for sound effects
>               __init__.py
>               echo.py
>               surround.py
>               reverse.py
>               ...
>       filters/                  # Subpackage for filters
>               __init__.py
>               equalizer.py
>               vocoder.py
>               karaoke.py
> ```
> 

```python
import sound.effects.echo
from sound.effects import echo # Does the same

echo.echofilter(input, output, delay=0.7, atten=4)
```

Here, the `__init__.py` files are required to make Python treat directories containing the file as packages.

When packages are structures into sub-packages (like the sound package above), you can use absolute and relative imports, the later being based on the current module's name.
```python
from . import echo
from .. import formats
from ..filters import equalizer
```


***
## Virtual environment
### Concept and manual use

A virtual environment is an isolated Python environment to **manage project-specific dependencies without conflicts** (especially because most dependencies we will use aren't from the standard library).
IDEs like PyCharm makes this step easier, but you can still do it manually:
```python
python3 -m venv my_project_env      # Create a venv
source my_project_env/bin/activate  # Activate (Linux/macOS)
my_project_env\Scripts\activate.bat # Activate (Windaube)
pip install requests numpy          # Install packages
deactivate                          # Deactivate

pip freeze > requirements.txt       # Export installed packages (yes, its specified in a simple text file)
pip install -r requirements.txt     # Install from a requirements file
```
> [!info]
> Activating a venv modifies the shell’s `PATH` to prioritise the venv’s Python. It also sets `VIRTUAL_ENV` to point to the venv directory.
> When deactivating, it unsets `VIRTUAL_ENV`, removes the venv’s `bin` from `PATH`, and restores the shell to use the **system Python** and globally installed packages.

### pip & pipx

Let's dig into that `pip` keyword we saw above.
Pip is python’s default package installer. It can install packages both globally or in a virtual environment.
```bash
pip install <package>           # Install a package
pip uninstall <package>         # Remove a package
pip list                        # Show installed packages

# Install/upgrade pip itself (system-wide)
python -m pip install --upgrade pip

# Install a package globally (avoid this unless necessary)
pip install --user <package>    # User-level install (no sudo)
sudo pip install <package>      # System-wide install (risky)

# Install a package locally (within a venv)
source my_venv/bin/activate     # Activate venv first
pip install <package>           # Installed only in the venv

pip list --outdated             # Check outdated packages
pip install --upgrade <package> # Update a specific package

# Update all packages (careful!)
pip list --outdated --format=freeze | grep -v '^\-e' | cut -d = -f 1 | xargs -n1 pip install -U

pip freeze > requirements.txt   # Update requirements.txt after manual upgrades
```

Another useful tool is `pipx`, which installs Python CLI tools **globally in isolated environments** to avoid dependency conflicts. So, if you have to install something globally, do it with `pipx`.
	*Each tool is installed in a separated venv, it offers a cleaner global python environment*
```bash
pipx ensurepath              # Add to shell PATH
pipx install black           # Install a CLI tool globally
pipx install black==1.2.3    # Install a specific version of a tool
pipx run cowsay "Hello!"     # Run a tool once without installing
pipx list                    # List installed tools

python -m pip install --user --upgrade pipx # Upgrade pipx itself
pipx upgrade-all             # Update all installed tools
pipx upgrade yt-dlp          # Update a single tool

pipx install yt-dlp          # Install a CLI tool globally (isolated)
pipx run cowsay "Moo"        # Run a tool once without installing

# Inject a dependency into an existing tool’s venv
pipx inject <package> <dependency>  # e.g., pipx inject black httpx
```

|                 | **pip**                | **pipx**               |
| --------------- | ---------------------- | ---------------------- |
| **Purpose**     | Install libraries      | Install CLI tools      |
| **Environment** | Global or project venv | Isolated per-tool venv |
| **Safety**      | Risk of conflicts      | No conflicts           |


***
## OOP
### Class

There is no true encapsulation in python (no visibility concept). So, the convention says that any element (data member, method, function) beginning with an underscore (e.g., `_myvar`) should be *considered* private (so we just "pretend" its private lol). 

Python supports multiplie inheritance (unlike Java), but composition remains better...
We use `isinstance()` for type checks instead of a `type() ==` syntax.

Basic class looks like this:
```python
class Dog:
    species = "Canis familiaris"    # Class attribute (shared between all instances

    def __init__(self, name: str):  # Constructor
        self.name = name            # Instance attribute

    def bark(self) -> None:         # Instance method
        print(f"{self.name} says woof!")


buddy = Dog("Buddy") # Instantiate
buddy.bark()         # 'Buddy says woof!'
```
> [!info]
> There is this convention that `self` (equivalent of `this` in Js or Java) should be the first parameter in instance methods (convention).

Here is how getters & setters are done:
```python
class Temperature:
    def __init__(self, celsius: float):
        self._celsius = celsius       # "Private" variable

    @property
    def celsius(self) -> float:       # Getter
        return self._celsius

    @celsius.setter
    def celsius(self, value: float):  # Setter
        if value < -273.15:
            raise ValueError("Too cold!")
        self._celsius = value
```
> [!info]
> Those `@` things are called decorators

It's possible to have "data classes" which are similar to C structs (they simply bundle together named data items):
```python
from dataclasses import dataclass

@dataclass
class Dog:
    name: str
    race: str
    age: int

oopy = Dog('oopy', 'cool dog', 5)
oopy.age # 5
```

### Inheritance

Composition is smarter, but here's how it looks anyway:
```python
class GoldenRetriever(Dog):  # Inherits from the Dog class we defined above
    def bark(self):
        super().bark()       # Call parent method
        print("...and wags tail!")

goldie = GoldenRetriever("Goldie")
goldie.bark()                # 'Goldie says woof! ...and wags tail!'
```

Python uses **double underscores** (`__`) to **mangle** attribute names and avoid naming collisions in subclasses.
```python
class Mapping:
    def __init__(self, iterable):
        self.items_list = []
        self.__update(iterable)  # Becomes _Mapping__update

    def update(self, iterable):
        for item in iterable:
            self.items_list.append(item)

    __update = update            # Private copy

class MappingSubclass(Mapping):
    def update(self, keys, values):
        # This is safe! Doesn’t override parent’s __update
        for item in zip(keys, values):
            self.items_list.append(item)
```
> [!caution]
> Name mangling is not a security feature which introduces privacy of data, it's mostly here to avoid accidental overrides

Furthermore, Python supports multiple inheritance with a **Method Resolution Order (MRO)**. 
We can use **mixins** for reusable behaviour:
```python
class JSONSerializableMixin:
    def to_json(self) -> str:
        import json
        return json.dumps(self.__dict__)

class User(JSONSerializableMixin):
    def __init__(self, name: str):
        self.name = name

user = User("Alice")
print(user.to_json())  # '{"name": "Alice"}'
```

### Composition Over Inheritance

Composition (using objects as attributes) is cleaner than deep inheritance hierarchies:
```python
class Engine:
    def start(self):
        print("Engine started")

class Wheels:
    def rotate(self):
        print("Wheels rotating")

class Car:
    def __init__(self):
        self.engine = Engine()  # Compose, don’t inherit
        self.wheels = Wheels()

    def drive(self):
        self.engine.start()
        self.wheels.rotate()

tesla = Car()
tesla.drive()  # "Engine started", "Wheels rotating"
```

### ABCs
*Abstract Base Classes*

Enforce method implementation in subclasses:
```python
from abc import ABC, abstractmethod

class Animal(ABC):
    @abstractmethod
    def speak(self) -> str:
        pass

class Dog(Animal):
    def speak(self) -> str:  # Must implement
        return "Woof!"

# animal = Animal()  # Error: Can’t instantiate abstract class
```

### Special methods
*Also called "magic methods", like in PHP*

Implement these for operator overloading and context management:
```python
class Vector:
    def __init__(self, x: float, y: float):
        self.x, self.y = x, y

    def __str__(self) -> str:          # Like .toString in Java
        return f"Vector({self.x}, {self.y})"

    def __repr__(self) -> str:         # Unambiguous representation (debugging)
        return f"Vector({self.x!r}, {self.y!r})"

    def __add__(self, other: "Vector") -> "Vector":
        return Vector(self.x + other.x, self.y + other.y)

    def __eq__(self, other: object) -> bool:
        return isinstance(other, Vector) and self.x == other.x and self.y == other.y

    def __enter__(self):                 # Context manager (for "with" blocks)
        print("Starting vector context")
        return self

    def __exit__(self, *args):
        print("Exiting vector context")
```

### Static & class methods

Like this:
```python
class MyClass:
    @classmethod
    def from_string(cls, s: str) -> "MyClass":  # Alternative constructor
        return cls(s.split(","))

    @staticmethod
    def is_valid(data: str) -> bool:            # Utility (no cls/self access)
        return "valid" in data
```


