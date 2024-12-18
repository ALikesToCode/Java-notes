# Chapter 9: Advanced Java Concepts

## Introduction

As developers progress in learning Java, they encounter advanced topics essential for building robust, efficient, and maintainable applications. This chapter consolidates these topics, covering optional types, stream processing, input/output operations, and serialization. By mastering these, developers can leverage Java's full potential to handle data efficiently, perform I/O operations seamlessly, and manage object persistence.

## Optional Types in Java

### Introduction to Optionals

Handling `null` values in Java can be cumbersome and error-prone, leading to `NullPointerException`s. To address this, Java introduced `Optional<T>`, a container that can hold a value or be empty, providing a more elegant way to handle the absence of values.

### Basic Usage

An `Optional` is like a container for a value that might be `null`. Here's how to create and use `Optional`:

```java
Optional<String> optionalName = Optional.of("John Doe");
Optional<String> emptyOptional = Optional.empty();
Optional<String> optionalNullable = Optional.ofNullable(null);
```

### Handling Absent Values

#### Using `orElse()` and `orElseGet()`

These methods return a default value if the `Optional` is empty:

```java
String name = optionalName.orElse("Default Name");
String computedName = optionalNullable.orElseGet(() -> "Computed Name");
```

#### Using `orElseThrow()`

Throws an exception if the `Optional` is empty:

```java
String name = optionalNullable.orElseThrow(IllegalArgumentException::new);
```

#### Using `ifPresent()`

Perform an action if a value is present:

```java
optionalName.ifPresent(System.out::println);
```

You can also handle both cases:

```java
optionalName.ifPresentOrElse(
    System.out::println,
    () -> System.out.println("No value present")
);
```

### Transforming Optionals

Use `map()` and `flatMap()` to transform the contents of an `Optional`:

```java
Optional<Integer> length = optionalName.map(String::length);
Optional<Double> sqrtResult = length.flatMap(this::safeSqrt);
```

### Converting an Optional to a Stream

Convert an `Optional` to a stream of zero or one element:

```java
Stream<String> nameStream = optionalName.stream();
```

## Stream Processing

### Introduction to Streams

Streams provide a way to process sequences of elements, supporting operations like filtering, mapping, and reducing data. Streams are not data structures but views of data, allowing you to perform bulk operations on collections.

### Basic Stream Operations

#### Creating Streams

You can create streams in several ways:

```java
Stream<String> streamOfStrings = Stream.of("one", "two", "three");
Stream<Integer> streamOfNumbers = Arrays.stream(new Integer[]{1, 2, 3});
```

#### Intermediate Operations

Intermediate operations return a new stream:

```java
streamOfStrings
    .filter(s -> s.startsWith("t"))
    .map(String::toUpperCase)
    .forEach(System.out::println);
```

#### Terminal Operations

Terminal operations produce a result or a side effect:

```java
long count = streamOfStrings.count();
List<String> list = streamOfStrings.collect(Collectors.toList());
```

### Collecting Results from Streams

#### Collecting to Collections

Use `collect()` to gather stream elements into a collection:

```java
List<String> list = streamOfStrings.collect(Collectors.toList());
Set<String> set = streamOfStrings.collect(Collectors.toSet());
```

#### Stream Summaries

To obtain statistics from a stream of numbers:

```java
IntSummaryStatistics stats = streamOfNumbers.collect(Collectors.summarizingInt(Integer::intValue));
double avg = stats.getAverage();
```

#### Converting Streams to Maps

Create a map from a stream of objects:

```java
Map<Integer, String> map = peopleStream.collect(
    Collectors.toMap(Person::getId, Person::getName)
);
```

### Advanced Stream Operations

#### Grouping and Partitioning

Group elements by a classifier function or partition them by a predicate:

```java
Map<String, List<Person>> groupedByName = peopleStream.collect(
    Collectors.groupingBy(Person::getName)
);

Map<Boolean, List<Person>> partitionedByAge = peopleStream.collect(
    Collectors.partitioningBy(p -> p.getAge() > 18)
);
```

