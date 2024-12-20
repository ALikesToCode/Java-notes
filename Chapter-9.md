# Week 9: Mastering Concurrency and Data Handling in Java

## Introduction

Welcome to Week 9, where we delve into the powerful concepts of concurrency and advanced data handling in Java. This module will equip you with the tools to handle multiple threads and efficiently manage data structures, going beyond basic operations to explore Java's core features for building robust, scalable applications.

---

## Module 1: Understanding Optional Types

### 1.1 The Problem: Handling the Absence of Data

In many programs, operations might fail to produce a value. Java introduces *Optional* types to avoid the pitfalls of returning `null`, which can lead to:

- **NullPointerException:** Accessing methods or properties of `null` causes the program to crash.
- **Code Clutter:** Explicit `null` checks add verbosity to the code.

Consider processing user data from an API, where user details like addresses might be missing. Instead of returning `null`, you can use optional types for safer and cleaner code.

### 1.2 The Solution: Introducing `Optional<T>`

Java's `Optional<T>` is a container object that may or may not hold a non-null value. 

**Key Concepts:**

- **Wrapper Object:** Optional is a wrapper for a value of a specific type.
- **State of an Optional:** An `Optional` can be either present or empty.
- **Type-Safe Handling:** Enforces type safety, requiring access through `Optional` methods.

### 1.3 Dealing with Empty Streams

Optionals are commonly used with Streams, particularly with terminal operations like `max()` and `min()`. These methods return an `Optional`, handling cases where a stream might be empty. 

```java
Optional<Double> maxRand = 
    Stream.generate(Math::random)
    .limit(100)
    .filter(n -> n < 0.001)
    .max(Double::compareTo);
```

Here, `maxRand` is an `Optional<Double>`, which may or may not contain a value.

### 1.4 Handling Missing Optional Values

**Common Methods:**

- **`orElse(T other)`:** Returns the value if present, otherwise returns a default.
  
    ```java
    Double fixedRand = maxRand.orElse(-1.0);
    ```

- **`orElseGet(Supplier<? extends T> supplier)`:** Allows complex default value generation when the optional is empty.

    ```java
    Double fixedRand = maxRand.orElseGet(() -> generateDefaultDouble());
    ```

- **`orElseThrow(Supplier<? extends X> exceptionSupplier)`:** Throws an exception if the value is absent.

    ```java
    Double fixedRand = maxRand.orElseThrow(IllegalStateException::new);
    ```

### 1.5 Ignoring Missing Optional Values

- **`ifPresent(Consumer<? super T> action)`:** Executes an action if the value is present.

    ```java
    maxRand.ifPresent(value -> processValue(value));
    ```

- **Adding to a Collection Using `ifPresent`:**

    ```java
    var results = new ArrayList<Double>();
    maxRand.ifPresent(results::add);
    ```

- **`ifPresentOrElse()`:** Handles both presence and absence of a value.

    ```java
    var results = new ArrayList<Double>();
    maxRand.ifPresentOrElse(
        results::add,
        () -> System.out.println("No max")
    );
    ```

### 1.6 Creating Optional Values

- **`Optional.of(T value)`:** Creates an Optional containing a non-null value.
  
    ```java
    Optional<Double> val = Optional.of(2.5);
    ```

- **`Optional.empty()`:** Creates an empty Optional.
  
    ```java
    Optional<String> str = Optional.empty();
    ```

- **`Optional.ofNullable(T value)`:** Creates an Optional that is tolerant of `null` values.

    ```java
    Optional<Double> val = Optional.ofNullable(myFunc());
    ```

### 1.7 Passing On Optional Values

- **`map(Function<? super T, ? extends U> mapper)`:** Applies a mapping function to the value if present, returning an Optional of the result.

    ```java
    Optional<Double> maxRandSquare = maxRand.map(v -> v * v);
    ```

### 1.8 Composing Optional Values

- **`flatMap(Function<? super T, ? extends Optional<? extends U>> mapper)`:** Allows chaining operations that return Optionals, flattening nested Optionals.

    ```java
    Optional<Double> result = inverse(x).flatMap(MyClass::squareRoot);
    ```

### 1.9 Turning an Optional into a Stream

- **`flatMap(Function<? super T, ? extends Stream<? extends U>> mapper)`:** Converts an Optional into a Stream for use in further stream operations.

    ```java
    Stream<User> users = ids.map(Users::lookup)
                            .flatMap(Optional::stream);
    ```

---

## Module 2: Collecting Results from Streams

### 2.1 From Collections to Streams, and Back Again

