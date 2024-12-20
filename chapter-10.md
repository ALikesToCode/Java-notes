## Week 10: Navigating Concurrency: Threads, Processes, Race Conditions, and Synchronization

### Introduction: The Essence of Concurrent Execution

Welcome to Week 10, a pivotal chapter in our journey through programming concepts. Here, we’ll confront the realities of building software that operates in a concurrent world. Our focus shifts to the art and science of concurrent programming, an essential skill in today's dynamic computing environment, where responsiveness and efficiency are paramount. It’s no longer about executing one command after the other; it's about orchestrating multiple tasks that appear to run simultaneously. From the mundane act of browsing the web while a file downloads to a sophisticated simulation of a complex system, the modern world operates concurrently, and so must our programs. 

This week, we will delve into the fundamental concepts of threads and processes, and unravel the intricate challenges of handling shared data through effective synchronization techniques.

### 1. Concurrency: Threads and Processes

#### 1.1 What Does Concurrency Really Mean?
 At its core, concurrency is about managing multiple tasks in a way that gives the impression of parallelism. It's not necessarily about performing all computations at exactly the same instant; rather, it's about making progress on several tasks concurrently, even on a single processor. This illusion is achieved through a clever management of execution units and shared resources. The two main mechanisms that we will look at are:

*   **Multiprocessing:** The operating system (OS) uses *time-slicing* to allocate small time slices of CPU time to different processes. Each process, is a self-contained program with its own private memory space, resources, and a dedicated flow of control. By switching rapidly between processes, an OS provides the illusion of having multiple processes executing at the same time. Although on multi-core machines actual parallel execution is possible, time-slicing is used to manage many processes simultaneously.

*   **Threads:** Threads are execution units within a process that share the process’s memory space, but they maintain their own flow of control. For example, a single program, like a web browser, can have several threads, one thread downloading a file, another rendering the webpage, and yet another handling user inputs.

    *   **Shared and Private Resources:** Processes have private local variables, and their own address space while threads within the same process share resources and communicate by using shared memory.
    *  **Context Switching**: The act of switching between processes and between threads is called context switching. Because threads reside inside a process, their context switches are much cheaper than those for processes. When a process is switched, the operating system has to save and restore large amounts of information including memory space, registers, stack etc. However, in the case of threads that share the same resources, the amount of information which needs to be saved and restored is far smaller.
    *   **Interchangeable Terms:** Though distinct, the terms “process” and “thread” are often used interchangeably in concurrency discussions, especially when focusing on intra-application, parallel execution.
    
#### 1.2 Shared Variables: The Crossroads of Concurrency

Shared variables are the principal vehicle for inter-thread communication within a process. They facilitate the exchange of data but can also lead to a whole range of issues if not handled with extreme care and precision.

Imagine our web browser example again where a download thread runs in parallel to the user interface thread. They share a boolean variable called `terminate`, that determines whether the download should be aborted.

The interaction between the threads on this variable reveals the core challenges of concurrency:
*   **Race Conditions**: if multiple threads can change the shared data, then unpredictable results can occur. This is called a race condition. For example, the browser download thread might test `terminate`, and based on it, it might decide to not stop while the user interface thread has changed the variable to be true, causing the download thread to continue.
*  **Consistent Updates:** When multiple threads can update the shared data, then care must be taken to ensure that the updates are done in the correct way. This often means ensuring that these updates are "atomic," that is, the updates are applied indivisibly, without partial side-effects.

#### 1.3 Creating Threads in Java

Java provides two primary methods for creating and managing concurrent tasks:

