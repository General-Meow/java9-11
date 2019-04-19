# Java 9-11
The following are notes on the changes from 9 to 11. Not all of them are here, only ones that mnight affect you as a developer

###Java 9
#####Smart compilation
- Just like build tools, smart compilation compiles files that only require compilation
- Also known as Smart Javac
- The tool sjavac
- Made to save time
- Is a smart wrapper around the javac tool

#####Interned Strings in Class Data Sharing (CSD)
- Java 5 to 8, the storage of interned strings was inefficient (both time and storage)
- It was Classes from System jar -> private internal representation -> dump to shared archive
- Strings were encoded as UTF 8 which takes up to 8 bits but most characters only really require 3
- The fix was include 2 new steps after dumping to the shared archive String space allocation -> String table compression
- So Java 9 gives a special area for Strings on the heap then they go into a table thats shared and deduped
- Java 10 improves on this whereby it adds application classes to the shared archive

#####Compact Strings
- Pre Java 9, Strings where internally stored as char arrarys
- chars requires 16 bits per char
- Majority of String objects only require 8 bits (1 byte) as most are Latin 1 charset characters
- Post J9 they are stored as a byte array with a flag to denote encoding

#####VarHandle
- Typed safe way to access class instance variables

#####Diamond operator with anonymous classes
- Diamond operator introduced in J7 to make using generics less verbose
- If you can infer the generic type from the constructor arguments, then you can use the diamond operator on anonymous classes

```java
abstract class MyClass<T> {
  T contents;
  
  abstract void run();
  MyClass(T value) {
    this.contents = value;
  }
}

class blah {
  public static void main(String[] args){
    MyClass a = new MyClass<>(1) {  //the constructor param is an int therefore the generic type is an int
      void run() {
        System.out.println(contents);
      }
    };
    a.run();
  }
}
```

#####Private Interface Methods
- J8 introduced static and public default methods in interfaces
- J9 allows private interface methods now
- So now interfaces support
  - constant variables
  - abstract methods
  - static methods
  - public default methods
  - private methods
  - private static methods

#####Module System
- Packages are containers for classes and interfaces, Modules are containers for packages
- Module names need to be unique and typically use the reverse domain name syntax (like packages)
- e.g. `module com.paulhoang.mymodule {}`
- Modules also need to declare what are its dependencies and what it exports
```java
module com.paulhoang.mymodule {
  requires com.paulhoang.sharedmodule;
  
  exports com.paulhoang.mymodule.api;
  
  opens <Package>;
  
  uses <Type>;
  
  provides <Type>;
}
```
- The Base module is the basic module that exports all the basic packages that the JDK provides
  - This includes packages for io, lang, maths, time, utils, concurrency etc
  - It doesn't require any other module, only exports the fundamental apis required for dev
  - Its not required to explicitly add the base module as a dependency as its always implicitly added
- Module declarations are done in the module-info.java file which lives in the root folder of the module (this file is compiled to a class file)
- The module system allowed for changes to the JDK as well as the JRE
  - previously, the JDK and JRE distributions were very different
  - The JDK included the JRE, native code, the binaries, tools, samples, demos, documentation
  - Now, the 2 are effectively the same, the JRE is a very stripped down version of the JDK
- New step `Link Time` is a new optional step between compile time and runtime
  - Used by jlink to link compiled modules together to make smaller runtime images
  - before jlink, applications would be packaged up with all libraries then another step would be used to remove things
  - e.g. `jlink <options> ---module-path <modulepath> --output <path>`
- There are 3 types of modules:
  - Automatic: when ares are on the module path, the modules are automatically created
  - Explicit/Named: ones that are defined in the module-info.java file, don't have access to types in Unnamed modules
  - Unnamed: when jars aren't defined with a module-info.java (pre J9 jars) and are available on the classpath. These have access to types
             in Named/Auto modules
- There are a few new options to javac and java to support project jigsaw
  - javac now can define a module-source-path that defines where to find the source files for the modules 
  - `javac -d mods --module-source-path . $(find . -name "*.java")`
  - java to define the module-path of the modules your app needs as well as -m or -module <MODULENANE>/<PATH AND CLASS> to define the module and class to run
  - `java --module-path mods -m calculator/com.packt.calculator.Calculator`
- You can use modular jars in pre jigsaw applications as if they are normal libraries (jars). You just have to ensure that when you run the app, that the modular jar is in the classpath
  - during compilation `javac --class-path=<MODULAR DIR> -d classes --source-path xxx`
  - running `java -cp <MODULAR JAR DIR> classnameWithPackage`
- creating a modular jar is done with te jar command.
  - `jar --create --file=<targetDirAndFilename> --module-version <VER> -C <DIR> .`
- running a modular jar
  - `java -p <PATH> -m <JAR>` or `java -d --file=<PATH OF JAR WITH VER.jar>`
  

#####Module System - Migration

- First install the new JDK and compile the project, you might be lucky in that it'll just work. If not then the are some recommended steps
- Upgrade your tooling and project dependencies - so things like the IDE, build tool and pom dependencies
  - If your using a maven based project, you can user the maven plugin `versions-maven-plugin` by codehaus
  - the commands `mvn versions:display-dependency-updates`, `mvn versons:update-property` and `mvn versions:user-latest-snapshots(versions/releases)` will update your pom to use the latest dependency versions
- Compile again and watch for errors and fix
- javac options, source and target. Java is both backwards and forwards compatible. meaning the JRE can run applications compiled for old JREs and the JDK can compile forwards from an older source
  - source specifies the version that the source code must obide by, stating an old version e.g. 1.4 will ensure that the compiler will compile only code that supports java 1.4 features, so if there are autoboxing/generics/lambdas etc it will fail
  - target tells the compile what jvm to target. if it is left out, then the values defaults to the same value as the source option
