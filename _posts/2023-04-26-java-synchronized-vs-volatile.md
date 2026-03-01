---
title: "Java Synchronized vs Volatile"
categories:
  - Concurrency
tags:
  - Synchronized
  - Volatile
  - Multithreading
  - Java
  - Thread Safety
---

Picture this: you have a counter shared between two threads, and both of them are incrementing it. You run the program expecting 2,000,000 as the final count, but instead you get something like 1,387,422. You add the `volatile` keyword, run it again, and the number is still wrong. You switch to `synchronized`, and suddenly the answer is correct, but now your throughput drops noticeably under load. These are not obscure edge cases. They are the direct consequence of two distinct problems in concurrent programming: **execution control** and **memory visibility**. Choosing the wrong tool for the wrong problem is one of the most common concurrency mistakes I see in production code.



### Execution Control and Memory Visibility

Thread safety has two primary aspects:

* **Execution Control**: ensuring that operations execute in the correct order and that shared resources are not corrupted by concurrent access.
* **Memory Visibility**: ensuring that changes made by one thread are actually seen by other threads.

Understanding the difference between these two aspects is crucial for designing safe and efficient multithreaded applications. `synchronized` addresses both. `volatile` addresses only memory visibility.



### Execution Control

Execution control is concerned with determining when code is executed (including the order of instruction execution) and whether it can execute concurrently. Proper execution control is essential to prevent race conditions and ensure that shared resources are accessed correctly.

**Example: Bank Account**

Consider a simple bank account class with a balance variable and methods to deposit and withdraw money:

```java
public class BankAccount {
    private int balance;

    public void deposit(int amount) {
        balance += amount;
    }

    public void withdraw(int amount) {
        if (balance >= amount) {
            balance -= amount;
        } else {
            System.out.println("Insufficient balance.");
        }
    }
}
```
In a multithreaded environment, two threads might attempt to withdraw money at the same time. If both threads execute the check **balance >= amount** before either has a chance to update the balance, they may both proceed with the withdrawal, potentially causing the account balance to become negative. This is an issue with execution control since the order of operation execution is not properly managed.

To fix this issue, we can use synchronization:

```java
public class BankAccount {
    private int balance;

    public synchronized void deposit(int amount) {
        balance += amount;
    }

    public synchronized void withdraw(int amount) {
        if (balance >= amount) {
            balance -= amount;
        } else {
            System.out.println("Insufficient balance.");
        }
    }
}
```
Now, only one thread can execute the deposit or withdraw method at a time, ensuring proper execution control.



### Memory Visibility

Memory visibility deals with the timing of when the effects of what has been done in memory are visible to other threads. Proper memory visibility is essential to prevent threads from working with stale or inconsistent data.

Each thread may keep its own cached copy of a variable, and without explicit memory ordering guarantees, there is no requirement that a write by one thread ever becomes visible to another. The following diagram illustrates this:

```
┌─────────────────────┐     ┌─────────────────────┐
│      Thread 1       │     │      Thread 2       │
│  ┌───────────────┐  │     │  ┌───────────────┐  │
│  │  CPU Cache    │  │     │  │  CPU Cache    │  │
│  │  flag = true  │  │     │  │  flag = false │  │
│  └───────┬───────┘  │     │  └───────┬───────┘  │
│          │ write     │     │          │ read     │
└──────────┼──────────┘     └──────────┼──────────┘
           │                           │
           ▼                           ▼
    ┌──────────────────────────────────────────┐
    │            Main Memory                   │
    │            flag = ???                     │
    │  (may or may not reflect Thread 1's      │
    │   write, depending on memory ordering)   │
    └──────────────────────────────────────────┘
```

Without `volatile` or `synchronized`, Thread 2 may read a stale cached value of `flag` indefinitely, even after Thread 1 has already written `true`.

**Example: Task Status**

Consider a simple class that stores a flag indicating whether a task is completed:


```java
public class TaskStatus {
    private boolean taskCompleted = false;

    public void setTaskCompleted() {
        taskCompleted = true;
    }

    public boolean isTaskCompleted() {
        return taskCompleted;
    }
}
```

In a multithreaded environment, one thread may update the taskCompleted variable, while another thread checks its value. Due to caching, the second thread might not see the updated value immediately (or ever), leading to incorrect behavior based on an outdated taskCompleted value.

To fix this issue, we can use the **volatile** keyword:

```java
public class TaskStatus {
    private volatile boolean taskCompleted = false;

    public void setTaskCompleted() {
        taskCompleted = true;
    }

    public boolean isTaskCompleted() {
        return taskCompleted;
    }
}
```

By declaring the taskCompleted variable as volatile, we guarantee that a write to this variable **happens-before** every subsequent read of it. In practice, this means any change made by one thread becomes visible to all other threads. The common description of volatile as "reading/writing directly to main memory" is a useful simplification. What the JVM actually guarantees is happens-before ordering as defined in the Java Memory Model. The actual implementation may involve cache coherence protocols rather than literal main memory access. The practical effect is the same: volatile ensures proper memory visibility but does not protect from race conditions.

It is also worth noting that `synchronized` provides memory visibility guarantees in addition to mutual exclusion. When a thread exits a synchronized block, all writes performed inside that block are guaranteed to be visible to any thread that subsequently enters a synchronized block on the same monitor. So if your code already uses `synchronized` for execution control, you get memory visibility for free.



### Where Volatile Falls Short: Read-Modify-Write

One of the trickiest misconceptions about `volatile` is that it makes all operations on a variable thread-safe. It does not. Volatile guarantees that reads and writes to the variable itself are visible to all threads, but it cannot make a compound operation atomic.

Consider a simple counter:

```java
public class VolatileCounter {
    private volatile int count = 0;

    public void increment() {
        count++; // NOT thread-safe, even with volatile
    }

    public int getCount() {
        return count;
    }
}
```

The expression `count++` looks like a single operation, but the JVM executes it as three separate steps:

1. **Read** the current value of `count`
2. **Modify** the value (add 1)
3. **Write** the new value back

With `volatile`, each of those individual reads and writes is visible to all threads. But nothing prevents two threads from both reading the same value (say, 42), both incrementing to 43, and both writing back 43. The result: one increment is silently lost.

```java
// Both threads read count = 42
// Thread 1: 42 + 1 = 43, writes 43
// Thread 2: 42 + 1 = 43, writes 43
// Expected count: 44, actual count: 43
```

This is the classic **read-modify-write** race condition, and `volatile` cannot prevent it. You need either `synchronized` or an atomic class to handle this correctly.



### Double-Checked Locking: A Practical Volatile Use Case

A classic example where `volatile` is the right tool is the **double-checked locking** pattern for lazy initialization. The goal is to create an instance only when it is first needed, while still being thread-safe and avoiding the cost of synchronization on every access.

```java
public class ExpensiveService {
    private static volatile ExpensiveService instance;

    private ExpensiveService() {
        // expensive initialization
    }

    public static ExpensiveService getInstance() {
        if (instance == null) {                // first check (no lock)
            synchronized (ExpensiveService.class) {
                if (instance == null) {        // second check (with lock)
                    instance = new ExpensiveService();
                }
            }
        }
        return instance;
    }
}
```

The `volatile` keyword is essential here, and the reason is subtle. Without it, the JVM is allowed to reorder the steps of object construction. The assignment `instance = new ExpensiveService()` involves allocating memory, calling the constructor, and assigning the reference. Without `volatile`, another thread could see a non-null `instance` that points to a partially constructed object, because the reference assignment was reordered before the constructor finished. Declaring `instance` as `volatile` prevents this reordering by establishing a happens-before relationship between the write and any subsequent read.



### Atomic Classes

