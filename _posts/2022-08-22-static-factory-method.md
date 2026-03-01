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

Here is a custom example of the same idea. The `Shape.of` method picks the right subclass based on the input:

```java
public interface Shape {
    double area();

    static Shape of(String type, double size) {
        return switch (type.toLowerCase()) {
            case "circle"    -> new Circle(size);
            case "square"    -> new Square(size);
            default          -> throw new IllegalArgumentException("Unknown shape: " + type);
        };
    }
}

class Circle implements Shape {
    private final double radius;

    Circle(double radius) { this.radius = radius; }

    @Override
    public double area() { return Math.PI * radius * radius; }
}

class Square implements Shape {
    private final double side;

    Square(double side) { this.side = side; }

    @Override
    public double area() { return side * side; }
}
```

```java
Shape circle = Shape.of("circle", 5.0);
Shape square = Shape.of("square", 3.0);
```

The caller never mentions `Circle` or `Square`. If a `Triangle` implementation is added tomorrow, existing client code does not change.

**Encapsulate construction logic.** When object creation involves validation, default values, or derived fields, a static factory keeps that logic out of the constructor. The constructor stays simple (just field assignment), and the factory method handles everything else.

**Decouple from concrete classes.** Because the factory method's return type can be an interface, the concrete implementation class does not need to exist at the time the factory is written. New implementations can be added later without changing the factory's API. This is the foundation for service-provider frameworks like JDBC. When you call `DriverManager.getConnection(url)`, the return type is the `Connection` interface. The actual implementation comes from a database driver JAR (MySQL, PostgreSQL, etc.) that was developed entirely independently of `DriverManager`.

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

### Factory Methods for Testing

Static factory methods can also make tests easier to write and read. Instead of constructing objects with long parameter lists in every test, you can provide factory methods with sensible defaults:

```java
public class Order {

    private final String customer;
    private final String product;
    private final int quantity;
    private final String shippingAddress;

    private Order(String customer, String product, int quantity, String shippingAddress) {
        this.customer = customer;
        this.product = product;
        this.quantity = quantity;
        this.shippingAddress = shippingAddress;
    }

    public static Order of(String customer, String product, int quantity, String shippingAddress) {
        return new Order(customer, product, quantity, shippingAddress);
    }

    /** Creates an Order with sensible defaults for tests. */
    public static Order createDefault() {
        return new Order("Test Customer", "Test Product", 1, "123 Test Street");
    }

    /** Creates a test Order where only the product matters. */
    public static Order createForTest(String product) {
        return new Order("Test Customer", product, 1, "123 Test Street");
    }

    // getters omitted for brevity
}
```

```java
// In production code
Order order = Order.of("Alice", "Laptop", 2, "456 Main St");

// In tests: focus on what matters, ignore irrelevant details
Order defaultOrder = Order.createDefault();
Order laptopOrder  = Order.createForTest("Laptop");
```

This pattern keeps tests focused. When only one field is relevant to the test, the rest get reasonable defaults and do not clutter the setup.

### When to Prefer Static Factory Methods

Use static factory methods when the class has multiple ways of being constructed and meaningful names would help the caller, when you want control over instance creation (caching, singletons, returning subtypes), or when construction logic is complex enough that it should not live inside a constructor. For simple value classes with one obvious way to construct them, a plain public constructor is perfectly fine.

> **Related posts**: [Builder Design Pattern](/posts/builder-design-pattern/), [Singleton Pattern](/posts/singleton-pattern/)