- Run `jdeps` against the code to issues with the code and apply the suggestions
- `jdeps` runs against your compiled class files, so compile the project first, you can also run it against a jar
  - to create a jar you can use `jar cvfm new-file.jar manifest.mf -C classes .`
  - to run jdeps `jdeps -cp <CLASSOPATH TO FIND THE DEPENDENCIES> <COMPILED CLASSES PATH>` e.g. `jdeps -cp classes:/lib/* classes/mypackage/Dog.class`
  - an equivalent would be `jdeps -verbose:package -cp classes/:lib/* classes/com/packt/Sample.class` the `-verbose:package` configs it to output the dependant packages.
  - another option would be to use `-verbose:class`
  - or the command options `-summary`, `-jdkinternals` for dependencies against the jdk or, `-s` to run against a jar, `-p` to find dependencies for a package
  - check out all the other command line options 
- Run your app by breaking the encapsulation
  - You should do this only if you need to
  - Using the java command, you have the options `--add-opens`, `--add-exports` and `--permit-illegal-access`
  - `--add-opens` allows your code to access non public members. e.g. --add-opens <module>/<package>=<target-module>
  - `--add-exports` allows your code to access internal APIs. The syntax is as above
  - `--permit-illegal-access` allows operations by libraries onto the classpath
- One possible issue in using J9 modules are applications that have direct access to the JRE, internal APIs, internal libraries etc, jar URLs, effectively modules that have been marked as unsafe
  - you can use the `jdeps` tool. it helps in finding if an app has any dependencies on any internal API's
  - access to the $JavaHome/lib directory shouldn't be used
  - `jar:file` is the prefix for jar url that locates files within a jar file - thats now been deprecated. Should now use `jrt` schema `jrt:[module[/path]]`
- There are 2 methods of migrating an application to be modular.
  - Top down - Where you make use of automatic module by making the jvm automatically create modules and have the top most app depend on them, then slowly migrate the dependencies
  - Bottom up - where you migrate all the dependencies and migrate those that depend on them (making them named modules) and any dependencies that arent maintained by you, you make them automatic modules by moving the jars into the module directory without the version number in the name (give the jar a valid java identifier)
  

#####Custom Runtime Images
- There is no major difference between the JDK and JRE in J9 onwards now.
- The only difference is that the platform is now modular and that you can only add what you need for your application
- You do this by using the application `jlink`
- `jlink --module-path mlib:$JAVA_HOME/jmods --add-modules
    calculator,math.util --output image --launcher
    launch=calculator/com.packt.calculator.Calculator`
- The runtime image is placed in the output directory


#####Compiling for other platform versions
- Pre J9 used the `source` and `target` command line options to define the version of the code and the version of the platform the compiled classes for run on
- when skipping these options when compiling, the compiler defaults the to latest version
- If you happen to compile a project with a version that is higher than the version it is executed on, you may get the Unsupported Class Version Error
- The `release` option makes things simpler by allowing you to define what versu=ion you want the classes to run against


#####Multi release Jars
- Jars can now have multiple version of class files in the jar
- Created to allow the support of multiple versions of the JDK
- These Jars will have a META-INF dir and under that a versions dir with the java version dir within that.
  - each of these will have classes that differ for each version

#####Collection API changes
- New static methods have been added to Set, List, Map to make the creatation of them with values easier
```java
Set.of("x","y","z");
List.of("x","y","z");
Map.of("k1", "v1", "k2", "v2");
```

#####Keystores
- Effectively a file that stores `public key certificates` and `private keys`
- typically stored in directory `/jre/lib/security/cacerts`
- it stores an alias for each entry
- password protects each entry
- Pre Java 9 the keystore was Java Keystore (JKS)
- Post J9 it is PKCS12 type which provides stronger cryptographic algorithms
- J9+ still supports JKS

#####Garbage Collection
- Pre J9 the default was the Parallel GC, From J9 onwards its G1 (Generation First)
- There are 3 GCs that will now stop the app from starting
  - DefNew + CMS
  - Incremental + CMS
  - ParNew + SerialOld

#####JShell
- To start type: `jshell`
- To exit: `/exit`
- Usefull commands:
  - `/history`
  - `/list [name or id] | -all | -start`
  - `/methods`
  - `/types`
  - `/vars`
  - `/reset`
- Feedback modes are the levels of verbosity, the available settings are concise (-q), normal (-n), silent (-s) and verbose (-v)
- you set the level by starting jshell with the option or use `/set feedback`
 

###Java 10

#####Local Variable Declarations
- You can now declare variables without the type but with `var` if the type can be inferred
- `var` is actually an identifier and not a keyword, its is technically a reserved type name
- it cannot be used when
  - no initializer is used
  - multiple declarations in a statement
  - multi dimentional arrays
  - ref to an init variable
- `var` can also be used in lamda's, if it is, then all params must use `var`
```java
class Blah {
	public static void main(String[] args){
		 (a, b) -> {...}; 
		 (Object a, Integer b) -> {...}; 
		 //new
		 (var a, var b) -> {...}; 
	}
}
```

#####Alternative memory heap allocation
- Use the java options XX:AllocateHeapAt=<FILE PATH>

#####Removal of JEE and COBRA modules
- In J9 these were marked as deprecated
- J10 removes the modules
  - Aggregator module (java.se.ee)
  - Common Annotations (java.xml.ws.annotation) CORBA (java.corba)
  - JAF (java.activation)
  - JAX-WS (java.xml.ws)
  - JAX-WS tools (jdk.xml.ws)
  - JAXB (java.xml.bind)
  - JAXB tools (jdk.xml.bind)
  - JTA (java.transaction)

###Java 11