## Week 10: Mastering Concurrency – Threads, Processes, and Synchronization

### Introduction

Modern software systems seldom perform a single task at a time. Even on a single-processor machine, your web browser can download files while you scroll through webpages, and your computer’s operating system can run multiple applications concurrently. This apparent simultaneity is achieved through techniques like time-slicing, threads, and processes. With multicore processors now commonplace, true parallel execution is possible, but even historically, concurrency was achieved by rapidly switching among tasks, giving the illusion that many programs or activities are progressing at once.

Programming concurrent systems is challenging because of the need to manage multiple threads of execution that share access to data structures. These threads can interleave their actions in unpredictable ways, creating subtle problems such as race conditions, deadlocks, and starvation. In this chapter, we examine the fundamentals of concurrency, the nature of threads and processes, and how to avoid synchronization pitfalls by employing well-designed coordination techniques like mutual exclusion locks, semaphores, monitors, and higher-level language constructs.

### Concurrency Fundamentals

#### Processes vs. Threads

A **process** is a self-contained execution environment with its own state and memory space. On a single-core machine, concurrency at the process level is achieved by the operating system allocating time slices to each process, rapidly switching among them. To the user, it appears as though multiple applications are running simultaneously. On a multicore machine, multiple processes can literally run at the same time, each on a different core.

A **thread** is a lighter-weight mechanism that also supports concurrency but shares the parent process’s address space and data. Multiple threads within the same process can run concurrently, enabling finer-grained parallelism. For example, a program can have one thread handling user interaction, another processing data in the background, and yet another communicating over the network. Because threads share memory, communicating and passing data between them is far simpler than between processes. However, this shared memory also introduces complexity: threads can interfere with each other’s updates to shared variables unless carefully synchronized.

#### Creating Threads in Java

In Java, you can create threads by either extending the `Thread` class or by implementing the `Runnable` interface.

- **Extending `Thread`**:  
  You subclass `Thread` and override its `run()` method. After creating an instance, call the `start()` method, which spawns a new thread and invokes `run()` in that separate thread of control.

  ```java
  public class Parallel extends Thread {
      private int id;
      public Parallel(int i) { id = i; }

      public void run() {
          for (int j = 0; j < 100; j++) {
              System.out.println("My id is " + id);
          }
          try {
              Thread.sleep(1000);
          } catch (InterruptedException e) {
              // Handle interruption
          }
      }
  }
  ```

  ```java
  public class TestParallel {
      public static void main(String[] args) {
          Parallel[] p = new Parallel[5];
          for (int i = 0; i < 5; i++) {
              p[i] = new Parallel(i);
              p[i].start(); // Actually start the thread
          }
      }
  }
  ```

  Calling `start()` is essential. Simply calling `run()` executes it on the current thread, not concurrently.

- **Implementing `Runnable`**:  
  If your class must extend another class or if you prefer a cleaner separation of logic and threading, implement `Runnable` and pass an instance of your class to a new `Thread` object.

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
  ```

  ```java
  public class TestParallel {
      public static void main(String[] args) {
          Parallel[] tasks = new Parallel[5];
          Thread[] threads = new Thread[5];
          for (int i = 0; i < 5; i++) {
              tasks[i] = new Parallel(i);
              threads[i] = new Thread(tasks[i]);
              threads[i].start();
          }
      }
  }
  ```

Threads in Java offer several convenience methods for synchronization and lifecycle management: `Thread.sleep()` suspends execution, `interrupt()` signals interruption, and methods like `isInterrupted()` check interruption status.

### The Challenge of Shared Data

Threads within a process share data and can read and write the same variables. This shared state is a double-edged sword: it simplifies communication but introduces the risk of **race conditions**—unpredictable behavior caused by simultaneous writes or reads-and-writes to shared variables without proper coordination.

#### Race Conditions

A race condition occurs if the result of a computation depends on how threads are scheduled. Consider a shared integer `n` initially set to some value. Two threads intend to increment `n` by one:

```  
Thread A: read n into a local variable m
          increment m
          write m back to n

Thread B: read n into a local variable k
          increment k
          write k back to n
```

If the increments interleave poorly—if both threads read the old value of `n` before either writes back—both might write the same incremented value, effectively losing one increment. The final result then depends on timing, not logic.

### Ensuring Correctness: Mutual Exclusion

To handle shared data safely, threads must take turns when accessing critical sections—pieces of code that modify shared resources. **Mutual exclusion** ensures that only one thread at a time executes such a section.

#### Low-Level Attempts at Mutual Exclusion

Early attempts to ensure mutual exclusion use ordinary shared variables. For instance, you might use a `turn` variable that indicates whose turn it is to enter the critical section, or boolean `request` variables that indicate interest in accessing that section. However, these simplistic solutions often fail when both threads attempt to enter at the same time, either causing deadlock (both waiting forever) or starvation (one thread never gets a turn).

#### Peterson’s Algorithm

Peterson’s Algorithm is a classic two-thread mutual exclusion solution that avoids deadlock and starvation using only shared variables:

```java
// For Thread 1:
request_1 = true;
turn = 2;
while (request_2 && turn != 1) {
    // busy wait
}
// critical section
request_1 = false;

