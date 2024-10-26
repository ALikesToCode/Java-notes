# Chapter 3: Object-Oriented Programming Concepts in Java

## 1. Introduction to Object-Oriented Programming (OOP)

In the evolution of programming paradigms, the focus has shifted from procedural approaches to more data-centric models. Traditionally, programming emphasized algorithms and data structures, as highlighted in Niklaus Wirth's book *Algorithms + Data Structures = Programs*. This approach, known as **structured programming**, prioritizes the design of procedures or functions to perform tasks, with data structures crafted to support these algorithms.

### 1.1 Structured Programming

- **Procedures First**: Programs are built by designing specific algorithms for tasks, which are then combined into complex systems.
- **Data Representation Later**: Data structures are selected and constructed to accommodate these predefined procedures.

### 1.2 Object-Oriented Design (OOD)

**Object-Oriented Design (OOD)** reverses the structured programming approach by emphasizing the data (objects) first and the algorithms (methods) second.

- **Focus on Data (Objects)**: Identify the entities that need to be represented and manipulated.
- **Methods Operate on Data**: Design algorithms as methods that work with these objects.

This paradigm shift is particularly beneficial when developing **large systems**. For example, designing a web browser using OOD might involve creating around 100 classes, each encapsulating specific functionality. This contrasts with a procedural approach that might require 2,000 interdependent functions manipulating global data, making maintenance and debugging more challenging.

**Benefits of OOD in Large Systems**:

- **Manageability**: Breaking down complex systems into manageable objects.
- **Ease of Debugging**: Isolating bugs within specific object methods related to an object's state.

### 1.3 OOP Example: Order Processing System

Applying OOD to an **order processing system** involves:

- **Identifying Objects (Nouns)**: Items, Orders, Shipping Addresses, Payments, Accounts.
- **Defining Methods (Verbs)**: addItem, shipOrder, acceptPayment, rejectPayment.

Each object encapsulates data and the methods that operate on it. For example, an **Order** object may have a method to **add an item**, directly mapping actions to the relevant data.

**Contrast with Procedural Approach**:

- **OOD**: Clear separation between data and methods; objects manage their own state and behavior.
- **Procedural**: Functions and data are less distinctly separated, potentially leading to tighter coupling and harder maintenance.

## 2. Designing Objects in OOP

When designing objects in OOP, consider the following key aspects:

- **Behavior**: What methods can operate on the object?
- **State**: What data (instance variables or fields) does the object maintain?
- **Encapsulation**: How to protect the object's state from unauthorized access or modification.
- **Identity**: How to distinguish between different instances of the same class.

### 2.1 Behaviors and States

- **State**: The object's data, represented by its instance variables, which can change over time.
- **Behavior**: The object's methods that define how it can interact with other objects and modify its own state.
- **Encapsulation**: Restricting direct access to an object's state, ensuring that it can only be modified through its methods. This maintains data integrity and hides implementation details.

### 2.2 Relationships Between Classes

Classes interact in several ways:

1. **Dependence (Coupling)**:
   - One class depends on another to function.
   - Example: An **Order** object may depend on an **Account** object to verify credit.
   - **Goal**: Minimize unnecessary dependencies to enhance system robustness and facilitate independent changes.

2. **Aggregation**:
   - One class contains instances of another class.
   - Example: An **Order** contains multiple **Item** objects.

3. **Inheritance**:
   - A subclass inherits properties and behaviors from a superclass.
   - Example: An **ExpressOrder** class inherits from **Order** but adds features like expedited shipping.

## 3. Subclasses and Inheritance

**Inheritance** allows a new class to inherit properties and methods from an existing class, promoting code reuse and establishing hierarchical relationships.

### 3.1 Creating Subclasses

In Java, subclasses are created using the `extends` keyword.

**Superclass Example**:

```java
public class Employee {
    private String name;
    private double salary;

    // Constructor, getters, and setters
    public Employee(String name, double salary) {
        this.name = name;
        this.salary = salary;
    }

    public double bonus(float percentage) {
        return (percentage / 100.0) * salary;
    }
}
```

**Subclass Example**:

```java
public class Manager extends Employee {
    private String secretary;

    public Manager(String name, double salary, String secretary) {
        super(name, salary);
        this.secretary = secretary;
    }

    public void setSecretary(String secretary) {
        this.secretary = secretary;
    }

    public String getSecretary() {
        return secretary;
    }
}
```