Converting collections to streams offers flexibility in processing, but often we need to convert streams back into collections.

### 2.2 Processing a Stream as a Collection

- **Using `forEach()`:** Iterates over the stream, applying a function to each element.

    ```java
    mystream.forEach(System.out::println);
    ```

- **Converting to Array:**

    ```java
    Object[] result = mystream.toArray();
    String[] result = mystream.toArray(String[]::new);
    ```

### 2.3 Storing a Stream as a Collection

- **Using `collect()`:**

    - **Create a List:**

        ```java
        List<String> result = mystream.collect(Collectors.toList());
        ```

    - **Create a Set:**

        ```java
        Set<String> result = mystream.collect(Collectors.toSet());
        ```

    - **Create a Concrete Collection:**

        ```java
        TreeSet<String> result = stream.collect(
            Collectors.toCollection(TreeSet::new)
        );
        ```

### 2.4 Stream Summaries

- **Using `summarizingInt()`:** Collects multiple numeric properties of a stream of integers.

    ```java
    IntSummaryStatistics summary = mystream.collect(
        Collectors.summarizingInt(String::length)
    );
    double averageWordLength = summary.getAverage();
    double maxWordLength = summary.getMax();
    ```

### 2.5 Converting a Stream to a Map

- **`toMap(Function keyMapper, Function valueMapper)`:** Transforms a stream into a Map.

    ```java
    Map<Integer, String> idToName = people.collect(
        Collectors.toMap(Person::getId, Person::getName)
    );
    ```

- **Handling Duplicate Keys:**

    ```java
    Map<String, Integer> nameToID = people.collect(
        Collectors.toMap(
            Person::getName,
            Person::getId,
            (existingValue, newValue) -> existingValue
        )
    );
    ```

### 2.6 Grouping and Partitioning Values

- **Grouping by Key:**

    ```java
    Map<String, List<Person>> nameToPersons = people.collect(
        Collectors.groupingBy(Person::getName)
    );
    ```

- **Partitioning Using a Predicate:**

    ```java
    Map<Boolean, List<Person>> aAndOtherPersons = people.collect(
        Collectors.partitioningBy(p -> p.getName().startsWith("A"))
    );
    ```

---

## Module 3: Input/Output Streams

### 3.1 Understanding I/O Streams

I/O streams handle input and output operations, representing data as sequences of bytes.

- **Input Streams:** Read bytes from sources like files and network connections.
- **Output Streams:** Write bytes to destinations like files and network connections.

### 3.2 Input/Output Values and Types

Data in streams can be raw bytes, encoded text, or binary data like integers and floats.

### 3.3 Input and Output Transformations

Transformers process raw bytes, enabling complex data handling by chaining streams and transformers together.

### 3.4 Reading and Writing Raw Bytes

- **`InputStream` and `OutputStream`:** Abstract classes for reading and writing bytes.

    ```java
    InputStream in = ...;
    OutputStream out = ...;
    ```

### 3.5 Important I/O Operations

- **Closing the Stream:** Always close streams with `close()`.
- **Flushing:** Use `flush()` to ensure data is pushed to its destination.

    ```java
    out.flush();
    ```

### 3.6 Connecting a Stream to an External Source

- **File Streams:**

    ```java
    var in = new FileInputStream("input.class");
    var out = new FileOutputStream("output.bin", true); // Append
    ```

### 3.7 Reading and Writing Text

- **Scanner Class:** For parsing text input.

    ```java
    var scin = new Scanner(new FileInputStream("input.txt"));
    String s = scin.nextLine();
    ```

- **PrintWriter Class:** For writing text output.

    ```java
    var pout = new PrintWriter(new FileOutputStream("output.txt"));
    pout.println("Hello, World!");
    ```

### 3.8 Example: Copying Input Text File to Output Text File

```java
var in = new Scanner(new FileInputStream("input.txt"));
var out = new PrintWriter(new FileOutputStream("output.txt"));
while(in.hasNext()) {
    String line = in.nextLine();
    out.println(line);
}
```

### 3.9 Reading and Writing Binary Data

- **Data Streams:**

    ```java
    var din = new DataInputStream(new FileInputStream("input.class"));
    var dout = new DataOutputStream(new FileOutputStream("output.bin"));
    ```

### 3.10 Example: Copying Binary File to Another Binary File

```java
var in = new DataInputStream(new FileInputStream("input.bin"));
var out = new DataOutputStream(new FileOutputStream("output.bin"));
int bytesAvailable = in.available();
while(bytesAvailable > 0) {
    var data = new byte[bytesAvailable];
    in.read(data);
    out.write(data);
    bytesAvailable = in.available();
}
```

