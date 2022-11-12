---
title: "Singleton Pattern"
categories:
  - Blog
tags:
  - Aggregate
  - Patterns
  - Domain Driven Design
  - DDD
---

#### Singleton

Is a one of the most known design patterns. To be more particular, it's a creational design pattern, which ensures that a class would have just one instance.
So the all clients of that class would reuse that instance, that's why it's called singleton. 

You may be wondering why would someone want to have single instance of some class.
Let's imagine some shared resource, like class with database connection, or some service class, or any other class which should 
have just a single instance available to all clients.


Please note that we would need to get that object not by a classic constructor as by definition it requires returning new instance.
There are several ways to implementing singleton pattern, like:

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

    private static Singleton INSTANCE = null;

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


##### Advantages and disadvantages of those methods 

First of all of them ensure non-insatiability by defining private constructor explicitly.
It's required even though it's empty. Because otherwise by default, constructor is created with the same access modifier as the class.
So when class is public the default constructor either, same for protected.
<br>
The next thing that we should consider. When the singleton instance is created? 
The first and second proposed method haven't any performance difference.  
Minor advantage of the second approach is that it hide its implementation behind a [static factory method](https://matthewonsoftware.com/blog/static-factory-method/), so we've possibility to change an implementation without changing an API. 
 
<br>
<br>

Static fields are initialized at class loading time. So the advantage of the last proposed method is fact that is not created until we need it during a runtime.
In the other methods we create an object always at class loading time, with possibility that we not gonna use it later. 
It's not big issue as long as creating the instance is not too expensive.
<br>
Are those methods the best and safer option?

Let's answer to the question. Is there any other ways to create an instance of a class other than the constructor?
The answer is yes! There is possibility of initializing object using **serialization and deserialization** or **reflection**.


To make class serializable we need to implement Serializable interface.
But it's not enough to ensure that our class would not be initializing twice.

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
Also there are some **constraints** with enums. In regular classes, there are things that can be achieved but prohibited in enum classes.
For example accessing static field in a constructor.
