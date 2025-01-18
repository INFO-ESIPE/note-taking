[[Web And Multimedia Technologies]]
16/12/2024
***
**Note:** If you want to grasp the basics of this language, use this website: https://www.learn-php.org/fr/ (With uBlock origin turned on)
***
**Table of Contents**
```table-of-contents
```

****
## Fundamentals

PHP: Object-oriented, interpreted.

A PHP file is an upgraded HTML file which allows to embed PHP-specific instructions to it.
Those instructions are placed in between `<?php` and `?>`.
	*Those instructions will not appear on the final HTML document the client will receive from the webserver, it is purely server-side.
	Each line of PHP ends with a `;`*

In other words, if our PHP file looks like this:
```html
<?php $user = "John"; ?>
<html>
	<head></head>
	<body>
		<h1>Hello <?php echo $user; ?> !</h1>
	</body>
</html>
```

The client will receive:
```html
<html>
	<head></head>
	<body>
		<h1>Hello John !</h1>
	</body>
</html>
```
> [!important]
> This is because PHP acts as a server-side **preprocessor**, meaning the PHP code (`<?php ?>`) will be executed by the server, and only pure HTML will be returned to the client.

### Variables and types

PHP is a **weakly typed** language, which means it's up to PHP to decide a variable type upon assignment. 
	*This means that a variable type can change through time.*