// For Thread 2:
request_2 = true;
turn = 1;
while (request_1 && turn != 2) {
    // busy wait
}
// critical section
request_2 = false;
```

Peterson’s Algorithm ensures that if both threads want to enter, `turn` breaks the tie. It’s elegant and does not assume special hardware instructions, but extending it beyond two threads quickly becomes unwieldy.

### Beyond Two Threads: Higher-Level Primitives

As systems grow more complex, low-level synchronization constructs become too fragile. We need more robust and scalable abstractions.

#### Semaphores

A **semaphore** is an integer counter with two atomic operations, commonly called `P` and `V` (or `acquire()` and `release()`):

- `P(S)`: Wait until the semaphore `S` is positive, then decrement it and proceed.
- `V(S)`: Increment `S` and wake up any waiting threads if necessary.

If you initialize a semaphore to 1, `P(S)` will allow only one thread at a time to proceed, effectively creating a mutual exclusion region. Other threads calling `P(S)` must wait until the first thread calls `V(S)`.

While semaphores are powerful and can solve a broad class of synchronization problems, they’re still relatively low-level and must be carefully managed to avoid errors or forgotten releases.

#### Monitors

A **monitor** is a higher-level abstraction that combines mutual exclusion and condition variables into a single construct, typically supported by the language itself. A monitor is like a class whose methods are automatically executed atomically—only one thread can run a monitor method at a time. This enforces mutual exclusion without requiring you to manually lock and unlock shared data.

In addition, monitors support **condition variables** that allow a thread to wait for a specific condition and automatically release the monitor’s lock. Another thread can then enter the monitor and change the state that the waiting thread depends on. Afterward, it can call `notify()` or `notifyAll()` to wake the waiting threads, allowing them to re-check the condition and proceed.

Monitors thereby combine data encapsulation, mutual exclusion, and synchronization waiting. They reduce complexity and the potential for certain classes of errors.

### Condition Variables and Monitor Patterns

**Condition variables** within monitors use `wait()`, `notify()`, and `notifyAll()` operations:

- **`wait()`**: The calling thread waits and temporarily releases the monitor lock, allowing others to proceed.
- **`notify()`**: Wakes one waiting thread, allowing it to attempt reacquisition of the lock and continue.
- **`notifyAll()`**: Wakes all waiting threads, again requiring them to compete for the lock.

This pattern often appears as:

```java
// Within a monitor method:
while (!condition) {
    conditionVar.wait();
}
// Condition is satisfied; proceed with critical actions
```

When the state changes so that `condition` might now be true, another monitor method calls `conditionVar.notify()` to awaken waiting threads, which re-check the condition before proceeding.

### Handling Absent Values: Optionals and Streams

While concurrency is about running tasks in parallel, modern programming also involves handling uncertain or absent values. In Java, `Optional<T>` provides a safe and expressive way to handle possibly missing values without resorting to `null`.

For example, consider a maximum value computation over a filtered stream:

```java
Optional<Double> maxVal = Stream.generate(Math::random)
    .limit(100)
    .filter(n -> n < 0.001)
    .max(Double::compareTo);
```

`maxVal` may be empty if no values meet the criterion. Using `Optional`, we can avoid `null` checks and handle absence gracefully:

- `orElse(defaultValue)`: returns a default if the optional is empty.
- `orElseGet(supplier)`: lazily generates a default.
- `orElseThrow(exceptionSupplier)`: throws an exception if empty.
- `ifPresent(consumer)`: executes a block if the value is present.
- `ifPresentOrElse(consumer, runnable)`: executes one block if present, another if not.

For functional composition, `map()` transforms the contained value, if any, and `flatMap()` handles transformations that themselves produce Optionals, allowing fluent chaining.

```java
Optional<Double> maxSquared = maxVal.map(v -> v * v);

Optional<Double> result = inverse(x).flatMap(MyClass::squareRoot);
```

This approach avoids `null` checks and makes it easier to compose logic that may or may not yield a result. When integrating concurrency with functional pipelines, optional values mesh well with stream transformations, enabling cleaner, more declarative code.

### Conclusion

The key to building robust concurrent applications is recognizing the complexities of shared state and controlling access through well-designed synchronization constructs. Threads and processes enable concurrency, but without careful attention to mutual exclusion, race conditions and other subtle bugs arise. Lower-level mechanisms like Peterson’s algorithm and semaphores establish the groundwork, but higher-level abstractions like monitors and condition variables offer more structured, safer solutions.

At the same time, modern programming techniques such as the Java `Optional` type simplify error handling and composability, letting developers write clearer code even when dealing with uncertain values. By combining these concepts—concurrency primitives, language-level monitors, and sophisticated data abstractions—developers can create applications that are not only correct and efficient but also maintainable and elegant.