1.  **Extending the `Thread` Class:**

     - In this approach, you create a new class that inherits from Java's `java.lang.Thread` class.
    -  Within this new class, you override the `run()` method, which contains the code for your parallel operation.

    ```java
    public class Parallel extends Thread {
       private int id;
       public Parallel(int i) { this.id = i; }
        @Override
        public void run() {
           for (int j = 0; j < 100; j++) {
              System.out.println("Thread ID: " + id);
               try {
                    sleep(1000);  // Sleep for 1 second
                } catch (InterruptedException e) {
                    // Handle interrupted exceptions appropriately
                }
           }
        }
    }
    ```

    To utilize the new thread, instantiate the `Parallel` class and then invoke the `start()` method:
    ```java
      public class TestParallel {
        public static void main(String[] args) {
          Parallel[] p = new Parallel[5];
          for (int i = 0; i < 5; i++) {
              p[i] = new Parallel(i);
             p[i].start(); // Start a concurrent execution of p[i].run()
          }
        }
      }
    ```
    -  Invoking `p[i].run()` directly does not initiate the thread; it behaves like a regular function call. Use `p[i].start()` to run `p[i].run()` concurrently.

2.  **Implementing the `Runnable` Interface:**

    - If the class already inherits another class, which is typically the case with most programming tasks, then you cannot directly use the `Thread` class. Instead, you can implement `java.lang.Runnable` which has the same `run()` method as that of `Thread`.
    - You need to create a thread that uses a object of this class as an argument.

    ```java
    public class Parallel implements Runnable {
       private int id;
        public Parallel(int i) { this.id = i; }
       @Override
       public void run() {
          for (int j = 0; j < 100; j++) {
           System.out.println("Thread ID: " + id);
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
              t[i].start(); // Start the associated thread, triggering run()
           }
        }
      }
    ```

    This pattern allows threads to be associated with objects, while also allowing a class to inherit from a different class, which makes it much more flexible.

When using threads in Java, it is important to note that:
*  When using a thread's sleep() method, it can throw an InterruptedException, which must be handled using try and catch blocks.
*   The method `t.interrupt()` sends a request to interrupt the thread (setting an interrupt flag); this will only be honored if the thread is either waiting or executing a method that checks for the interrupt.
*   The methods `t.interrupted()` and `t.isInterrupted()` are used to check the interrupt flag. The former clears the interrupt flag while the latter does not.

### 2.  The Peril of Race Conditions

As noted earlier, race conditions are a fundamental problem with concurrency. It is not because that two threads running at the same time will produce incorrect results. Instead the potential loss of some updates (such as in the shared variable example from section 1.2) can lead to unpredictable outcomes that defy expectation. Let us go back to the shared variable example and see how we can explicitly describe a race condition.

We wanted a shared variable n, to be incremented twice so that we would expect a result equal to the initial value plus 2. Let us implement each increment as a set of atomic steps:
1. read n into a local variable
2. increment the local variable
3. write the local variable back to n.

If two threads try this approach, this is what may happen:

```
Thread 1                        Thread 2
...                             ...
m = n;                           k = n;
m++;                            k++;
n = m;                           n = k;
...                             ...
```

While intuitively this could imply that *n* increases by 2, in actuality, execution may be interleaved. Here is an example execution flow where n was initialized to 17:
1. Thread 2: reads n and puts the value (17) into variable k.
2. Thread 1: reads n and puts the value (17) into variable m.
3. Thread 1: increments m, and m becomes 18
4. Thread 2: increments k, and k becomes 18
5. Thread 1: writes value of m (18) back to n.
6. Thread 2: writes value of k (18) back to n.

In this scenario, the value of `n` becomes 18 (instead of the expected 19), due to Thread 2 overwriting the changes made by Thread 1. The result of the entire process is dependent on the precise timing at which the threads write back to variable n. The interleaved steps, although correct individually, lead to an incorrect final state of the shared data, a race condition.

### 3. Controlling Access Through Mutual Exclusion

To prevent race conditions, we must regulate concurrent access to shared resources, and force them to be mutually exclusive.

Consider the bank account example where the `transfer()` and `audit()` methods should not be run at the same time.

A naïve implementation may implement `transfer()` and `audit()` as separate methods, but those methods if called at the same time can result in race conditions.

```java
monitor bank_account {
   double accounts[100];

   boolean transfer(double amount, int source, int target) {
       if (accounts[source] < amount) {
        return false;
       }
        accounts[source] -= amount;
        accounts[target] += amount;
        return true;
    }

    double audit() {
        double balance = 0.00;
        for (int i = 0; i < 100; i++) {
             balance += accounts[i];
        }
        return balance;
    }
}
```

