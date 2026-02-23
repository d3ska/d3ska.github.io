---
title: "Static Factory Method"
categories:
  - Blog
tags:
  - Effective Java
  - Clean code
---

In an object-oriented language like Java, what could be wrong with constructors? Overall, nothing. Even so, the famous Joshua Bloch's Effective Java Item 1 clearly states:

"Consider static factory methods instead of constructors"

The following are main pros of this approach:

1. Static factory methods can have meaningful names, hence explicitly conveying what they do. Unlike the constructors.
2. Unlike constructors, they are not required to create a new object each time they're invoked, so we may implement the singleton pattern or cache frequently requested instances.
3. Unlike constructors, they can return an object of any subtype of their return type. For example, `EnumSet` has no public constructors -- its static factory `EnumSet.of(...)` returns either a `RegularEnumSet` or a `JumboEnumSet` depending on the number of elements, and the caller never needs to know which concrete class it receives.
4. The class of the returned object can vary from call to call as a function of the input parameters. A good example is `EnumSet` again: when the underlying enum has 64 or fewer elements, the factory returns a `RegularEnumSet` backed by a single `long`; for larger enums it returns a `JumboEnumSet` backed by a `long[]`. The caller code stays the same regardless.
5. Static factory methods can encapsulate all the logic required for pre-constructing fully initialized instances, so they serve well for moving this additional logic out of constructors. This prevents constructors from performing further tasks other than just initializing fields.
6. A sixth advantage of static factories is that the class of the returned object need not exist when the class containing the method is written. This is the foundation for service-provider frameworks like JDBC. When I call `DriverManager.getConnection(url)`, the actual `Connection` implementation class can come from a driver JAR that did not exist when `DriverManager` was compiled.

**Cons**

The main con of providing only static factory methods is that classes without public or protected constructors cannot be subclassed. For example, if `Collections` had convenience classes you wanted to extend, you would be out of luck -- all its utility types are package-private. In practice, this limitation can actually push you toward composition over inheritance, which is often a better design choice anyway.

A second shortcoming of static factory methods is that they are hard for programmers to find. Constructors stand out clearly in API documentation, while factory methods blend in with every other static method. Following well-known naming conventions helps mitigate this -- names like `of`, `valueOf`, `getInstance`, `create`, and `newInstance` signal to readers that the method is a factory.

A third downside worth mentioning: when a class exposes many factory methods with similar parameter lists, the API can become confusing. Careful naming and thorough Javadoc go a long way here.

**Example** <br>
There are plenty of examples of static factory methods in the JDK:
```java
String value1 = String.valueOf(1);
String value2 = String.valueOf(1.0L);
String value3 = String.valueOf(true);
String value4 = String.valueOf('a');

Optional<String> optValue1 = Optional.empty();
Optional<String> optValue2 = Optional.of("Baeldung");
Optional<String> optValue3 = Optional.ofNullable(null);


//implementation of static factory method in Boolean class
public static Boolean valueOf(boolean b) {
return b ? Boolean.TRUE : Boolean.FALSE;
}
```

The most representative example of static factory methods in the JDK is, in my opinion, the `Collections` class. It is a non-instantiable class that consists entirely of static methods. <br>

A few examples of Collections static factory methods:

![img]({{site.url}}/assets/blog_images/2022-08-22-static-factory-method/collections-method.png)

If you want to check some method of Collections class, you can find it [here](https://docs.oracle.com/javase/8/docs/api/java/util/Collections.html).

<br>

**Example of custom factory method**

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

    public static Client createWithDefaultCountry(String name, String phoneNo) {
        return new Client(name, phoneNo, "Poland");
    }

}
```

```java
Client client = Client.createWithDefaultCountry("John", "001111111");
```