However, modern versions of PHP allow **type enforcement** (explained [[14 - PHP#Forcing typing (out of class scope)|bellow]]). Main scalar types:
- `int`
- `float`
- `string`
- `bool`
- `array`
- Complex types: `NULL`, Object, Callable, Iterable, Resource, 
> The four first types are scalar types.

We declare variables with a `$` (like in PowerShell):
```c
$x = 1;          // Automatically recognized as an integer
$y = 2.5;        // Automatically recognized as a float
$sum = $x + $y;  
echo $sum;       // Outputs: 3.5
```

### Forcing typing (out of class scope)

Since PHP 7, strict and explicit typing have been introduced to reduce ambiguity. This works through several mechanics:
1. **Function typing**: Function arguments and return values can be type-hinted
```php
function multiply(int $a, int $b): int {
    return $a * $b;
}

echo multiply(3, 4); // 12
```

2. **Strict Type Mode**: Strict type checking is enabled at the file level using the `declare(strict_types=1)` directive:
```php
declare(strict_types=1);

function divide(float $a, float $b): float {
    return $a / $b;
}

echo divide(10, 2);    // 5.0
echo divide("10", 2);  // Throws a TypeError
```

3. **Typed Class Properties (PHP 7.4+)**: Class properties can have specific types
```php
class Product {
    public string $name;
    public float $price;
}

$item = new Product();
$item->name = "Laptop";  // OK
$item->price = 999.99;   // OK
```

4. **Union Types (PHP 8.0+)**: Allow parameters or return values to accept multiple types via the `|` symbol
```php
function showValue(int|string $value): void {
    echo $value;
}

showValue(123);        // 123
showValue("Hello");    // Hello
```

5. **Nullable Types**: Allow a type to also accept null using a `?` prefix (similar to Kotlin)
```php
function greet(?string $name): string {
    return $name ? "Hello, $name!" : "Hello, stranger!";
}

echo greet("John"); // Hello, John!
echo greet(null);   // Hello, stranger! (Loud City Song reference)
```

### String Format

Like in Perl, usage of double quotes allows to use variables directly inside the quotes. We can concat strings with `.`:
```c
$name = "Macron";
$money = 2_333_590;
echo "My name is {$name}, I have " . $money . "€ at Rothschild";
```
> [!info]
> The usage of curly braces is only a good practice (clarity), it works without them.

More sophisticated features for multi-lines are Heredoc and Nowdoc:
```php
// Multi-line strings with variable parsing
$text = <<<EOT
My name is $name, 
welcome to the party!
EOT;
echo $text; // My name is Macron, welcome to the party!

// Like the usage of single quotes, it doesn't parse variables
$text = <<<'EOT'
My name is $name, 
welcome to the party!
EOT;
echo $text; // My name is $name, welcome to the party!
```
> [!info]
> This is optional, strings can be concatenated on multiple lines with `.`

### Instruction Flow control and Operators

Same as JavaScript and most languages, except for switch cases and [[14 - PHP#Multidimensional Arrays|native foreach loop]].
PHP Switches doesn't allow for pattern matching. If need this feature, we have to use `match()`:
```php
$i = 3;
switch ($i) {
    case 0:
        echo "i equals 0";
        break;
    case 1:
        echo "i equals 1";
        break;
    case 2:
        echo "i equals 2";
        break;
    default:
       echo "i equals idk lol"; // last case doesn't need a "break"
}

$condition = 5;
try {
    match ($condition) {
        1, 2 => foo(),
        3, 4 => bar(),
        // default => $condition = 'kid'
    };
} catch (UnhandledMatchError $e) {
    var_dump($e);
}

echo $condition; // If I uncomment default, will return "kid"
```

### Arrays

Arrays are heterogeneous, which means we can store entities of various types in an array:
```php
$entities = ["John", 12.5, NULL, []];
print_r($odd_numbers); // Function used to print the content of an array
```
```
Array
(
    [0] => John
    [1] => 12.5
    [2] => 
    [3] => Array
        (
        )
)
```

We can manipulate arrays with index syntax `[index]`, or with built-in functions:
```php
$odd_numbers = [1, 3, 5, 7, 9];
$second_item = $odd_numbers[1];        // 3
$last_item = end($odd_numbers);        // 9
$last_index = count($odd_numbers) - 1; // 4
```
> [!info] 
> As you can see, native PHP functions like `end()` and `count()` are **globally available**, without namespaces or packages (unlike Java). *This is very annoying !!!!*

PHP arrays behaves like `ArrayDeque` in Java, allowing both stack and queue operations:
```php
$numbers = [1, 2, 3];
array_push($numbers, 4);    // [1, 2, 3, 4] (tail insertion, stack-like)
array_pop($numbers);        // [1, 2, 3] (removes and returns last element)

array_unshift($numbers, 0); // [0, 1, 2, 3] (head insertion, queue-like)
array_shift($numbers);      // [1, 2, 3] (removes and returns first element)
```
> [!caution]
> These methods modify the array in place, directly **changing its state**.

You can concatenate, merge, and slice arrays with built-in functions:
```php
$odd_numbers = [1, 3, 5, 7, 9];
$even_numbers = [2, 4, 6, 8, 10];
$all_numbers = array_merge($odd_numbers, $even_numbers); // [1, 3, 5, ..., 8, 10]

$numbers = [4, 2, 3, 1, 5];
sort($numbers); // [1, 2, 3, 4, 5] (sorts array in ascending order)

$numbers = [1, 2, 3, 4, 5, 6];
$sliced = array_slice($numbers, 3);       // [4, 5, 6] (ignores first 3 elements)
$sliced_pad = array_slice($numbers, 3, 2); // [4, 5] (takes 2 elements starting at index 3)
```
> [!caution]
> These functions **return a new array** rather than modifying the original's state.

### Associative Arrays

Its a map, but the syntax is the same as arrays:
```php
$ages = ["Peter" => 32, "Quagmire" => 30, "Joe" => 34];
$ages['Joe']; // 34
```

### Multidimensional Arrays

Like for most languages, arrays can be nested:
```php
$multi = [ 
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9],
];

print_r($multi[0][0]) // 1
print_r($multi[0][1]) // 2
print_r($multi[0][2]) // 3

$persons = [
    "john_doe" => [
        "fname" => "John",
        "name" => "Doe",
        "age" => 25,
    ],
    "jane_doe" => [
        "fname" => "Jane",
        "name" => "Doe",
        "age" => 23,
    ]
];
```

A convenient way to traverse them is via a `foreach` loop:
```php
// Retrieve value only
foreach ($persons as $person) {
    // Inner loop to iterate over the person's details (key-value)
    foreach ($person as $key => $value) { 
	    // ucfirst capitalises the first letter of the given string
        echo ucfirst($key) . ": $value\n";
    }
    echo "\n";
}
```

### Functions

Functions in PHP does not support **overloading** (unlike Java).
	*So, even if two methods have a different signature, they can't share the same name (due to weak typing)*

Function parameters can have default values, be varargs, or even use a **pass the reference** mechanism with `&`:
```php
// if no parameter specified on usage: greet()
// Then "Guest" will be used
function greet($name = "Guest") {
    echo "Hello, $name!";
}

function sum(...$numbers) { // varargs-like
    return array_sum($numbers);
}

function increment(&$value) { // mimmicks pointers
    $value++;
}

// PHP allows you to call a function dynamically through a variable!
$functionName = "sum";
echo $functionName(1, 2, 3);  // 5
```

PHP supports closures, like JavaScript and most modern languages:
```php
$greet = function($name) {
    return "Hello, $name!";
};
echo $greet("John");
```

### Exceptions

Classic `try-catch` blocks are available:
```php
try {
  echo 2 / 0;
} catch (Exception $e) {
  echo "Very stupid thing to do";
}
```
> Like for Java, `finally` can also be included.


***
## Web

### Superglobals

PHP provides **superglobal arrays**, which are pre-defined global variables accessible from anywhere in the file without needing to declare them explicitly:
```php
// $_SERVER: Contains server and request-related metadata
//           (headers, IP addresses...)
echo $_SERVER['HTTP_USER_AGENT']; // Browser information

// $_GET: Contains URL parameters passed via query strings.
// example with http://example.com/page.php?name=John
echo $_GET['name']; // John

// $_POST: Contains data sent via POST requests
if ($_SERVER["REQUEST_METHOD"] === "POST") {
    echo $_POST['username'];
}
```

Two more superglobals exists: 
- `$_SESSION`: Stores user-specific data during a session.
- `$_COOKIE`: Stores small pieces of data in the client’s browser.
Those are covered [[14 - PHP#Sessions and cookies|bellow]].

### Sessions and cookies

**Sessions** allow servers to store data associated with a user across multiple requests. This **data is stored on the server**, and a session ID is passed between the client and server:
```php
session_start();
$_SESSION['username'] = 'JohnDoe';
echo $_SESSION['username']; // JohnDoe
```

**Cookies** are small pieces of data **stored on the client’s browser** and **sent to the server with every request**. Cookies are used for persistent data storage across browser sessions:
```php
setcookie("user", "John", time() + 3600); // Cookie expires in 1 hour
echo $_COOKIE['user']; // John
```

*In short, cookies are client-side while sessions are server-side.*


****
## Database

### Interaction with MySQLi (for class)

MySQL Improved is a PHP extension specifically designed to interact with MySQL (unlike [[14 - PHP#Interaction with PHP Data Objects (out of class scope)|PDOs]] which are more flexible).
	*`mysqli` is a step-up from the deprecated `mysql` extension from PHP 5.5*

```php
// Opening connection
$mysqli = new mysqli('localhost', 'username', 'password', 'database');
if ($mysqli->connect_error) {
    die("Connection failed: " . $mysqli->connect_error);
}


// Execute query (dangerous, vulnerable to SQL Injections)
$result = $mysqli->query("SELECT * FROM users");
while ($row = $result->fetch_assoc()) {
    echo "Name: " . $row['name'] . ", Age: " . $row['age'] . "\n";
}
$result->free(); // Free result set


// Prepared Statements (better)
$stmt = $mysqli->prepare("SELECT * FROM users WHERE email = ?");
$stmt->bind_param("s", $email); // 's' for string
$email = "test@example.com";
$stmt->execute();

$result = $stmt->get_result();
while ($row = $result->fetch_assoc()) {
    echo "Name: " . $row['name'] . "\n";
}

$stmt->close();


// Closing connection
$mysqli->close();


// Demonstration: procedural use
$link = mysqli_connect('localhost', 'username', 'password', 'database');
if (!$link) {
    die("Connection failed: " . mysqli_connect_error());
}

$result = mysqli_query($link, "SELECT * FROM users");
while ($row = mysqli_fetch_assoc($result)) {
    echo "Name: " . $row['name'] . "\n";
}

mysqli_free_result($result);
mysqli_close($link);
```

### Interaction with PHP Data Objects (out of class scope)

Better choice than `mysqli`, a PDO is a database access layer that provides a consistent API regardless of the database system used (MySQL, PostgreSQL...). 
	*It supports secure interactions through prepared statements (mostly to avoid risks of SQL injection).*

```php
// Opening connection
$pdo = new PDO('mysql:host=localhost;dbname=test', 'user', 'password');

// Prepared Statements
$stmt = $pdo->prepare('SELECT * FROM users WHERE email = :email');
$stmt->execute(['email' => 'test@example.com']); // Parameter bind
$result = $stmt->fetchAll();

$stmt = $pdo->prepare('INSERT INTO users (name, age) VALUES (:name, :age)');
$stmt->execute(['name' => 'John', 'age' => 30]);
```

### Data Serialization

Allows to turn data into a storable/transferable format (JSON or XML, most of the time):
```php
// JSON Serialization
$data = ['name' => 'John', 'age' => 30];
$json = json_encode($data);          // Converts to JSON string
$decoded = json_decode($json, true); // Converts back to array

// XML Serialization
$xml = new SimpleXMLElement('<root><name>John</name></root>');
echo $xml->name; // John
```


***
## File Handling
*There are built-in functions allowing to manipulate files.*

### Open and check

We use `fopen()` to open a file:
```php
$file = fopen("example.txt", "r"); // Open in read mode
if (!$file) {
    die("Unable to open file!");
}
fclose($file); // Like in system programming, always close the file after use
```
> Here we opened the file in read-only mode by specifying `r`, but other modes exists:
> - `w`: Write-only (truncates/creates file)
> - `a`: Append (write-only but preserves existing data)
> - `x`: Create a new file (fails if file exists)
> - `+`: for both read and write access (e.g., r+).

Other useful commands:
```php
// Checks if the file exists at the given location
if (file_exists("example.txt")) {
    echo "File exists!";
}

// File metadata
echo filesize("example.txt");  // File size (bytes) 
echo filemtime("example.txt"); // Last modified time

// Delete file
unlink("example.txt");
```

### Read

```php
$file = fopen("example.txt", "r");
$content = fread($file, filesize("example.txt")); // binary-safe file reading
echo $content;
fclose($file);

$lines = file("example.txt"); // read the entire file into an array
foreach ($lines as $line) {
    echo $line;
}
```

### Write

```php
$file = fopen("example.txt", "w"); // binary-safe file writing
fwrite($file, "Hello, World!");
fclose($file);

file_put_contents("example.txt", "New content"); // direct write
```


***
## OOP (out of class scope)
*We assume you know OOP principles, just looking for the syntax and the implementation logic in this language*

### Fundamentals

Here are basic OOP things in PHP: 
```php
class Student {
	// properties can be implicitly declared, but this is very bad...
	private string $first_name;
    private string $last_name;

	// constructor must be declared with this name
    public function __construct(string $first_name, string $last_name) {
        $this->first_name = $first_name;
        $this->last_name = $last_name;
    }

	public function firstName(): string {
        return $this->first_name;
    }

    public function lastName(): string {
        return $this->last_name;
    }

	
    private function fullName() {
        return $this->first_name . " " . $this->last_name;
    }

	public function setFirstName(string $first_name): void {
		 // you can do this check in the constructor too
        if (strlen($first_name) < 2) {
            throw new InvalidArgumentException("At least 2 characters long.");
        }
        $this->first_name = $first_name;
    }

	public function introduce(): string {
        return "My name is " . $this->fullName() . "\n";
    }
}

final class MathStudent extends Student { // inheritence :/
    private static int $math_student_count = 0;
    // "final" keyword not available for variables, we use "const"
    public const MATH_CLUB = "Mathematics Enthusiasts Club";
    
    public function __construct(string $first_name, string $last_name) {
        parent::__construct($first_name, $last_name); // Call parent constructor
        self::$math_student_count++; // Manipulate static property with "self::"
    }

    public static function getMathStudentCount(): int {
        return self::$math_student_count;
    }

    public function sumNumbers(int $first_number, int $second_number): void {
        $sum = $first_number + $second_number;
        echo $this->firstName() . " " . $this->lastName() . " affirms that " . 
	         "{$first_number} + {$second_number} equals {$sum}\n";
    }

    // Overriding parent method
    public function introduce(): string {
        return "I am a Math Student. " . parent::introduce();
    }
}


$alex = new Student("Alex", "Jones");
echo $alex->introduce();

$eric = new MathStudent("Éric", "Chang");
echo $eric->introduce();
$eric->sumNumbers(3, 5);

// Same as java for static but we use "Class::method" instead of "Class.method"
echo "Total Math Students: " . MathStudent::getMathStudentCount() . "\n";
echo "Club Name: " . MathStudent::MATH_CLUB . "\n";

$alex->setFirstName("Alexander");
echo $alex->firstName() . " updated his name.\n";
```

### Interfaces

Like Java:
```php
interface Workable {
    public function work(): void;
}

class Engineer implements Workable {
    public function work(): void {
        echo "Engineering...\n";
    }
}

class Manager implements Workable {
    public function work(): void {
        echo "Managing projects...\n";
    }
}
// ...

function assignWork(Workable $worker) {
    $worker->work();
}

assignWork(new Engineer()); // Engineering...
assignWork(new Manager());  // Managing projects...
```

### Abstract Classes
*Like in Java, better to use interfaces...*

```php
abstract class Person {
	protected string $first_name;
    protected string $last_name;

    public function __construct(string $first_name, string $last_name) {
        $this->first_name = $first_name;
        $this->last_name = $last_name;
    }

    // must be implemented by child classes
    abstract public function getRole(): string;

    public function getFullName(): string {
        return "{$this->first_name} {$this->last_name}";
    }
}

class Teacher extends Person {
    public function getRole(): string {
        return "Teacher";
    }
}

class Student extends Person {
    public function getRole(): string {
        return "Student";
    }
}

$teacher = new Teacher("Alice", "Smith");
$student = new Student("Bob", "Brown");
echo "{$teacher->getFullName()} is a {$teacher->getRole()}\n";
echo "{$student->getFullName()} is a {$student->getRole()}\n";
```

### Late Static Binding

Seems obvious, but it works:
```php
class Base {
    public static function who(): void {
        echo __CLASS__ . "\n";
    }

    public static function callWho(): void {
        static::who(); // Late static binding
    }
}

class Child extends Base {
    public static function who(): void {
        echo __CLASS__ . "\n";
    }
}

// Demonstration
Base::callWho();   // Base
Child::callWho();  // Child
```

### Magic Methods

Magic methods like `__get`, `__set`, or `__call` allows **dynamic handling of properties and methods**:
```php
class MagicClass {
    private array $properties = [];

    public function __get(string $name) {
        return $this->properties[$name] ?? null;
    }

    public function __set(string $name, $value): void {
        $this->properties[$name] = $value;
    }

    public function __call(string $name, array $arguments) {
        echo "Called method '$name' with arguments: " . 
	        implode(", ", $arguments) . "\n";
    }
}

$obj = new MagicClass();
$obj->title = "Dynamic Property";
echo $obj->title . "\n"; // Dynamic Property

$obj->nonexistentMethod("arg1", "arg2"); // Called method 'nonexistentMethod'
										 // with arguments: arg1, arg2
```

### Traits

Allow code reuse across unrelated classes:
```php
trait Logger {
    public function log(string $message): void {
        echo "[LOG]: $message\n";
    }
}

class Developer {
    use Logger; // mind the "use" keyword

    public function writeCode(): void {
        $this->log("Developer is writing code.");
    }
}

class Tester {
    use Logger;

    public function testCode(): void {
        $this->log("Tester is testing code.");
    }
}

$developer = new Developer();
$tester = new Tester();

$developer->writeCode(); // [LOG]: Developer is writing code.
$tester->testCode();     // [LOG]: Tester is testing code.
```

### Enumerations

Enumerations only defines possible values, we cannot include methods:
```php
enum UserRole: string {
    case ADMIN = 'admin';
    case EDITOR = 'editor';
    case VIEWER = 'viewer';
}

function checkAccess(UserRole $role): void {
    match ($role) {
        UserRole::ADMIN => print("Admin has full access.\n"),
        UserRole::EDITOR => print("Editor can edit content.\n"),
        UserRole::VIEWER => print("Viewer can only view content.\n"),
    };
}

$userRole = UserRole::EDITOR;
checkAccess($userRole);  // Editor can edit content.

$userRole2 = UserRole::VIEWER;
checkAccess($userRole2);  // Viewer can only view content.
```


***
## Namespaces (out of class scope)

Namespaces in PHP help organise code into logical groups and prevent name collisions. 
For code to work correctly, one must both **load the files** containing the functions/classes and **resolve the namespaces**.

### Files Loading

The two keywords `require` and `include` are used to **load PHP files into the current script**.
	*Without them the PHP interpreter can't know where to find the classes, functions, or constants you want to use.*

Difference between both keywords:
- `include`: File is **included dynamically** (while interpreted). It throws a warning if the file cannot be loaded but allows the script to continue.
- `require`: File is included **before interpretation** Throws a fatal error if the file cannot be loaded.
	*This is equivalent to a `#include` directive in C*
```php
if($user == "admin") {
	include 'admin_fonctions.php'; // This works as expected, loaded only if
	                               // the user is admin
}
if($user == "admin") {
	require 'admin_fonctions.php'; // This will NOT work as expected.
	                               // This file is loaded no matter what, even
	                               // if the user isn't admin.
}
```
> [!tip]
> So, use `import` for optional/conditional dependencies.

There are variants ensuring we don't load a same file several times:
```php
require_once 'src/Models/User.php';
include_once 'src/Utils/Logger.php';
```
```c
// The logic is the same as what we do in `.h` headers in C:
#ifndef test
#define final
#endif
```

### Namespace Resolution

Namespaces allows to organise code (classes, functions, constants...) into logical groups and prevent name collisions in larger applications.
	*PHP Frameworks have a tendency to give a same name to billions of classes. This namespace features comes out handy in this kind of situation.*
```php
namespace MyApp\Models; // Namespace declared here

class User {
    public function __construct() {
        echo "User class from MyApp\Models namespace";
    }
}
```
> Code in file `src/Models/User.php`

The `use` keyword is used to **resolve namespaces** and make code easier to write by **avoiding the use of Fully Qualified Names (FQNs)**. Let's use our code above in a different file:
```php
require 'src/Models/User.php'; // Load the file containing the class`
use MyApp\Models\User;         // Resolves the namespace for User

$user = new User(); // Uses "User" from "MyApp\Models" namespace
// $user = new \MyApp\Models\User(); // FQN. Using namespace avoids doing this
```

It doesn’t load files—it merely tells PHP which namespace or class to reference in your script.

To avoid conflicts, we can add aliases to our namespaces:
```php
require 'src/Models/User.php';
require 'src/LdapRecord/User.php';

use MyApp\Models\User as MyUser;
use Library\Ldap\User as DependencyUser;

$user = new MyUser(); // Avoids collision with "User" class, as it would force
                      // us to rely on FQN
```

### Autoloading

Having to both load and resolve is kind of annoying. For modern projects, manual `require` statements are replaced by **autoloading**, which dynamically loads files when a class is referenced.
Composer autoloads classes automatically, following PSR-4 standards:
```json
"autoload": {
    "psr-4": {
        "MyApp\\": "src/"
    }
}
```

Now, we just have to require the `autoload` file:
```php
require 'vendor/autoload.php';

use MyApp\Models\User;


$user = new User();
```


***
## Package Managing (out of class scope)

In PHP, the widely-used solution for this is **Composer**, which is a dependency management tool. It helps installing, updating, and autoloading third-party libraries or packages in PHP your project.

We declare everything in the `composer.json` file:
```json
{
    "type": "project",
    "require": {
        "php": ">=8.1",
        "symfony/console": "^6.4",
        "symfony/flex": "^1.0",
        "symfony/framework-bundle": "^6.4"
    },
    "require-dev": {
        "symfony/dotenv": "^6.4"
    },
    "config": {
        "preferred-install": {
            "*": "dist"
        },
        "sort-packages": true
    },
    "autoload": { // What we have seen in the "Autoloading" chapter
        "psr-4": {
            "App\\": "src/"
        }
    }
}
```