- **Inheritance Hierarchy**: `Manager` inherits from `Employee`, gaining access to its methods and instance variables (subject to access modifiers).

### 3.2 Adding Features and Overriding Methods

Subclasses can:

- **Add New Fields**: Introduce new instance variables unique to the subclass.
- **Add New Methods**: Provide additional functionality.
- **Override Methods**: Redefine methods from the superclass to alter or enhance behavior.

**Overriding Example**:

```java
@Override
public double bonus(float percentage) {
    return 1.5 * super.bonus(percentage); // Managers receive a 50% higher bonus
}
```

- **`@Override` Annotation**: Indicates that a method is intended to override a method in the superclass.

### 3.3 Subclass Constructors and `super()`

Constructors in subclasses must ensure that the superclass is properly initialized.

**Using `super()`**:

```java
public Manager(String name, double salary, String secretary) {
    super(name, salary); // Calls the Employee constructor
    this.secretary = secretary;
}
```

- **Purpose of `super()`**: Calls the superclass constructor to initialize inherited fields, especially when they are `private` and inaccessible directly.

### 3.4 Dynamic Dispatch and Polymorphism

**Dynamic Dispatch** allows the program to determine at runtime which method implementation to invoke based on the actual object's class.

**Polymorphism** enables a single reference type to point to objects of multiple types (subtypes).

**Example**:

```java
Employee emp = new Manager("Alice", 90000, "Bob");
```

- **Declared Type**: `Employee`
- **Actual Type**: `Manager`
- **Method Invocation**: Calling `emp.bonus(5)` will execute the `bonus()` method defined in `Manager` due to dynamic dispatch.

### 3.5 Binding Methods: Static vs. Dynamic

- **Static Binding** (Compile-Time Binding):
  - Method calls are resolved based on the reference type.
  - Occurs during compilation.
  - Example: Overloaded methods.

- **Dynamic Binding** (Run-Time Binding):
  - Method calls are resolved based on the object's actual class.
  - Occurs during program execution.
  - Example: Overridden methods.

**Important Note**:

- Methods not defined in the reference type cannot be called, even if the actual object type has them.
- Example: `emp.getSecretary()` will not compile because `getSecretary()` is not defined in `Employee`.

## 4. Overloading and Overriding

### 4.1 Overloading

**Method Overloading** allows multiple methods with the same name but different parameter lists within the same class.

- **Resolved at Compile-Time**: The compiler determines the appropriate method to call based on the method signature (name and parameters).
- **Example**:

```java
public class Printer {
    public void print(String document) { ... }
    public void print(String document, int copies) { ... }
    public void print(String document, String printerName) { ... }
}
```

### 4.2 Overriding

**Method Overriding** allows a subclass to provide a specific implementation of a method already defined in its superclass.

- **Same Signature**: The method in the subclass must have the same name and parameter list.
- **Resolved at Runtime**: The JVM determines which method to invoke based on the object's actual class.
- **Example**:

```java
@Override
public boolean equals(Object obj) {
    // Custom equality logic
}
```

**Summary**:

- **Overloading**:
  - **Different Signatures**
  - **Compile-Time Resolution**

- **Overriding**:
  - **Same Signature**
  - **Run-Time Resolution via Dynamic Dispatch**

## 5. Java Class Hierarchy and Multiple Inheritance

### 5.1 Single Inheritance in Java

- **Single Inheritance Model**: Each class in Java can have only one direct superclass.
- **Avoids the Diamond Problem**: Eliminates ambiguity that arises in multiple inheritance scenarios where a class inherits from two classes that have a common superclass.

**Universal Superclass**:

- **`Object` Class**: The root of the class hierarchy. Every class implicitly extends `Object`.
- **Essential Methods in `Object`**:
  - `equals()`
  - `toString()`
  - `hashCode()`

### 5.2 Using Polymorphism in the Java Class Hierarchy

**Polymorphism** allows methods to operate on objects of various types through a common interface.

**Example**: Searching for an object in an array.

```java
public int find(Object[] array, Object target) {
    for (int i = 0; i < array.length; i++) {
        if (array[i].equals(target)) {
            return i;
        }
    }
    return -1;
}
```