### 3.11 Other Useful Features

- **BufferedInputStream:** Wraps streams for efficiency with buffering.

    ```java
    var din = new DataInputStream(new BufferedInputStream(new FileInputStream("grades.dat")));
    ```

- **PushbackInputStream:** Supports speculative reads with `read()` and `unread()`.

    ```java
    var pbin = new PushbackInputStream(new BufferedInputStream(new FileInputStream("grades.dat")));
    ```

### 3.12 Summary of Input/Output Streams

Java's I/O streams allow for efficient data handling by separating concerns and chaining different streams together. Techniques like buffering enhance performance significantly.

---

## Module 4: Serialization

### 4.1 What is Serialization?

Serialization converts an object into a stream of bytes, while deserialization reconstructs it.

### 4.2 Reading and Writing Objects

Java uses `ObjectOutputStream` and `ObjectInputStream` for serialization.

- **Using `ObjectOutputStream`:**

    ```java
    var out = new ObjectOutputStream(new FileOutputStream("employee.dat"));
    var emp = new Employee(...);
    out.writeObject(emp);
    ```

- **Using `ObjectInputStream`:**

    ```java
    var in = new ObjectInputStream(new FileInputStream("employee.dat"));
    var e1 = (Employee) in.readObject();
    ```

- **The `Serializable` Interface:** Classes must implement `Serializable` to enable serialization.

    ```java
    public class Employee implements Serializable { ... }
    ```

### 4.3 How Serialization Works

- **`ObjectOutputStream`:** Saves field contents.
- **`ObjectInputStream`:** Reconstructs the object.

### 4.4 Object Sharing and Serial Numbers

Objects shared among many objects are tracked using serial numbers to ensure only a single copy is archived.

### 4.5 Customizing Serialization

- **`transient` keyword:** Excludes fields from serialization.

    ```java
    private transient int counter;
    ```

- **Overriding `writeObject()` and `readObject()`:**

    ```java
    private void writeObject(ObjectOutputStream out) throws IOException {
        out.defaultWriteObject();
        out.writeDouble(point.getX());
    }

    private void readObject(ObjectInputStream in) throws IOException {
        in.defaultReadObject();
        double x = in.readDouble();
    }
    ```

### 4.6 Handle with Care

- **Version Control:** Serialized objects can become incompatible with newer classes.
- **Security Risk:** Deserialization can execute potentially harmful code.

### 4.7 Summary of Serialization

Serialization allows for object persistence and transfer, but care must be taken with version control and security.

---

## Module 5: Concurrency with Threads and Processes

### 5.1 Understanding Threads and Processes

- **Multiprocessing:** Different computations execute "in parallel" through time-slicing, creating an illusion of parallelism.
- **Processes:** Independent computations with private variables.
- **Threads:** Lightweight processes within a single process, sharing variables and memory.

### 5.2 Shared Variables and Race Conditions

- **Shared Variables:** Efficient for communication but prone to unpredictability and inconsistencies, known as race conditions.

### 5.3 Creating Threads in Java

Java offers two main ways to create and manage threads.

- **Extending the `Thread` Class:**

    ```java
    public class Parallel extends Thread {
        private int id;
        public Parallel(int i) { id = i; }
        public void run() {
            for (int j = 0; j < 100; j++) {
                System.out.println("My id is " + id);
            }
        }
    }
    
    public class TestParallel {
        public static void main(String[] args) {
            Parallel p[] = new Parallel[5];
            for(int i = 0; i < 5; i++) {
                p[i] = new Parallel(i);
                p[i].start();
            }
        }
    }
    ```

- **Implementing the `Runnable` Interface:**

    ```java
    public class Parallel implements Runnable {
        private int id;
        public Parallel(int i) { id = i; }
        public void run() {
            for (int j = 0; j < 100; j++) {
                System.out.println("My id is " + id);
            }
        }
    }

    public class TestParallel {
        public static void main(String[] args) {
            Parallel[] p = new Parallel[5];
            Thread[] t = new Thread[5];
            for (int i = 0; i < 5; i++) {
                p[i] = new Parallel(i);
                t[i] = new Thread(p[i]);
                t[i].start();
            }
        }
    }
    ```

### 5.4 Java Thread States

Java threads can be in one of six states: New, Runnable, Blocked, Waiting, Timed Waiting, and Dead.

- **Determining Thread State:** Use `t.getState()` to query a thread's state.

### 5.5 Interrupts