Now, if two threads attempt to do the following:
```
Thread 1                                  Thread 2
status = transfer(500.00,7,8);       System.out.print(audit());
```
then the audit may not reflect the total assets correctly.
To solve this, we need a way to ensure that the `transfer()` and `audit()` methods, or more accurately, their critical sections, cannot be executed simultaneously.

#### 3.1 First Attempt: The Flawed "Turn" Variable

We might be tempted to introduce a simple variable called `turn` which acts as a shared variable. The critical sections would be entered when the turn variable has the appropriate value, and it should be set to the opposite value on exiting the critical section.

```java
Thread 1                             Thread 2
...                                  ...
while (turn != 1) {                  while (turn != 2) {
   // "Busy" wait                          // "Busy" wait
}                                    }
// Enter critical section         // Enter critical section
...                                  ...
// Leave critical section           // Leave critical section
turn = 2;                            turn = 1;
...                                  ...
```

 While this method could provide mutual exclusion in some situation, it suffers from several major flaws:

*   **Unspecified Starting Value:**  This approach assumes that the initial value of `turn` is known, and it is valid (i.e., 1 or 2), but makes no provision to ensure this is true.
*   **Atomicity Assumption:** it incorrectly assumes that updates to the turn variable are atomic, which may not be guaranteed.
*   **Starvation:** If the other thread does not cooperate or terminates prematurely, the other thread may wait indefinitely. This is known as a *starvation* problem.

#### 3.2 Second Attempt: The Flawed "Request" Variables

To try to remove starvation, we can use an approach where each thread declares its intention to enter the critical section:

```java
Thread 1                              Thread 2
...                                  ...
request_1 = true;                     request_2 = true;
while (request_2) {                   while (request_1) {
    // "Busy" wait                          // "Busy" wait
}                                      }
// Enter critical section          // Enter critical section
...                                   ...
// Leave critical section            // Leave critical section
request_1 = false;                 request_2 = false;
...                                   ...
```
 This approach fixes the starvation issue but now introduces another problem called **deadlock**, since both threads could get stuck waiting on the other to set its `request` variable to false.

### 4. Peterson’s Algorithm

To address the limitations of both attempts, Peterson’s algorithm provides a combination that guarantees both mutual exclusion and freedom from starvation and deadlock. It uses both a shared request variable for each thread and the shared variable turn to give precedence:

```java
Thread 1                                Thread 2
...                                       ...
request_1 = true;                         request_2 = true;
turn = 2;                                  turn = 1;
while (request_2 && turn != 1) {          while (request_1 && turn != 2) {
  // "Busy" wait                            // "Busy" wait
}                                         }
// Enter critical section               // Enter critical section
...                                       ...
// Leave critical section                // Leave critical section
request_1 = false;                      request_2 = false;
...                                       ...
```
The above ensures that if both threads try to execute the critical section, the `turn` variable will resolve which thread can go through. And if one thread is inactive, then the `request` is stuck to false enabling the other thread to proceed.

However, Peterson's algorithm has its own challenges; for one, generalizing this algorithm to more than two processes is not a trivial task and requires much more sophisticated techniques.

### 5. Semaphores: A Higher-Level Primitive

Instead of relying on shared variables, we can use *semaphores*, integer variables that can only be manipulated through atomic operations:
- P(s): If S > 0, decrement S; else wait until S to become positive.
- V(s): If there are threads waiting for S to become positive, wake one of them up; else increment S.
- These two operations can be used to provide the mutually exclusive access. If a thread is inside the critical section, it would have passed P(s), and another thread trying to enter the section must wait till the V(s) is called by the thread that holds the lock to release its control.

```java
Thread 1                                    Thread 2
...                                  ...
P(S);                                P(S);
// Enter critical section                // Enter critical section
...                                  ...
// Leave critical section            // Leave critical section
V(S);                                 V(S);
...                                  ...
```
While semaphores are a step forward, they still have some drawbacks:
*   **Low level:** There is no specific link between the semaphore and the critical section it protects.
*   **Cooperation Required:** All the thread must cooperate in releasing the semaphore through `V(S)`.
*   **Incomplete Enforcement:** The language cannot ensure that every `P(S)` has a matching `V(S)` and can execute V(S) without P(S), potentially leading to unexpected behavior.