- **Dynamic Method Dispatch**: The `equals()` method invoked is the one overridden in the actual object's class.
- **Benefit**: Code is more flexible and can work with any object type.

### 5.3 Overriding `equals()`

When overriding the `equals()` method:

- **Method Signature Must Match**: `public boolean equals(Object obj)`
- **Implement Type Checking**:

```java
@Override
public boolean equals(Object obj) {
    if (this == obj) return true;
    if (obj == null || getClass() != obj.getClass()) return false;
    MyClass other = (MyClass) obj;
    // Compare fields for equality
}
```

- **Use `instanceof`**: To check if `obj` is an instance of the class.
- **Cast Object**: After confirming type, cast to access specific fields.

## 6. Final and Static Modifiers

### 6.1 `private` and `public`

**Access Modifiers** control the visibility of class members:

- **`private`**:
  - Accessible only within the class.
  - Used to encapsulate and protect the object's state.
- **`public`**:
  - Accessible from any other class.
  - Used for methods that provide the interface to the object's functionality.

**Encapsulation Practices**:

- Declare instance variables as `private`.
- Provide `public` getter and setter methods to access and modify the variables.

### 6.2 Static Modifiers

**Static Members** belong to the class rather than any instance.

- **Static Fields**:
  - Shared among all instances.
  - Example: Counting the number of instances.

```java
public class Employee {
    private static int employeeCount = 0;

    public Employee() {
        employeeCount++;
    }
}
```

- **Static Methods**:
  - Can be called without creating an instance.
  - Cannot access instance variables directly.
  - Example: Utility methods like `Math.max()`.

### 6.3 Final Modifier

The `final` keyword can be applied to variables, methods, and classes.

- **final Variable**:
  - Value cannot be changed once assigned.
  - Used for constants.

```java
public static final double PI = 3.14159;
```

- **final Method**:
  - Cannot be overridden by subclasses.
  - Ensures the method's behavior remains consistent.

- **final Class**:
  - Cannot be subclassed.
  - Used to prevent inheritance.

**Use Cases**:

- **Constants**: Define unchangeable values.
- **Security**: Prevent modification of critical behavior.
- **Design Decision**: Lock down classes or methods for stability.

## 7. Subtyping vs. Inheritance

### 7.1 Differences Between Subtyping and Inheritance

- **Subtyping**:
  - **Interface Compatibility**: Focuses on whether an object can be used in a context that expects another type.
  - **"Is-a" Relationship**: If `B` is a subtype of `A`, a `B` object can be used wherever an `A` object is expected.

- **Inheritance**:
  - **Code Reuse**: Allows a class to inherit implementation (methods and fields) from another class.
  - **Implementation Hierarchy**: Establishes a relationship for sharing code, not necessarily for substitutability.

**Example**:

- **Data Structures**:

  - **Queue**: Supports `insertRear` and `deleteFront`.
  - **Stack**: Supports `insertFront` and `deleteFront`.
  - **Deque**: Supports both sets of operations.

- **Subtyping**:

  - `Deque` is a subtype of both `Queue` and `Stack` because it can perform all their operations.

- **Inheritance**:

  - `Queue` and `Stack` can inherit from `Deque` to reuse code for shared functionality.

### 7.2 Combining Inheritance and Subtyping

In Java, the class hierarchy often serves both to establish subtyping relationships and to facilitate code reuse through inheritance.

**Single Inheritance Limitation**:

- A class can only extend one superclass.
- Cannot inherit implementation from multiple classes.

**Interfaces for Subtyping**:

- **Interfaces** define methods that a class must implement.
- A class can implement multiple interfaces, providing a way to achieve multiple subtyping relationships.

**Example with Interfaces**:

```java
public interface Queue {
    void enqueue(Object item);
    Object dequeue();
}

public interface Stack {
    void push(Object item);
    Object pop();
}

public class Deque implements Queue, Stack {
    // Implement all methods
}
```

- **Benefit**: `Deque` can be used as both a `Queue` and a `Stack`.
- **Inheritance for Code Reuse**: `Deque` can contain common implementation for both interfaces.

**Best Practices**:

- **Use Interfaces**: To define types and establish subtyping relationships.
- **Use Inheritance**: For code reuse when classes share common implementation.
