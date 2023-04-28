---
title: "Java Synchronized vs Volatile"
categories:
  - Concurrency
tags:
  - Synchronized
  - Volatile
  - Multithreading
---

### Execution Control and Memory Visibility

Thread safety has two primary aspects:

* Execution Control
* Memory Visibility

Understanding the differences between execution control and memory visibility is crucial for designing safe and efficient multithreaded applications.

<br>

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

<br>

### Memory Visibility

Memory visibility deals with the timing of when the effects of what has been done in memory are visible to other threads. Proper memory visibility is essential to prevent threads from working with stale or inconsistent data.

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

In a multithreaded environment, one thread may update the taskCompleted variable, while another thread checks its value. Due to caching, the second thread might not see the updated value immediately, leading to incorrect behavior based on an outdated taskCompleted value.

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
By declaring the taskCompleted variable as volatile, we ensure that all reads and writes to it happen directly in main memory, and any changes are immediately visible to all threads. This ensures proper memory visibility but does not protect from race conditions.

<br>

### Atomic Classes

In addition to using 'synchronized' and 'volatile', Java provides a set of atomic classes in the 'java.util.concurrent.atomic' package, which can be used as an alternative to manage shared variables in a more efficient way. These classes, such as AtomicInteger, AtomicLong, and AtomicReference, use low-level, lock-free operations to provide atomicity

<br>

### When to Use Each

When designing thread-safe applications, it's crucial to choose the appropriate approach to manage shared variables. Here is a summary of when to use **synchronized**, **volatile**, and **atomic classes**:

* **Synchronized**: Use 'synchronized' when you need to enforce mutual exclusion (i.e., only one thread can access the shared resource at a time) or when you need to ensure that a sequence of operations is atomic (i.e., executed without interruption). This is suitable for scenarios where multiple threads need to modify a shared variable, and the order of execution is critical. Examples include managing access to shared data structures, such as lists or maps, and implementing complex operations like depositing and withdrawing money from a bank account.

* **Volatile**: Use 'volatile' when you want to guarantee memory visibility of a shared variable, ensuring that all threads see the most recent value of the variable. However, 'volatile' does not provide any guarantee on the order of execution, so it is not suitable for scenarios where race conditions could arise. An example use case for volatile is a simple flag that indicates the completion of a task, where one thread updates the flag, and others only read it.

* **Atomic Classes**: Atomic classes, such as 'AtomicInteger', 'AtomicLong', and 'AtomicReference', are a more efficient alternative to using 'synchronized' for managing shared variables. These classes provide atomic operations (e.g., 'getAndSet', 'compareAndSet') that guarantee both execution control and memory visibility. Use atomic classes when you need to perform atomic operations on shared variables without the overhead of synchronization.

In summary, choose **synchronized** when you need strict execution control, volatile for simple memory visibility, and atomic classes for efficient and fine-grained control over shared variables.