### 6. Monitors: Enforcing Structured Synchronization

Monitors provide a higher-level abstraction for handling mutual exclusion and condition synchronization, and make it easier to reason about concurrency.

#### 6.1 Core Monitor Principles
At its heart, a monitor is similar to a class that has:
*   Data definition, which can only be accessed in a specific manner using
*  Functions that operate on that data.
*   All functions operating within the monitor are implicitly mutually exclusive. That means at most one function can be active at any given time, preventing race conditions.

Example bank_account implementation:
```java
monitor bank_account {
    double accounts[100];

    boolean transfer(double amount, int source, int target){
      if(accounts[source] < amount){
         return false;
      }
      accounts[source] -= amount;
      accounts[target] += amount;
      return true;
    }

    double audit(){
      double balance = 0.00;
       for(int i=0; i<100; i++){
         balance += accounts[i];
      }
      return balance;
    }
}
```
The above implementation, implies that all the methods within the bank_account are mutually exclusive, which is that if one function is in execution, all other functions have to wait.

#### 6.2 External Queue: Waiting for Access
A monitor is associated with an external queue. The threads trying to execute a function within a monitor will be made to wait if another thread is already inside the monitor.

#### 6.3 Condition Variables: More Flexible Waiting
Monitors use condition variables to control a thread's behavior once inside the monitor.  Threads that cannot proceed because of a particular state can call the `wait()` operation to suspend themselves and give up the monitor, to allow other threads to progress. Later the suspended threads can be notified by calling the `notify()` function when some relevant condition occurs.

By using condition variables, monitors can become flexible and can implement complex wait and notify behavior:
- The `wait()` function suspends the thread until another thread signals a change, where the thread releases the monitor during that time.
- The `notify()` function will resume one of the waiting threads. Note that if other threads are blocked in the external queue, these will not be notified as the wait is for a different reason.

```java
monitor bank_account {
    double accounts[100];
    queue q[100]; // one internal queue for each account

    boolean transfer(double amount, int source, int target){
       while(accounts[source] < amount){
        q[source].wait();  // wait in the queue associated with source
      }
       accounts[source] -= amount;
       accounts[target] += amount;
       q[target].notify(); // notify the queue associated with target
       return true;
    }
    double audit() { ... }
}
```

In the above, if the source account has insufficient funds, then the thread will wait until some other transfer has credited funds to the specified source account, in which case the corresponding notify() will wake up the waiting thread.  The use of internal queues with wait/notify avoids starvation, as we will see next.

There are several different ways a notify method can behave:
- **Signal and exit:**  The notifying process immediately exits the monitor, and gives opportunity for a blocked thread to enter the monitor.
-  **Signal and wait:** The notifying process swaps its place with the notified process.
-  **Signal and continue:** The notifying process continues execution and the notified process proceeds once the notifier has finished execution.

Java uses signal and continue model for its monitors. Also, `wait()` method can be interrupted using an `InterruptedException`, and should be handled. Also,  `wait()` and `notify()` should be used only within synchronized methods or blocks.

### 7. Turning an Optional into a Stream

When using Java streams for processing a sequence of values, we may encounter situations where some values in the stream might be absent or null. For this, Java provides a special class called `Optional` which indicates a value may or may not be present. This provides a type-safe mechanism of handling values that may be absent, and also helps avoid Null Pointer Exception, which are notorious for introducing runtime errors.

For example, the method `.max()` when applied to an empty stream, is undefined. So instead of returning a null, it will return an Optional.

```java
 Optional<Double> maxrand =
     Stream.generate(Math::random)
       .limit(100)
       .filter(n -> n < 0.001)
       .max(Double::compareTo);
```

The result is an `Optional<Double>`, which may or may not have a valid double value. To work with an `Optional<T>` object, we need to check if a value is present and to extract it safely.

