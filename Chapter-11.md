# Week 11: Concurrency in Java – Monitors, Threads, and Thread-Safe Collections

## Introduction

Modern software often demands efficient, responsive, and scalable execution of multiple tasks simultaneously. From complex server-side applications to interactive desktop software, concurrency has become a crucial aspect of building responsive and high-performance systems. This week, we explore core concepts of concurrent programming in Java, focusing on monitors, threads, and thread-safe collections. We begin with the fundamental notions of synchronization, then delve into the specifics of Java’s concurrency model, culminating in mastering thread-safe data structures and patterns such as producer-consumer.

Our journey proceeds step-by-step, starting with conceptual understanding and building up to practical applications. Throughout, we’ll reinforce theoretical principles with concrete examples, exercises, and scenarios drawn from real-world contexts.

---

## 1. Monitors in Java: Coordinating Concurrent Access

### 1.1. The Concept of Monitors

A monitor is an abstract concept originating in concurrent programming theory, serving as a high-level synchronization construct. It provides a structured mechanism to ensure orderly, controlled access to shared resources. Conceptually, a monitor encapsulates:

- A set of variables (shared data)
- A set of operations (methods) that operate on these variables
- An implicit synchronization mechanism ensuring that only one thread can execute any monitor operation at a time

This guarantees that internal data remains consistent and free from race conditions—situations where multiple threads attempt conflicting updates simultaneously.

**Key Idea:** If one thread is active inside a monitor function, no other thread can execute any of the monitor’s functions until the first thread leaves. This inherent mutual exclusion helps maintain data correctness.

### 1.2. Data Definition and Mutual Exclusion

Monitors combine data structures and the procedures that manipulate them. By design, all methods within a monitor are mutually exclusive:

- **Mutual Exclusion:** At most one thread holds the monitor’s lock at any given moment.
- **Atomic Execution of Methods:** Each method call either runs fully without interruption by another monitor method, or the other caller waits.

This setup prevents interleaved, partial updates. For example, consider a shared bank account:

```java
monitor bank_account {
    double accounts[100];

    boolean transfer(double amount, int source, int target) {
        if (accounts[source] < amount) return false;
        accounts[source] -= amount;
        accounts[target] += amount;
        return true;
    }

    double audit() {
        double balance = 0.0;
        for(int i=0; i<100; i++) balance += accounts[i];
        return balance;
    }
}
```

Within such a monitor, no two threads can simultaneously run `transfer()` or `audit()` on the same instance, ensuring integrity of the `accounts` array.

### 1.3. Internal Queues and Condition Variables

Monitors also provide a mechanism for threads to voluntarily suspend and resume operations when conditions are not favorable. This is where condition variables come in.

- **Condition Variables:** Special queues inside a monitor where a thread can wait until a certain state is reached.
- **Wait and Notify:** A thread waiting for a condition calls `wait()` and suspends itself, releasing the monitor lock. Another thread, upon making a state change that could help the waiting thread proceed, calls `notify()` or `notifyAll()` to wake up the waiting threads.

For example, a bank transfer must wait if the source account has insufficient funds. Once funds arrive, another thread calls `notify()` to wake up the waiting thread.

### 1.4. Monitoring in Java: `synchronized` Methods and Blocks

Java integrates monitor concepts directly into the language:

1. **`synchronized` Methods:**
   Declaring a method as `synchronized` ensures that only one thread at a time can call it on a given object:
   ```java
   public class BankAccount {
       private double[] accounts = new double[100];

       public synchronized boolean transfer(double amount, int source, int target) {
           while (accounts[source] < amount) {
               try { wait(); } catch (InterruptedException e) {}
           }
           accounts[source] -= amount;
           accounts[target] += amount;
           notifyAll();
           return true;
       }

       public synchronized double audit() {
           double balance = 0.0;
           for (double v : accounts) balance += v;
           return balance;
       }
   }
   ```
   
   Here, `transfer()` and `audit()` cannot run simultaneously on the same `BankAccount` object.

2. **`synchronized` Blocks:**
   For finer control, synchronize a specific block:
   ```java
   Object lock = new Object();
   synchronized(lock) {
       // critical section
   }
   ```
   This allows multiple methods to share a common lock object without locking an entire object’s methods.

### 1.5. Wait, Notify, and NotifyAll

Within synchronized methods or blocks, threads can coordinate with `wait()`, `notify()`, and `notifyAll()`:

- **`wait()`**: Thread suspends and releases the lock, waiting in the monitor’s internal queue.
- **`notify()`**: Wakes one waiting thread, but the choice is arbitrary.
- **`notifyAll()`**: Wakes all waiting threads.

Threads typically wait in a loop:
```java
while (conditionNotMet) {
    wait();
}
// Proceed when condition is met
```

