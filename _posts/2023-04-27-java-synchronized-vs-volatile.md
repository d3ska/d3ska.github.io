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
By declaring the taskCompleted variable as volatile, we ensure that all reads and writes to it happen directly in main memory, and any changes are immediately visible to all threads. This ensures proper memory visibility.

<br>

### Synchronized vs. Volatile: When to Use Each

Understanding the differences between synchronized and volatile can help you decide which to use depending on your specific use case.

* Use **synchronized** when you need to ensure execution control for a block of code and guarantee atomicity for compound operations, such as read-update-write sequences. Synchronization can be more expensive in terms of performance because it requires acquiring and releasing locks, which can lead to contention if multiple threads are competing for the same lock. However, it is necessary for complex scenarios where multiple operations need to be performed atomically or when a combination of shared variables must be accessed together.

* Use **volatile** when you only need to ensure memory visibility for individual reads and writes to a shared variable, and atomicity of compound operations is not required. Volatile is generally faster as it only ensures memory visibility without the need for locking. It is often the better choice for simpler scenarios, such as sharing a simple flag or single variable across threads with proper memory visibility.

In the context of read-update-write operations, volatile is generally insufficient for ensuring thread safety, as it only guarantees atomicity for individual reads and writes, not for compound operations like read-update-write. For read-update-write operations, you should use synchronized to ensure proper execution control and atomicity, or use Atomic classes provided by the 'java.util.concurrent.atomic' package.

In summary, consider the specific requirements of your multithreaded application when choosing between synchronized and volatile. Use synchronized for complex scenarios that require atomic compound operations and execution control, and use volatile for simpler scenarios that only require memory visibility for individual reads and writes. By making the right choice, you can ensure your application's thread safety while maximizing its performance and efficiency.