- **Interrupting a Thread:** A thread can interrupt another using `interrupt()`.
- **Checking Interrupt Status:** Use `interrupted()` or `isInterrupted()` to manage interruptions.

### 5.6 Yield and Join Methods

- **Yield():** Allows a thread to give up running status.
- **Join():** Waits for another thread to terminate.

---

## Module 6: Race Conditions and Mutual Exclusion

### 6.1 Understanding Race Conditions

- **Concurrent Updates:** Multiple threads updating a shared variable concurrently can cause data inconsistencies.

### 6.2 Mutual Exclusion

- **The Goal:** Ensure only one thread accesses a critical section at a time to avoid race conditions.

### 6.3 Initial Approaches (and their Problems)

- **First Attempt (using `turn`):** Leads to starvation if one thread dies.
- **Second Attempt (using `request_1` and `request_2`):** Can lead to deadlock.

### 6.4 Peterson's Algorithm

Combines `turn` and request variables to achieve mutual exclusion without deadlock.

### 6.5 Beyond Two Processes

Generalizing Peterson's algorithm to more than two threads requires complex solutions like Lamport's Bakery Algorithm.

### 6.6 Summary of Module

- **Race Conditions:** Can lead to unpredictable outcomes.
- **Mutual Exclusion:** Ensures only one thread accesses critical sections at a time, reducing race conditions.

---

## Module 7: Test and Set, Semaphores, and Monitors

### 7.1 The Heart of Race Conditions: Test-and-Set

- **Atomic Operation:** Combining check and set operations into an atomic step is necessary for consistent updates.

### 7.2 Semaphores

- **Definition:** A semaphore is an integer variable with atomic test and set operations, providing a mechanism for mutual exclusion.

### 7.3 Problems with Semaphores

- **Low Level:** Lack of enforcement and relationship with critical regions makes semaphores prone to errors.

### 7.4 Monitors

- **Definition:** A high-level synchronization primitive attaching synchronization control to protected data.

### 7.5 Enhancing Monitors

- **Using `wait()` and `notify()`:** Provide a mechanism for threads to suspend and resume based on changing conditions.

### 7.6 Monitors in Java

- **Synchronized Keyword:** Ensures only one thread executes a synchronized method at a time.

### 7.7 Reentrant Locks

- **ReentrantLock:** Allows a thread holding a lock to reacquire it multiple times, managing access to critical sections.

---

## Module 8: Turning Optionals into a Stream and Collecting Stream Results

### 8.1 Turning an Optional into a Stream

- **`flatMap` method:** Converts an optional into a stream for further processing.

    ```java
    Stream<User> users = ids.map(Users::lookup)
                            .flatMap(Optional::stream);
    ```

### 8.2 Collecting Stream Results

Streams are versatile in processing data, but often need conversion back into collections.

- **forEach():** Applies an action to each stream element.

    ```java
    mystream.forEach(System.out::println);
    ```

- **toArray():** Converts a stream into an array.

    ```java
    String[] result = mystream.toArray(String[]::new);
    ```

- **collect(Collectors.toList()):** Converts a stream into a List.

    ```java
    List<String> result = mystream.collect(Collectors.toList());
    ```

### 8.3 Stream Summaries

- **Summarizing Operations:** Extracts summary results from numeric streams.

    ```java
    IntSummaryStatistics summary = mystream.collect(
        Collectors.summarizingInt(String::length)
    );
    ```

### 8.4 Converting a Stream to a Map

- **`toMap()` Function:** Collects a stream into a Map, providing functions for keys and values.

    ```java
    Map<Integer, String> idToName = people.collect(
        Collectors.toMap(Person::getId, Person::getName)
    );
    ```

- **Handling Duplicate Keys:**

    ```java
    Map<String, Integer> nameToID = people.collect(
        Collectors.toMap(
            Person::getName,
            Person::getId,
            (existingValue, newValue) -> existingValue
        )
    );
    ```

### 8.5 Grouping and Partitioning Values

- **Grouping by Key:**

    ```java
    Map<String, List<Person>> nameToPersons = people.collect(
        Collectors.groupingBy(Person::getName)
    );
    ```

- **Partitioning Using a Predicate:**

    ```java
    Map<Boolean, List<Person>> aAndOtherPersons = people.collect(
        Collectors.partitioningBy(p -> p.getName().startsWith("A"))
    );
    ```

---

## Conclusion

Week 9 has provided a deep dive into the essential concepts of concurrency and data handling in Java. By mastering threads, synchronization, and data handling with streams and optionals, you're equipped to build robust, scalable applications in the real world.
