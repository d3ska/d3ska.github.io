---
title: "Singleton Pattern"
categories:
  - Design Patterns
tags:
  - Singleton
  - Design Patterns
  - Creational Patterns
---

#### Singleton

It is one of the most well-known design patterns. To be more particular, it is a creational design pattern that ensures a class has just one instance.
All clients of that class reuse that instance, which is why it is called a singleton.

You may wonder why someone would want to have a single instance of some class.
Imagine a shared resource, like a class with a database connection, or some service class, or any other class that should
have just a single instance available to all clients.

Please note that we cannot obtain that object through a classic constructor, since by definition a constructor requires returning a new instance.
There are several ways to implement the singleton pattern, like:

##### Singleton With Public Static Final Field

```java
public class Singleton {

    public static final Singleton INSTANCE = new Singleton();

    private Singleton() {}

}
```

<br>

##### Singleton With Public Static Factory Method

```java
public class Singleton {

    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {}

    public static Singleton getInstance(){
        return INSTANCE;
    }

}
```

<br>

##### Singleton With Lazy Initialization

```java
public class Singleton {

    private static volatile Singleton INSTANCE = null;

    private Singleton() {}

    public static Singleton getInstance() {
        if (INSTANCE == null) {
            synchronized (Singleton.class) {
                if (INSTANCE == null) {
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }

}
```

<br>

##### Bill Pugh Singleton (Initialization-on-Demand Holder)

The Bill Pugh approach, also known as the initialization-on-demand holder idiom, combines the benefits of lazy initialization with the simplicity of the static final field approach. It relies on the JLS guarantee that a nested class is not loaded until it is referenced for the first time.

```java
public class Singleton {

    private Singleton() {}

    private static class SingletonHolder {
        private static final Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return SingletonHolder.INSTANCE;
    }

}
```

Because the JVM loads `SingletonHolder` only when `getInstance()` is called, the singleton instance is created lazily. At the same time, class loading is inherently thread-safe, so there is no need for `synchronized` blocks or `volatile` fields. This makes the Bill Pugh approach one of the cleanest lazy singleton implementations in Java.

##### Advantages and disadvantages of those methods

First of all, each of these approaches ensures non-instantiability by defining a private constructor explicitly.
This explicit constructor is required even though it is empty, because without it the compiler generates a default constructor with the same access modifier as the class itself. A public class would therefore receive a public default constructor, defeating the entire purpose of the singleton.
<br>
The next thing to consider is when the singleton instance gets created.
The first and second proposed methods do not have any performance difference.
A minor advantage of the second approach is that it hides its implementation behind a [static factory method](https://matthewonsoftware.com/blog/static-factory-method/), so we have the possibility to change the implementation without changing the API.

<br>
<br>

Static fields are initialized at class loading time. So the advantage of the lazy initialization method (and the Bill Pugh approach) is the fact that the instance is not created until we actually need it at runtime.
In the eager approaches we create an object at class loading time, with the possibility that we might not use it later.
This is not a big issue as long as creating the instance is not too expensive.
<br>
Are those methods the safest option?

Let's answer the question. Are there any other ways to create an instance of a class other than the constructor?
The answer is yes! There is the possibility of initializing an object using **serialization and deserialization** or **reflection**.


To make a class serializable we need to implement the Serializable interface.
But that alone is not enough to ensure that our class would not be instantiated twice.

The solution is that we have to implement the readResolve method, which is called when preparing the deserialized object before returning it to the caller.

```java
public class Singleton implements Serializable{

    public static final Singleton INSTANCE = new Singleton();

    private Singleton() {
    }

    protected Object readResolve() {
        return INSTANCE;
    }

}
```


Our second issue is with **reflection**, because when a non-accessible private constructor becomes accessible, then the whole idea of making the class a singleton breaks.


<br>

##### Singleton with Enum

All the mentioned possible issues and others are resolved out of the box by Enum.


Enums are inherently serializable, so we don't have to worry about it.<br>
The reflection problem is also not there. <br>
Creation of Enum instance is also thread-safe, so we don't need to worry about double-checked locking.<br>
Thus, this method is recommended as **the best method of making singletons in Java.**

**Example:**

```java
public enum SingletonEnum {
    INSTANCE;

    String value;

    public String getValue() {
        return value;
    }

    public void setValue(String value) {
        this.value = value;
    }
}
```

**The disadvantage** is that when serializing an Enum, field variables are not getting serialized.
For example, if we serialize and deserialize the SingletonEnum class, we will lose the value of the String value field.
<br>
<br>
There are also some **constraints** with enums. In regular classes, there are things that can be achieved but are prohibited in enum classes.
For example, accessing a static field in a constructor. Additionally, enum singletons cannot extend other classes, since every enum implicitly extends `java.lang.Enum`. If the singleton needs to inherit from a specific base class, the enum approach is not an option.

<br>

##### Singleton as an Anti-Pattern

I should mention that the singleton pattern is a frequent subject of debate. Many experienced developers consider it an anti-pattern, and for good reason.

**Global mutable state.** A singleton is essentially a global variable dressed in object-oriented clothing. Any part of the codebase can access and mutate its state, making it difficult to reason about data flow and introducing hidden coupling between otherwise unrelated components.

**Testing difficulty.** Singletons make unit testing painful. Because the instance is shared across the entire application, tests cannot easily replace it with a mock or stub. One test can silently affect another through the singleton's state, leading to flaky and order-dependent test suites.

**Tight coupling.** Classes that call `Singleton.getInstance()` directly are tightly coupled to both the singleton class and its lifecycle. This makes it harder to swap implementations, apply the dependency inversion principle, or reuse those classes in a different context.

In modern applications, dependency injection frameworks (Spring, Guice, CDI) manage object lifecycles and scoping. They provide singleton-scoped beans without the drawbacks of the classic singleton pattern -- the container controls creation and wiring, while each class simply declares its dependencies. If I need a single shared instance, I reach for DI container scoping rather than the pattern described above.