This ensures that upon awakening, the thread re-checks the condition since other threads may have also updated the state.

### 1.6. Approaches to Signaling

- **Signal and Exit:** Notifying thread leaves immediately after `notify()`.
- **Signal and Continue (Java’s Approach):** Notifying thread completes its synchronized code, then waiting threads get a chance.
  
This design simplifies reasoning about monitor state.

---

## 2. Threads in Java: Lightweight Units of Execution

### 2.1. What are Threads?

A thread represents a path of execution within a program. Multiple threads can execute simultaneously, providing concurrency. Threads share the process’s memory space but have their own execution stack. This allows:

- Improved responsiveness: Tasks can run concurrently rather than sequentially waiting for each other.
- Enhanced utilization on multicore systems.

### 2.2. Creating Threads

**Two primary approaches:**

1. **Extend `Thread` class**:
   ```java
   class MyThread extends Thread {
       public void run() {
           // code that runs in parallel
       }
   }
   MyThread t = new MyThread();
   t.start();
   ```

2. **Implement `Runnable` interface**:
   ```java
   class MyTask implements Runnable {
       public void run() {
           // code that runs in parallel
       }
   }
   Thread t = new Thread(new MyTask());
   t.start();
   ```

Using `Runnable` is often preferred for flexibility and because Java only supports single inheritance.

### 2.3. Thread Lifecycle

A thread passes through several states:

- **New:** Created but not started.
- **Runnable:** Started and can be scheduled to run by the JVM.
- **Blocked:** Waiting for a lock to enter a synchronized block.
- **Waiting:** Suspended via `wait()` or waiting for `join()` or `notify()`.
- **Timed Waiting:** Suspended for a specified time, e.g., `sleep(1000)`.
- **Terminated (Dead):** Run method completed.

The scheduler decides which runnable threads to run. Threads may yield control with `Thread.yield()`.

### 2.4. InterruptedException and Thread Interruption

Threads can be interrupted to signal that they should stop or change what they are doing:

- **`interrupt()`:** Sets an interrupt flag on the target thread. If the thread is sleeping or waiting, it throws `InterruptedException`.
- **`interrupted()` or `isInterrupted()`:** Checks the interrupt status within the thread’s run method. `interrupted()` clears the flag, `isInterrupted()` does not.

By carefully handling interruptions, threads can be gracefully shut down or asked to stop early.

### 2.5. Joining Threads and Yielding Control

- **`join()`:** One thread waits for another thread to finish. Useful for synchronization after parallel work is done.
- **`sleep(millis)`:** Makes the current thread pause execution for a given time, releasing CPU but not locks.
- **`yield()`:** Suggests to the scheduler that the current thread is willing to let others run.

---

## 3. Concurrent Programming: Ensuring Safe and Correct Interaction

### 3.1. Shared Data and Race Conditions

When multiple threads read or write shared variables without proper coordination, race conditions can arise. The outcome depends on execution order, leading to unpredictable results.

**Example:** Two threads incrementing a shared counter can cause missed increments if not synchronized, because read-modify-write sequences can interleave incorrectly.

### 3.2. Atomicity, Ordering, and Visibility

**Atomic Operations:** Must execute fully or not at all. Using synchronized blocks or atomic variables ensures safe increments and updates.

**Ordering and Visibility:** Without synchronization, changes made by one thread may not be visible to others in a timely manner. Synchronization ensures a consistent view of memory.

### 3.3. Locks and Critical Sections

- **Critical Sections:** Regions of code accessing shared data that must not run concurrently.
- **Locks** ensure that only one thread at a time enters these critical sections.

**Java’s `ReentrantLock`** provides finer control than `synchronized`:
```java
Lock lock = new ReentrantLock();
lock.lock();
try {
    // critical section
} finally {
    lock.unlock();
}
```
Always release locks in a `finally` block to avoid deadlocks if exceptions occur.

### 3.4. Deadlocks, Starvation, and Fairness

- **Deadlock:** Two threads wait forever for each other’s locks.
- **Starvation:** A thread never gets access to resources because others repeatedly jump ahead.
- **Fairness:** Locks and conditions can be configured to ensure that no thread is indefinitely denied access.

Proper locking strategy and structured design can mitigate these issues.

---

## 4. Thread-Safe Collections in Java

### 4.1. Why Thread-Safe Collections?

Using non-thread-safe collections like `ArrayList` or `HashMap` can cause race conditions under concurrent updates. Thread-safe collections automate the internal synchronization needed to keep data consistent.

### 4.2. Built-In Thread-Safe Collections

Java’s `java.util.concurrent` package provides several thread-safe and scalable collections:

