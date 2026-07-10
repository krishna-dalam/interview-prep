# JVM Architecture

## Interview Priority

**High**

The JVM architecture is important for understanding:

* Java platform independence
* Memory allocation
* Garbage collection
* Class loading
* Application startup
* Runtime performance
* Thread behavior
* Memory leaks
* Production troubleshooting

---

## What Is the JVM?

The Java Virtual Machine, or JVM, is the runtime environment responsible for executing Java bytecode.

Java source code is not executed directly by the operating system.

The simplified execution flow is:

```text
Java source code
       |
       | javac
       v
Java bytecode
       |
       | JVM
       v
Machine-specific instructions
```

A Java source file is compiled into a `.class` file containing bytecode.

The JVM loads, verifies, and executes this bytecode on the target machine.

This provides Java's platform-independent execution model:

```text
Write Java code once
        |
        v
Compile it into bytecode
        |
        v
Run that bytecode on any compatible JVM
```

---

## JDK vs JRE vs JVM

### JVM

The JVM executes Java bytecode.

Its responsibilities include:

* Loading classes
* Verifying bytecode
* Managing runtime memory
* Executing bytecode
* Compiling frequently executed code
* Managing garbage collection
* Supporting threads
* Providing runtime diagnostics

### JRE

The Java Runtime Environment contains the components required to run Java applications.

Conceptually:

```text
JRE = JVM + runtime libraries
```

The JRE does not contain all the tools required to develop Java applications.

### JDK

The Java Development Kit contains the tools required to develop and run Java applications.

Conceptually:

```text
JDK = Runtime environment + development tools
```

Common JDK tools include:

* `javac`
* `java`
* `jcmd`
* `jps`
* `jstat`
* `jstack`
* `jmap`
* `javadoc`

In modern Java distributions, a separate JRE installation may not always be provided. Applications are commonly developed and executed using a JDK or a custom runtime image.

---

# High-Level JVM Architecture

```text
                    Java Source Code
                           |
                           | javac
                           v
                       Bytecode
                           |
                           v
+---------------------------------------------------+
|                       JVM                         |
|                                                   |
|  +---------------------------------------------+  |
|  |          Class Loader Subsystem             |  |
|  |                                             |  |
|  |  Loading -> Linking -> Initialization       |  |
|  +---------------------------------------------+  |
|                           |                       |
|                           v                       |
|  +---------------------------------------------+  |
|  |           Runtime Data Areas                |  |
|  |                                             |  |
|  | Heap                                        |  |
|  | JVM Stacks                                  |  |
|  | Method Area                                 |  |
|  | PC Registers                                |  |
|  | Native Method Stacks                        |  |
|  +---------------------------------------------+  |
|                           |                       |
|                           v                       |
|  +---------------------------------------------+  |
|  |             Execution Engine                |  |
|  |                                             |  |
|  | Interpreter                                 |  |
|  | JIT Compiler                                |  |
|  | Garbage Collector                           |  |
|  +---------------------------------------------+  |
|                           |                       |
|                           v                       |
|  +---------------------------------------------+  |
|  |       JNI and Native Method Libraries       |  |
|  +---------------------------------------------+  |
+---------------------------------------------------+
```

The primary JVM components are:

1. Class-loader subsystem
2. Runtime data areas
3. Execution engine
4. Java Native Interface
5. Native libraries

---

# 1. Class-Loader Subsystem

The class loader loads `.class` files into the JVM when they are required.

The major class-loading stages are:

```text
Loading
   |
   v
Linking
   |
   v
Initialization
```

## Loading

During loading, the JVM:

1. Finds the class bytecode.
2. Reads the `.class` file.
3. Creates the runtime representation of the class.
4. Creates the corresponding `Class` object.

Classes are generally loaded lazily when they are first referenced.

## Linking

Linking consists of three steps:

```text
Verification
Preparation
Resolution
```

### Verification

The JVM verifies that the bytecode is structurally valid and safe to execute.

Examples of verification include:

* Valid bytecode instructions
* Correct class-file structure
* Type safety
* Valid operand-stack usage
* Correct method access

This prevents malformed bytecode from compromising JVM execution.

### Preparation

During preparation, memory is allocated for static fields and default values are assigned.

