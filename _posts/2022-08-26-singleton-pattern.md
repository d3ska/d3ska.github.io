---
title: "Singleton Pattern"
categories:
  - Design Patterns
tags:
  - Singleton
  - Design Patterns
  - Creational Patterns
---

### The Problem

Some resources should exist exactly once in an application. A database connection pool, a thread-safe cache, or a logging configuration are examples where creating a second instance would be wasteful or actively harmful. Two connection pools mean double the connections. Two caches mean stale data in one of them.

The **Singleton** is a creational design pattern that restricts a class to a single instance and provides a global access point to it. The idea is straightforward: make the constructor private so nobody else can call it, and expose the one instance through a static method or field.

There are several ways to implement this in Java, each with different trade-offs around thread safety, lazy initialization, and resilience to attacks.

### Eager Initialization

The simplest approach creates the instance when the class is loaded:

```java
public class ConnectionPool {

    public static final ConnectionPool INSTANCE = new ConnectionPool();

    private ConnectionPool() {
        // initialize pool
    }
}
```

A variation hides the field behind a static factory method, which gives you the flexibility to change the implementation later without changing the API:

```java
public class ConnectionPool {

    private static final ConnectionPool INSTANCE = new ConnectionPool();

    private ConnectionPool() {}

    public static ConnectionPool getInstance() {
        return INSTANCE;
    }
}
```

Both approaches are thread-safe because the JVM guarantees that static fields are initialized exactly once during class loading. The private constructor is required even though it is empty, because without it the compiler generates a public default constructor, defeating the entire purpose.

The downside is that the instance is created even if the application never uses it. For lightweight objects this is irrelevant, but for expensive resources it can matter.

### Lazy Initialization with Double-Checked Locking

If construction is expensive and you want to defer it until the first call, you need lazy initialization. The naive approach (check if null, then create) is not thread-safe: two threads could both see `null` and each create an instance. The standard solution is double-checked locking:

```java
public class ConnectionPool {

    private static volatile ConnectionPool instance;

    private ConnectionPool() {}

    public static ConnectionPool getInstance() {
        if (instance == null) {
            synchronized (ConnectionPool.class) {
                if (instance == null) {
                    instance = new ConnectionPool();
                }
            }
        }
        return instance;
    }
}
```

The `volatile` keyword is essential here, and the reason is subtle. Without it, the JVM is allowed to reorder the steps of object construction. Thread A might write the reference to `instance` before the constructor body has fully executed. Thread B would then see a non-null reference (skipping the synchronized block entirely) and use a partially constructed object. The `volatile` modifier establishes a happens-before relationship: it guarantees that all writes performed during construction are visible to any thread that subsequently reads the field.

The outer null check avoids the cost of synchronization after the instance has been created. The inner null check prevents the race condition where two threads enter the synchronized block before the instance exists.

### Bill Pugh Singleton (Initialization-on-Demand Holder)

This idiom achieves lazy initialization and thread safety without `volatile` or `synchronized`:

```java
public class ConnectionPool {

    private ConnectionPool() {}

    private static class Holder {
        private static final ConnectionPool INSTANCE = new ConnectionPool();
    }

    public static ConnectionPool getInstance() {
        return Holder.INSTANCE;
    }
}
```

This works because the Java Language Specification (JLS 12.4) guarantees that a class is not initialized until it is first referenced. The `Holder` class is only referenced when `getInstance()` is called, so the instance is created lazily. Class initialization is also inherently thread-safe: the JVM acquires an internal lock during class loading, so no explicit synchronization is needed.

This makes the Bill Pugh approach one of the cleanest singleton implementations in Java.

### Enum Singleton

Joshua Bloch (Effective Java, Item 3) recommends using an enum:

```java
public enum ConnectionPool {
    INSTANCE;

    private final List<Connection> connections = new ArrayList<>();

    public Connection acquire() {
        // ...
    }

    public void release(Connection connection) {
        // ...
    }
}
```

Enums are inherently serializable, reflection-proof, and thread-safe. The JVM guarantees that enum values are instantiated exactly once. This eliminates entire categories of problems that the other approaches must handle manually.

The trade-off is that enums cannot extend other classes (every enum implicitly extends `java.lang.Enum`). If your singleton needs to inherit from a specific base class, the enum approach will not work. Additionally, when serializing an enum, field values are not preserved. If you serialize and deserialize the `ConnectionPool` above, the `connections` list will be empty on the other side.

