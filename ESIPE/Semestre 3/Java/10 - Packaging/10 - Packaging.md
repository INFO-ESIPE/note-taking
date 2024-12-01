[[Java]]
****
**Table of Contents**
```table-of-contents
```

****
## Encapsulation

### Modules and packaging

There are 3 levels of encapsulation in java:
1. **Class**: Contains members (fields, methods, inner classes) with the four levels of visibility we know
2. **Package**: Contains classes in two visibility: `public` and `default`
3. **Module**: Contains packages in two visibility: `default` and `exports`

An **exported** module is described by a `module-info.java` file.
	*Only exported modules are described, that's why*
```java
// module-info.java

module fr.umlv.foo {
	exports fr.umlv.foo;
	exports fr.umlv.foo.api;
}
```
![[export.png]]

The `module-info.java` file should be placed in the directory containing the packages:
```bash
myproject
	src/main/java
		module-info.java                    # module fr.umlv.foo
		/fr
			/umlv
				/foo                        # package fr.umlv.foo
					Main.java
						/api                # package fr.umlv.foo.api
							Fizz.java
							Buzz.java
						/util               # package fr.umlv.foo.util
							Helper.java
```

This matches with what we could observe since the beginning of our journey in java: our classes contains `package` instructions at the top of the file, before the declaration of the class itself:
```java
package fr.umlv.foo.api;          // The package where the file belongs. If the
								  // file is moved to a different directory, it
								  // will not compile.

public class Fizz {/* ... */}
```

Furthermore, each package can contain a descriptor file named `package-info.java`, where we—mostly–indicate the Javadoc (how to handle the API we provide):
```java
/** 
* Javadoc goes here !
*/
package fr.umlv.foo.api;
```
> For the `fr.umlv.foo.api`. Should be placed inside the package (same path as the package name). Our `api` folder now contains three files: `Fizz.java`, `Buzz.java` and `package-info.java` which describe the overall package.

**Should we always bother declaring all of this ?**
	*Well, if you don't plan on working on a real application or library, it does not matter much. 
	If modules aren't declared, all packages will be placed in an **unnamed module**, but **all packages will be exported**.
	If packages aren't declared, all classes will be placed in the **default package**.
	However, ==if a module is declared, it is mandatory to put classes in a package as well.==*


### Jar

The `.jar` format allows to package our `.class` files. Under the hood, it uses the zip format.
**Only one module can be stored per JAR.**

The `jar` command allows us to **store metadata** (which will be placed in the `module-info.java` file for the associated module).
	*Version of the jar, Main class, OS Version for native libraries ...*

Before Java 9, all jar files composing an application were indicated in the `CLASSPATH`, like so:
	`java -classpath lib/fr.umlv.foo-1.0.jar:lib/bar.jar:etc...`, each jar being separated by a `:`. This `CLASSPATH` feature tends to disappear with time.
> it is important to mind the order, as the JVM will search in the jar files in a linear way (like the `$PATH` on Linux).


### Dependence

We declare dependencies required by a module with the `required` keyword):
```java
module fr.umlv.foo {
	exports /*...*/;
	requires java.base; // not required, included by default
	requires java.logging;
	requires fr.umlv.bar;
}
```
![[dependence.png]]
> At runtime, the JVM ensures that all modules are present before going forward

The `ModulePath` contains directories where the jar are.


### Compile

Let's compile our module:
```bash
javac --release 17
	-d output/modules/fr.umlv.foo/
	--module-path other/modules 
	$(find src/main/java/fr.umlv.foo/ -name "*.java")
```

If we want to compile several modules:
```bash
javac --release 17
	--module-source-path src
	-d output/modules/
	--module-path other/modules 
	$(find src/main/java -name "*.java")
```


### Run

We run our project:
```bash
# If the module explicitly specify a Main class
java --module-path mlibs --module fr.umlv.foo

# If we want to specify the Main class ourselves
java --module-path mlibs --module fr.umlv.foo/fr.umlv.foo.Main
```


****
## Security

We can export our package only to specific modules with the `export <package> to <module>` clause:
```java
module fr.umlv.foo {
	requires /*...*/;
	export fr.umlv.foo.util to fr.umlv.bar;
}
```
> The `fr.umlv.bar` module will now have visibility over classes present in the `fr.umlv.foo.util` package. The main purpose of this mechanism is to ==share implementation packages between modules created by the same person.==  

A real world example of this is the `jdk.internal.misc` package, which contains the `Unsafe` class:
```bash
javap --module java.base module-info
```
```java
module java.base {
	exports jdk.internal.misc to
	java.rmi,
	java.sql,
	jdk.charsets,
	// ...
}
```
> Only selected modules of the jdk has visibility over this package. 


We can do [[09 - Reflection#Security|deep reflection]] on **open packages**.
	*Call private methods, change final fields...*
```java
// Open a single package
module fr.umlv.foo {
	// ...
	open fr.umlv.foo.api; // deep reflection allowed
}

// Or directly declare the entire module as open (all its packages will be open)
open module fr.umlv.foo {
	// ...
}
```


****
## `jlink`

If our application is solely composed of modules, we can generate a platform-dependant image containing the JDK and the application.
	*This allows us to not depend on pre-installed JDK version, which is useful for [[02 - OS-Level Virtualization|cloud]]*

`jlink` command will create the image, containing:
- Classes provided by the JDK, and the ones needed by the application
- The VM
- An executable that launches everything

It looks like this:
```bash
# Launch image build
jlink --modulepath /usr/jdk/jdk-9/jmods:mlib
	--add-modules fr.umlv.foo
	--strip-debug
	--output image

# Content of the image
ls -R image
image:
bin conf lib release
image/bin:
fr.umlv.foo java keytool
image/lib:
amd64 classlist jexec modules security tzdb.dat
# ...
image/lib/amd64/server:
classes.jsa libjsig.so libjvm.so Xusage.txt

# Execute
./image/bin/fr.umlv.foo
```

