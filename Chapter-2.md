## Week 2: Programming Concepts Using Java

### Lecture 1: **A First Taste of Java**

The journey into programming usually begins with a simple program—the famous "Hello World." This is not just a tradition but helps beginners deal with the fundamental mechanics of compiling, executing, and outputting in a specific language. This lecture deals with the complexity of setting up even a simple Java program, illustrating how Java is syntactically heavier compared to other languages like **Python** or **C**.

#### Python Comparison:

```python
print("hello, world")
```
- Python allows a simple function call `print()` to print directly, making it beginner-friendly.

#### C Comparison:

```c
#include <stdio.h>
main() {
    printf("hello, world\n");
}
```
- C requires including standard libraries (like `stdio.h`) and two levels of effort: function declarations and calling `printf()` for output.

#### Java Example: 

```java
public class HelloWorld {
    public static void main(String[] args) {
        System.out.println("hello, world");
    }
}
```

#### Why so complicated in Java?

Let's break the structure down:
1. **Classes**: In Java, **everything is inside a class**. This means it does not allow free-floating functions like Python.
2. **Public Modifier**: Java introduces visibility modifiers like **`public`** to control access to classes, functions, and variables.
3. **Main Function**: Java uses the `main()` function as the entry point for execution, following a convention in languages like C.
4. **Signature of main()**: 
   - **Return type**: The main function returns **`void`** as it doesn't return a value.
   - **Input arguments**: The main function takes an array of **`String`** as input for command-line arguments.
5. **Static**: The `static` modifier allows a method like **`main()`** to execute without instantiating the class. It indicates that the method belongs to the class itself rather than to any specific object.

#### Printing in Java:
```java
System.out.println("hello, world");
```
- **System.out** is an **output stream** object used to connect to the output screen.
- **println()** prints text and moves to a new line (similar to Python `print()`).

##### Punctuation:
- Java uses braces **`{}`** to define blocks (like functions or classes), and each statement must end with a **semicolon (`;`)**, which contrasts with Python's use of indentation.

#### Compilation and Execution:

1. **JVM** (Java Virtual Machine):
   - Java compiles to a **bytecode** that runs on the JVM.
   - This ensures **portability** across operating systems—slogan: **“Write once, run anywhere.”**

2. **Compilation Command**:
   ```bash
   javac HelloWorld.java
   java HelloWorld
   ```
   - **`javac`** produces a **`.class`** file that contains bytecode.
   - **`java`** runs the bytecode file without needing the **`.class`** extension.

**Reflection**: At this stage, the lecture provides a complete explanation of Java's basic structure and why Java requires elements such as **classes, modifiers (public, static)**, and **main**. These concepts are fundamental because they will anchor all further discussions.

---

### Lecture 2: **Basic Datatypes in Java**

Java is **object-oriented**, but for efficiency and ease of use, eight **scalar types** are treated as primitive types (i.e., non-objects).

#### Primitive Types in Java:
1. **Numerics**:
   - **`int`**: 4 bytes
   - **`long`**: 8 bytes
   - **`short`**: 2 bytes
   - **`byte`**: 1 byte

2. **Floating-point**:
   - **`float`**: 4 bytes
   - **`double`**: 8 bytes

3. **Character**:
   - **`char`**: 2 bytes, optimized for **Unicode** text handling.

4. **Boolean**:
   - **`boolean`**: 1 byte, values **true** and **false**.

#### Variable Declaration and Assignment:
- Variables must be declared before use as shown:
```java
int x, y;
x = 5;
y = 10;
```

- **Strings**:
   ```java
   String s = "Hello";
   ```

#### Key Feature in **Java: Unicode**:
- Java prioritizes support for international characters with **2-byte char** for Unicode.

```java
char c = 'x';
char pi = '\u03C0'; // Greek symbol for pi
```

#### Constants with **final** Modifier:
Java allows constants using `final`:
```java
final float pi = 3.1415927f;
```
Restricting re-assignment:
```java
pi = 3.0; // Error
```

#### Operators & Shortcuts:
1. Standard operators: `+, -, *, /, %`
2. Java lacks a separate division operator `//` for integer division.
3. Exponentiation via **`Math.pow()`**:
   ```java
   Math.pow(2,3); // results in 8.0
   ```

4. Increment/Decrement:
   ```java
   a++; // Same as a = a + 1;
   ```

