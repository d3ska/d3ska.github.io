---
title: "Static Factory Method"
categories:
  - Design Patterns
tags:
  - Effective Java
  - Clean Code
  - Creational Patterns
  - Java
  - Design Patterns
---

Constructors in Java have a fundamental limitation: they must be named after the class. If a class needs to be created in different ways, you end up with multiple constructors that differ only in their parameter lists. The caller sees `new Order(String, String)` and has no idea what those two strings represent. Joshua Bloch's *Effective Java* (Item 1) puts it simply: "Consider static factory methods instead of constructors."

A **static factory method** is a static method that returns an instance of the class. It is not the same as the Factory Method design pattern from the Gang of Four. It is a simpler concept: instead of calling `new`, you call a named static method.

### Why Static Factory Methods?

**Meaningful names.** A constructor must match the class name. A static factory method can say exactly what it does:

```java
// Constructor: what do these parameters mean?
Order order = new Order("Warsaw", "Express");

// Static factory: the name tells you
Order order = Order.withExpressShipping("Warsaw");
```

**Control over instance creation.** A constructor always creates a new object. A static factory can return a cached instance, enforce a singleton, or return a pre-existing object. The `Boolean.valueOf` method is a classic example:

```java
public static Boolean valueOf(boolean b) {
    return b ? Boolean.TRUE : Boolean.FALSE;
}
```

It never creates a new `Boolean` object. It always returns one of the two cached constants.

**Return subtypes.** A constructor can only return the exact class it belongs to. A static factory can return any subtype of its declared return type. The caller programs against the interface and never needs to know the concrete class.

`EnumSet` is a great example: it has no public constructors. Its static factory `EnumSet.of(...)` returns either a `RegularEnumSet` (backed by a single `long`, for enums with 64 or fewer elements) or a `JumboEnumSet` (backed by a `long[]`, for larger enums). The caller just sees `EnumSet` and the JDK is free to change the internal implementations without breaking any code.

**Encapsulate construction logic.** When object creation involves validation, default values, or derived fields, a static factory keeps that logic out of the constructor. The constructor stays simple (just field assignment), and the factory method handles everything else.

**Decouple from concrete classes.** The class of the returned object need not exist when the class containing the factory method is written. This is the foundation for service-provider frameworks like JDBC. When you call `DriverManager.getConnection(url)`, the actual `Connection` implementation comes from a driver JAR that did not exist when `DriverManager` was compiled.

### Cons

The main limitation of providing only static factory methods is that classes without public or protected constructors cannot be subclassed. In practice, this can actually push you toward composition over inheritance, which is often a better design choice anyway.

A second shortcoming is discoverability. Constructors stand out clearly in API documentation, while factory methods blend in with every other static method. Following well-known naming conventions helps mitigate this.

### Naming Conventions

Common naming patterns that signal a method is a factory:

```java
// of - concise factory, common for value types
List<String> list = List.of("a", "b", "c");
EnumSet<Color> colors = EnumSet.of(Color.RED, Color.BLUE);

// valueOf - verbose alternative to of
Integer i = Integer.valueOf(42);
BigDecimal d = BigDecimal.valueOf(3.14);

// from - type-conversion factory
Instant instant = Instant.from(zonedDateTime);
Date date = Date.from(instant);

// create / newInstance - guarantees a new instance each time
Object obj = Array.newInstance(String.class, 10);

// getInstance - may return a shared instance
Calendar cal = Calendar.getInstance();
```

### JDK Examples

The `Collections` class is perhaps the most representative example. It is a non-instantiable utility class that consists entirely of static methods, many of which are factory methods:

```java
List<String> empty   = Collections.emptyList();
List<String> single  = Collections.singletonList("only");
Map<String, Integer> singleMap = Collections.singletonMap("key", 1);

// Wrapping factories: return a decorated view of the original collection
List<String> readOnly = Collections.unmodifiableList(mutableList);
List<String> safe     = Collections.synchronizedList(mutableList);
```

`Optional` is another good example:

```java
Optional<String> empty    = Optional.empty();
Optional<String> present  = Optional.of("value");
Optional<String> nullable = Optional.ofNullable(mayBeNull);
```

Each method name communicates exactly what kind of Optional you get back.

### Custom Factory Method

Here is a practical example for a domain class. The constructor is private, so the only way to create a `Client` is through the factory methods:

```java
public class Client {

    private final String name;
    private final String phoneNo;
    private final String country;

    private Client(String name, String phoneNo, String country) {
        this.name = name;
        this.phoneNo = phoneNo;
        this.country = country;
    }

    public static Client of(String name, String phoneNo, String country) {
        return new Client(name, phoneNo, country);
    }

    public static Client withDefaultCountry(String name, String phoneNo) {
        return new Client(name, phoneNo, "Poland");
    }
}
```

```java
Client client = Client.withDefaultCountry("John", "001111111");
```

The method name `withDefaultCountry` tells the caller exactly what is happening. A second constructor with two `String` parameters would be ambiguous and would not even compile if the three-parameter constructor already existed (both would have overlapping erasure in some cases).

### When to Prefer Static Factory Methods

Use static factory methods when the class has multiple ways of being constructed and meaningful names would help the caller, when you want control over instance creation (caching, singletons, returning subtypes), or when construction logic is complex enough that it should not live inside a constructor. For simple value classes with one obvious way to construct them, a plain public constructor is perfectly fine.

> **Related post:** [Builder Design Pattern](/posts/builder-design-pattern/)