If a value is present, then you could invoke the `isPresent()` method to find out, however, there are better alternatives:
*   **`orElse(defaultValue)`:** To return a default value if the `Optional` is empty.
*  **`orElseGet(() -> alternative())`:** To lazily generate a default value using a function, if the optional is empty.
*  **`orElseThrow(() -> exception)`:** To throw an exception if the optional is empty.
*  **`ifPresent( (v) -> ...)`:** To apply a method if the `Optional` contains a value.

Some examples:

```java
Double fixrand = maxrand.orElse(-1.0);
Double fixrand = maxrand.orElseGet( () -> SomeFunctionToGenerateDouble());
Double fixrand = maxrand.orElseThrow( () -> IllegalStateException());
optionalValue.ifPresent(v -> Process v);
```

Furthermore, one could transform the value using the map function, or return an alternative `Optional` using the `or` method.
```java
Optional<Double> maxrandsqr = maxrand.map(v -> v*v);
Optional<Double> fixrand =  maxrand.or(() -> Optional.of(-1.0));
```

 If you want to compose functions which return an Optional, for example suppose we have a function to generate the inverse of a value, and a function to get the square root of a value, both of which return an optional, then we must chain it with a `flatMap` operation:

```java
public static Optional<Double> inverse (Double x) {
    if (x == 0) {
        return Optional.empty();
    } else {
        return Optional.of(1 / x);
    }
}

public static Optional<Double> squareRoot (Double x){
    if (x < 0) {
        return Optional.empty();
    }else{
        return Optional.of(Math.sqrt(x));
    }
}

Optional<Double> result = inverse(x).flatMap(MyClass::squareRoot);
```

Finally, using `flatMap`, we can even convert an `Optional<User>` to a `Stream<User>`.

```java
Stream<String> ids = ...;
Stream<User> users = ids.map(Users::lookup)
.flatMap(Optional::stream);
```
And using `ofNullable()`, we can simulate this behaviour when `lookup` is returning a `null` value instead of an optional.

```java
Stream<String> ids = ...;
Stream<User> users = ids.flatMap(
      id -> Stream.ofNullable(Users.oldLookup(id))
  );
```

### 7. Summary
In this chapter we explored how to handle multiple tasks using threads, and how to manage access to shared resources to prevent race conditions using mutual exclusion techniques and semaphores and monitors.

Key takeaways include:
*   **Concurrency** is essential for building responsive and efficient software. 
*   **Threads** operate on a shared memory space, and often this requires synchronisation primitives.
*   **Race conditions** occur when updates to shared data can be interleaved incorrectly, producing errors.
*   **Mutual Exclusion** ensures that only one thread can access a critical section at a time.
*   **Peterson’s algorithm** provides a clever solution to mutual exclusion but does not scale to many threads.
*   **Semaphores** solve many problems of mutual exclusion and starvation, but are still low-level, requiring careful usage.
*   **Monitors** offer a more structured, high-level approach to thread synchronisation using mutual exclusion and condition variables.
*  **Condition variables** using wait() and notify() allow more flexible control within monitors.
*  Java provides `Optional` and `Streams` which allows developers to work with potentially absent values and compose their operations effectively.

Next week, we delve into the specifics of how to construct Graphical Interfaces in Java using Swing and how it uses event driven programming to achieve responsiveness.

**Exercises**
1. Peterson’s Algorithm. Implement Peterson’s algorithm for mutual exclusion, create two threads, and verify that only one thread gets to execute its critical section.

2. Semaphores. Using Java’s Semaphore class, implement the producer-consumer pattern, ensuring that the producer waits for space and the consumer waits for data.

3. Monitors. Implement a simple bank account transfer system using monitors, where several transfer operations can be done to different accounts in parallel.

4. Swing API. Build a simple graphical interface using Swing with a single button, such that when the button is pressed the application prints a message.

**Further Reading:**
*   "Operating System Concepts," by Silberschatz, Galvin, and Gagne, for a discussion on synchronization primitives.
*   The Java documentation on java.lang.Thread, java.lang.Runnable, and java.util.concurrent, for more information about thread-safe collections and related mechanisms.
*   "Concurrent Programming in Java," by Doug Lea, for an in-depth discussion on the subtleties of thread and concurrency control.