In addition to using `synchronized` and `volatile`, Java provides a set of atomic classes in the `java.util.concurrent.atomic` package, which can be used as an alternative to manage shared variables in a more efficient way. These classes, such as AtomicInteger, AtomicLong, and AtomicReference, use low-level, lock-free operations (typically CAS, Compare-And-Swap) to provide atomicity.

```java
AtomicInteger counter = new AtomicInteger(0);

// Thread-safe increment without synchronized
counter.incrementAndGet(); // atomically increments and returns the new value

// Compare-and-swap: only updates if the current value matches the expected value
boolean updated = counter.compareAndSet(1, 10); // sets to 10 only if current value is 1
```

This is the correct solution for the counter problem shown earlier. Where `volatile` fails at `count++`, `AtomicInteger.incrementAndGet()` performs the read-modify-write as a single atomic operation using hardware-level CAS instructions.



### Performance Characteristics

The three approaches carry different performance costs, which matters when choosing between them:

* **`synchronized`**: Highest overhead. Acquiring and releasing a monitor lock involves OS-level operations (though the JVM optimizes uncontended locks with biased locking and lock elision). Under contention, threads block and context-switch, which is expensive. Use this when you need mutual exclusion or need to protect a sequence of operations.

* **`volatile`**: Moderate overhead. No locking is involved, but each read and write requires a memory barrier that prevents CPU caching optimizations and instruction reordering. Cheaper than `synchronized` by roughly an order of magnitude, but not free. Use this for simple flags and state variables where only one thread writes.

* **Atomic classes**: Lowest overhead for single-variable operations. CAS-based operations are lock-free and typically execute in just a few CPU cycles. Under low to moderate contention, they outperform `synchronized` significantly. Under very high contention, the CAS retry loop can waste CPU cycles, but for most practical workloads atomics are the fastest option for individual counters and references.

As a rough guideline: if you are protecting a single variable with simple operations (increment, set, compare-and-swap), atomic classes are almost always the best choice. If you need visibility for a flag that one thread sets and others read, `volatile` is sufficient. If you need to protect multiple variables or a sequence of operations as a unit, `synchronized` (or `java.util.concurrent` locks) is the right tool.



### When to Use Each

When designing thread-safe applications, it's crucial to choose the appropriate approach to manage shared variables. Here is a summary of when to use **synchronized**, **volatile**, and **atomic classes**:

* **Synchronized**: Use `synchronized` when you need to enforce mutual exclusion (i.e., only one thread can access the shared resource at a time) or when you need to ensure that a sequence of operations is atomic (i.e., executed without interruption). This is suitable for scenarios where multiple threads need to modify a shared variable, and the order of execution is critical. Examples include managing access to shared data structures, such as lists or maps, and implementing complex operations like depositing and withdrawing money from a bank account.

* **Volatile**: Use `volatile` when you want to guarantee memory visibility of a shared variable, ensuring that all threads see the most recent value of the variable. Volatile does provide ordering guarantees through the happens-before relationship (a write to a volatile variable happens-before every subsequent read of that variable), but it does not provide mutual exclusion or compound atomicity, so it is not suitable for scenarios where race conditions could arise from read-modify-write sequences. Good use cases include simple flags, status variables, and the double-checked locking pattern.

* **Atomic Classes**: Atomic classes, such as `AtomicInteger`, `AtomicLong`, and `AtomicReference`, are a more efficient alternative to using `synchronized` for managing shared variables. These classes provide atomic operations (e.g., `getAndSet`, `compareAndSet`) that guarantee both execution control and memory visibility. Use atomic classes when you need to perform atomic operations on shared variables without the overhead of synchronization.

In summary, choose **synchronized** when you need strict execution control, **volatile** for simple memory visibility, and **atomic classes** for efficient and fine-grained control over shared variables.

> **Related posts**: [Concurrency, Threading and Parallelism](/posts/concurrency-and-parallelism/), [Java Memory Management](/posts/java-memory-management/), [Singleton Pattern](/posts/singleton-pattern/)