Example:

```java
public class Account {
    private static int totalAccounts = 100;
}
```

During preparation:

```text
totalAccounts = 0
```

The explicit value `100` is generally assigned later during initialization.

### Resolution

During resolution, symbolic references are converted into direct runtime references.

For example, a symbolic reference to another class or method is resolved to the corresponding runtime structure.

## Initialization

During initialization:

* Static variables receive their explicitly assigned values.
* Static initialization blocks execute.
* Parent classes are initialized before child classes when required.

Example:

```java
public class Application {

    private static int port = 8080;

    static {
        System.out.println("Application class initialized");
    }
}
```

During class initialization:

```text
port becomes 8080
Static block executes
```

---

# Class-Loader Hierarchy

A common class-loader hierarchy is:

```text
Bootstrap Class Loader
          |
          v
Platform Class Loader
          |
          v
Application Class Loader
          |
          v
Custom Class Loader
```

## Bootstrap Class Loader

Loads core Java runtime classes.

Examples include classes from packages such as:

```text
java.lang
java.util
java.io
```

The bootstrap class loader is implemented as part of the JVM runtime and may appear as `null` when inspected from Java.

## Platform Class Loader

Loads platform classes that are not part of the core bootstrap set.

## Application Class Loader

Loads classes from the application's classpath or module path.

This commonly includes:

* Application classes
* Framework dependencies
* Third-party libraries

## Custom Class Loaders

Applications and frameworks can create custom class loaders.

Custom class loaders may be used for:

* Application servers
* Plugin systems
* Hot deployment
* Runtime isolation
* Dynamic module loading

---

# Parent Delegation Model

Class loaders generally follow parent delegation.

```text
Application Class Loader
          |
          | asks parent
          v
Platform Class Loader
          |
          | asks parent
          v
Bootstrap Class Loader
```

The process is generally:

1. Check whether the class is already loaded.
2. Ask the parent class loader to load it.
3. Load the class locally only when the parent cannot find it.

## Why Parent Delegation Matters

Parent delegation:

* Prevents duplicate loading of core classes
* Protects Java runtime classes
* Maintains class-loading consistency
* Reduces the risk of replacing trusted platform classes

For example, an application should not normally be able to replace `java.lang.String` with its own implementation.

---

# 2. Runtime Data Areas

The main JVM runtime data areas are:

```text
Shared across threads:
- Heap
- Method area

Private to each thread:
- JVM stack
- Program-counter register
- Native-method stack
```

---

## Heap

The heap is the runtime memory area where class instances and arrays are allocated.

```text
Heap
 |
 +-- Objects
 +-- Arrays
 +-- Object fields
 +-- Shared application data
```

Example:

```java
Customer customer = new Customer();
```

Conceptually:

```text
customer reference -> thread stack
Customer object    -> heap
```

The heap:

* Is created when the JVM starts
* Is shared by JVM threads
* Is managed by the garbage collector
* Can have a configurable minimum and maximum size
* Can cause `OutOfMemoryError` when insufficient memory remains

Common JVM options include:

```bash
-Xms512m
-Xmx2g
```

Where:

```text
-Xms = initial heap size
-Xmx = maximum heap size
```

### Important Interview Point

Local variables are not automatically objects stored entirely on the stack.

A local reference may be stored in a stack frame, while the object it points to is normally allocated in the heap.

---

## JVM Stack

Every JVM thread has its own JVM stack.

```text
Thread 1 -> JVM Stack 1
Thread 2 -> JVM Stack 2
Thread 3 -> JVM Stack 3
```

A new stack frame is created whenever a method is called.

Each frame may contain:

* Local variables
* Operand stack
* Method return information
* References to runtime constant-pool data
* Intermediate calculation values

Example:

```java
public int add(int first, int second) {
    int result = first + second;
    return result;
}
```

The corresponding stack frame contains conceptual values such as:

```text
first
second
result
operand stack
return information
```

When the method finishes, its frame is removed.

### StackOverflowError

A `StackOverflowError` commonly occurs when a thread requires more stack space than is available.

Example:

```java
public void recurse() {
    recurse();
}
```

This creates stack frames continuously until the thread stack is exhausted.

The stack size may be configured using