### Vulnerabilities: Serialization and Reflection

The non-enum approaches are vulnerable to two attacks that can break the singleton guarantee.

**Serialization** creates a new instance during deserialization, bypassing the private constructor:

```java
// Serialize
ConnectionPool original = ConnectionPool.getInstance();
ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream("pool.ser"));
out.writeObject(original);
out.close();

// Deserialize — creates a SECOND instance
ObjectInputStream in = new ObjectInputStream(new FileInputStream("pool.ser"));
ConnectionPool deserialized = (ConnectionPool) in.readObject();
in.close();

System.out.println(original == deserialized); // false
```

The fix is to implement `readResolve`, which tells the serialization framework to substitute the deserialized object with the existing instance:

```java
private Object readResolve() {
    return getInstance();
}
```

**Reflection** can also break the singleton by making the private constructor accessible:

```java
Constructor<ConnectionPool> constructor = ConnectionPool.class.getDeclaredConstructor();
constructor.setAccessible(true);
ConnectionPool second = constructor.newInstance(); // second instance created
```

There is no clean way to prevent this in a regular class. You can throw an exception from the constructor if the instance already exists, but this is fragile. The enum approach sidesteps both problems entirely because the JVM enforces enum instantiation rules at a level that reflection cannot bypass.

### Comparison

| Approach | Thread-safe | Lazy | Serialization-safe | Reflection-safe |
|---|---|---|---|---|
| Public static final field | Yes | No | No | No |
| Static factory method | Yes | No | No | No |
| Double-checked locking | Yes | Yes | No | No |
| Bill Pugh (holder idiom) | Yes | Yes | No | No |
| Enum | Yes | No | Yes | Yes |

### Why Singleton Is Often an Anti-Pattern

Knowing how to implement a singleton is useful. Knowing when not to use one is more important.

**Global mutable state.** A singleton is a global variable in object-oriented clothing. Any part of the codebase can access and mutate its state, which makes data flow hard to trace and introduces hidden coupling between components that should be independent.

**Testing difficulty.** Because the instance is shared across the entire application, tests cannot easily replace it with a test double. One test can silently affect another through the singleton's state, leading to flaky and order-dependent test suites.

**Tight coupling.** Classes that call `ConnectionPool.getInstance()` are coupled to both the singleton class and its lifecycle. This makes it harder to swap implementations, apply the [dependency inversion principle](/posts/solid-the-first-5-principles-of-object-oriented-design/), or reuse those classes in a different context.

### The Modern Alternative: Dependency Injection

Dependency injection frameworks manage object lifecycles without the drawbacks of the classic singleton pattern. Spring, for example, makes every bean a singleton by default:

```java
@Configuration
public class AppConfig {

    @Bean
    public ConnectionPool connectionPool() {
        return new ConnectionPool(dataSource, poolSize);
    }
}
```

```java
@Service
public class OrderService {

    private final ConnectionPool connectionPool;

    public OrderService(ConnectionPool connectionPool) {
        this.connectionPool = connectionPool;
    }
}
```

The container creates exactly one `ConnectionPool` and injects it wherever needed. `OrderService` does not know or care that only one instance exists. It declares a dependency and receives it through the constructor. Testing becomes trivial: just pass a different `ConnectionPool` (or a mock) in the test.

If you need a different scope (one instance per HTTP request, per session, or per thread), you change the scope annotation on the bean definition, not the class itself:

```java
@Bean
@Scope("request")
public ShoppingCart shoppingCart() {
    return new ShoppingCart();
}
```

### When Singleton Is Still Appropriate

Despite its drawbacks, the classic singleton pattern has legitimate uses. The key condition is that the class is **stateless** or holds **read-only configuration** that is truly global:

- **JDK utility singletons** like `Runtime.getRuntime()` and `Collections.EMPTY_LIST`.
- **Logging facades** that delegate to a configured backend.
- **Read-only configuration** loaded once at startup and never modified.

If the singleton carries mutable state, manages a resource lifecycle, or needs to be swapped during testing, a DI container is almost always a better fit.

> **Related posts**: [Static Factory Method](/posts/static-factory-method/), [Inversion of Control and Dependency Injection](/posts/inversion-of-control-and-the-dependency-injection/)