- **ConcurrentHashMap:** Allows concurrent reads and updates across different map segments, improving scalability and reducing contention.
- **CopyOnWriteArrayList:** Suitable for scenarios with many reads and few writes. On updates, it copies the entire underlying array, ensuring that readers see a consistent snapshot.
- **BlockingQueue (e.g., ArrayBlockingQueue, LinkedBlockingQueue):** A queue that automatically blocks threads attempting to insert into a full queue or remove from an empty one.

### 4.3. Atomic Variables and Lock-Free Programming

For single variables, Java provides `java.util.concurrent.atomic` classes:

- **`AtomicInteger`, `AtomicLong`, `AtomicReference`**
  
They perform read-modify-write operations atomically without the overhead of synchronization. This can be crucial for performance-sensitive sections.

### 4.4. Producer-Consumer with Blocking Queues

The producer-consumer pattern is a fundamental concurrency pattern. Producers generate data and put it into a shared buffer, while consumers retrieve data from it.

Using a `BlockingQueue`:

- If a producer tries to add to a full queue, it waits (blocks).
- If a consumer tries to remove from an empty queue, it waits.
- This automatic blocking removes the need for explicit synchronization, making the code simpler and safer.

**Example:**

```java
BlockingQueue<String> queue = new LinkedBlockingQueue<>(10);

class Producer implements Runnable {
    public void run() {
        try {
            while (true) {
                String item = produceItem();
                queue.put(item); // blocks if queue is full
            }
        } catch (InterruptedException e) {
            // handle interruption
        }
    }
}

class Consumer implements Runnable {
    public void run() {
        try {
            while (true) {
                String item = queue.take(); // blocks if empty
                consumeItem(item);
            }
        } catch (InterruptedException e) {
            // handle interruption
        }
    }
}
```

### 4.5. Choosing the Right Data Structure

Selecting the right thread-safe collection depends on your use case:

- Frequent reads and rare writes: `CopyOnWriteArrayList`.
- High concurrency with key-based updates: `ConcurrentHashMap`.
- Need for ordered traversal with concurrency: `ConcurrentSkipListMap`.
- Producer-consumer: `BlockingQueue`.

---

## Real-World Applications and Case Studies

**Servers and Web Applications:** Web servers often maintain shared caches (using `ConcurrentHashMap`) accessed by multiple request-handling threads. Thread-safety ensures consistent caching behavior under high load.

**GUI Applications:** Background threads may fetch data while the UI thread displays it. `Swing` or `JavaFX` platforms often use locks or thread-safe queues to safely pass data between the background worker and the UI.

**High-Frequency Trading Systems:** Require low-latency concurrency control. Atomic variables or lock-free structures can provide performance benefits where strict ordering and low overhead are critical.

**Historical Example:** Early concurrent systems before high-level concurrency frameworks were difficult to implement without race conditions. Monitors, introduced by C.A.R. Hoare, laid the foundation for safer concurrency. Java’s concurrency utilities build upon these early concepts, providing robust tools and patterns.

---

## Exercises

1. **Simple Banking System:**
   - Implement `BankAccount` with `deposit()`, `withdraw()` and `getBalance()` methods synchronized.
   - Create multiple threads performing deposits and withdrawals concurrently.
   - Verify data integrity using `audit()`.

2. **Producer-Consumer using BlockingQueue:**
   - Implement a producer that generates random integers and a consumer that processes them.
   - Run multiple producers and consumers and confirm that the queue never corrupts data and that consumers block when empty.

3. **Interrupt Handling:**
   - Write a thread that sleeps periodically.
   - Interrupt it from another thread and ensure it gracefully handles the interruption and stops.

4. **Concurrent Updates with ConcurrentHashMap:**
   - Use a `ConcurrentHashMap<String, Integer>` to maintain a word count for incoming text lines from multiple threads.
   - Safely aggregate counts without explicit locking.

5. **Deadlock Simulation:**
   - Intentionally create a deadlock scenario by having two threads each trying to lock two resources in different order.
   - Detect and fix it by imposing a strict lock ordering.

---

## Conclusion

Concurrency in Java can be complex but becomes manageable with the right tools and practices. By understanding monitors, synchronized methods, condition variables, threads, locks, and thread-safe collections, programmers can create systems that are both responsive and correct, avoiding pitfalls like race conditions or deadlocks.

This week’s exploration—from fundamental concepts like mutual exclusion and waiting conditions to advanced constructs like `ConcurrentHashMap` and blocking queues—provides a strong foundation for designing, building, and maintaining concurrent Java applications. With careful synchronization, the use of appropriate data structures, and proper handling of threads and conditions, concurrency transforms from a challenge into a powerful tool in the Java programmer’s arsenal.

No matter the domain—finance, web services, real-time control systems—the principles covered here serve as a guiding framework for mastering concurrency in Java.