#### Strings:
Strings in Java are **objects**.
```java
String s = "Hello";
String message = s + " " + "world!";
```
- Strings are **immutable**. Modifications like substring operations generate new string objects instead of modifying in place.

#### Arrays:
Arrays are objects in Java:
```java
int[] a = new int[5];
```
Key features:
- They reference specific memory slots.
- Arrays have a **`.length`** property.
- They are 0-based indexed.

Key reflection here is emphasizing that **primitive types** help Java balance its object-centered paradigm with the need for performance, efficiency, and simplicity in handling numbers and text.

---

### Lecture 3: **Control Flow in Java**

In-depth exploration of control structures in Java, reflecting core programming logic.

#### Conditional Structures:
```java
if (condition) {
    //code
} else {
    //code
}
```
- Braces are necessary for multiple statements but **optional for single statements**.

#### Loops:
##### While Loops:
```java
while (condition) {
    //code
}
```
- **Do-while Loop** checks the condition **after** the loop body, ensuring at least one traversal:
```java
do {
    //code
} while (condition);
```

##### `for` Loops (C-Style Inheritance):
Standard C-style loop:
```java
for (int i = 0; i < n; i++) {
    //code
}
```

##### Enhanced `for` loop:
Introduced to iterate over arrays or collections:
```java
for (int element : array) {
    //code
}
```

#### Multi-way Branching - `switch`:
```java
switch (expression) {
    case value1:
        //code
        break;
    case value2:
        //code
        break;
    default:
        //code
}
```
Java’s `switch` falls through unless **`break`** is explicitly called.

Summarizing, we see how Java’s **control flow** mechanisms are rooted deeply in traditional C-structured examples while incorporating more modern idioms like the enhanced for-loop.

---

### Lecture 4: **Defining Classes and Objects in Java**

At the heart of Java’s object orientation lies the **class** definition, which centralizes the logic for defining new data types.

#### Class Definition and Visibility:
Everything in Java begins with a class:
```java
public class Date {
    private int day, month, year;
}
```
- The `public` keyword makes the class externally visible.
- **Instance variables** (like `day, month, year`) are typically declared **private** to enforce encapsulation.

#### Creating and Initializing Objects:
Objects are instantiated using `new`:
```java
Date d = new Date();
```

#### Methods and `this`:
Methods provide access and manipulation of fields.
```java
public void setDate(int d, int m, int y) {
    this.day = d;
    this.month = m;
    this.year = y;
}
```
- `this` **references the current instance.**

#### Constructors:
Constructors initialize objects at the time of their creation:
```java
public Date(int day, int month, int year) {
    this.day = day;
    this.month = month;
    this.year = year;
}
```
- Java supports **overloading constructors**—multiple constructors based on different parameter signatures:
```java
public Date(int day, int month)
```

#### Copy Constructors:
- This allows creating new instances from existing ones.
```java
public Date(Date aDate);
```

### Reflection:
Java emphasizes **strict encapsulation** and the **importance of controlled data initialization** with constructors and methods like `getDay()`, marking a clear departure from implicit, weakly-typed data handling.

---

### Lecture 5: **Basic Input and Output in Java**

Input and Output are crucial for interacting with programs, and Java provides sophisticated ways to handle both.

#### Handling Input:
##### 1. **Console Class**:
   - Similar to Python's **`input()`**:
```java
Console cons = System.console();
String username = cons.readLine("Username: ");
```

##### 2. **Scanner Class**:
   - Capable of reading different types (integer, string, etc.)
```java
Scanner in = new Scanner(System.in);
String name = in.nextLine();
int age = in.nextInt();
```

#### Handling Output:
1. Normal output via **`System.out.println()`**:
```java
System.out.println("Hello");
```
2. Formatted output is handled similarly to **`printf()`** in C:
```java
System.out.printf("Name: %s, Age: %d", name, age);
```

**Reflection**: Java provides a **layered** and **modular** approach to input/output, whether from basic command-line consoles or complex user streams.

---

### Final Reflection for Week 2:

We’ve integrated the core principles for working in Java, from its heavy reliance on **class structure**, its inherent **object-oriented** approach, control mechanisms (input/output), and handling of **basic data types**. Java shows a clear focus on offering **type safety**, **encapsulation**, and **object manipulation** through methods like constructors and accessors. This rigid structure ensures **portability**, security, and strong typing but adds syntactic weight, which can be reduced through best practices.


