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

#####Module System - Migration
- One possible issue in using J9 modules are applications that have direct access to the JRE, internal APIs, internal libraries etc, jar URLs, effectively modules that have been marked as unsafe
  - you can use the `jdeps` tool. it helps in finding if an app has any dependencies on any internal API's
  - access to the $JavaHome/lib directory shouldn't be used
  - `jar:file` is the prefix for jar url that locates files within a jar file - thats now been deprecated. Should now use `jrt` schema `jrt:[module[/path]]`
- 

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
(a, b) -> {...}; 
(Object a, Integer b) -> {...}; 
//new
(var a, var b) -> {...};
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