## Input/Output Streams

### Basic I/O in Java

Java provides a powerful I/O system to handle input and output through streams, supporting operations on files, network connections, and memory buffers.

### Reading and Writing Bytes

#### Classes: `InputStream` and `OutputStream`

`InputStream` and `OutputStream` are abstract classes for reading and writing bytes:

```java
InputStream in = new FileInputStream("input.txt");
OutputStream out = new FileOutputStream("output.txt");
int data = in.read();
out.write(data);
```

#### Methods

- `read()`: Reads a byte.
- `write(int b)`: Writes a byte.
- `close()`: Closes the stream.
- `flush()`: Flushes the stream, ensuring data is written out.

### Reading and Writing Text

#### Using `Scanner` and `PrintWriter`

`Scanner` is used to read text, while `PrintWriter` is used to write text:

```java
Scanner scanner = new Scanner(new FileInputStream("input.txt"));
PrintWriter writer = new PrintWriter(new FileOutputStream("output.txt"));

while (scanner.hasNextLine()) {
    String line = scanner.nextLine();
    writer.println(line);
}
```

### Reading and Writing Binary Data

#### Using `DataInputStream` and `DataOutputStream`

These classes handle primitive data types:

```java
DataOutputStream dataOut = new DataOutputStream(new FileOutputStream("data.bin"));
dataOut.writeInt(123);
dataOut.writeDouble(456.78);

DataInputStream dataIn = new DataInputStream(new FileInputStream("data.bin"));
int number = dataIn.readInt();
double decimal = dataIn.readDouble();
```

### Chaining Streams

Streams can be chained to combine functionalities, such as buffering or reading compressed data:

```java
BufferedInputStream bufferedIn = new BufferedInputStream(new FileInputStream("largefile.txt"));
DataInputStream dataIn = new DataInputStream(bufferedIn);
int number = dataIn.readInt();
```

## Serialization in Java

### Introduction to Serialization

Serialization allows objects to be converted to a byte stream and back, preserving their state for storage or transmission.

### Making a Class Serializable

A class must implement the `Serializable` interface:

```java
public class Person implements Serializable {
    private String name;
    private int age;
    // Constructors, getters, setters
}
```

### Writing and Reading Objects

Use `ObjectOutputStream` and `ObjectInputStream` to serialize and deserialize objects:

```java
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("person.dat"));
out.writeObject(new Person("John", 30));

ObjectInputStream in = new ObjectInputStream(new FileInputStream("person.dat"));
Person person = (Person) in.readObject();
```

### Transient Fields and Custom Serialization

Fields marked `transient` are not serialized:

```java
public class UserSession implements Serializable {
    private String username;
    private transient Socket socket; // Not serialized
}
```

Override `writeObject()` and `readObject()` to customize serialization:

```java
private void writeObject(ObjectOutputStream out) throws IOException {
    out.defaultWriteObject(); // Serialize non-transient fields
    // Additional serialization for transient fields
}

private void readObject(ObjectInputStream in) throws IOException, ClassNotFoundException {
    in.defaultReadObject(); // Deserialize non-transient fields
    // Additional deserialization for transient fields
}
```

### Serial Version UID

Declare `serialVersionUID` for version consistency:

```java
public class Employee implements Serializable {
    private static final long serialVersionUID = 1L;
    // Class contents
}
```

### Security and Risks

- Ensure compatibility of serialized versions by managing `serialVersionUID`.
- Be cautious about deserializing untrusted data due to security risks.

## Conclusion

Advanced Java concepts, including optional types, stream processing, input/output streams, and serialization, empower developers to write efficient, scalable, and maintainable applications. Understanding these topics is crucial for handling data, performing I/O operations, and managing object persistence seamlessly. By mastering these advanced concepts, developers open the door to creating sophisticated Java applications capable of addressing complex programming challenges.
