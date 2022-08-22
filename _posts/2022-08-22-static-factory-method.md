---
title: "Static Factory Method"
categories:
  - Blog
tags:
  - Effective Java
  - Clean code
---

In an object-oriented language like Java, what could be wrong with constructors? Overall, nothing. Even so, the famous Joshua Block's Effective Java Item 1 clearly states:

“Consider static factory methods instead of constructors”

The following are main pros of this approach: 

1. Static factory methods can have meaningful names, hence explicitly conveying what they do. Unlike the constructors.
2. Unlike constructors, they are not required to create a new object each time they’re invoked, we may implement singleton pattern.
3. Unlike constructors, they can return an object of any subtype of their return type.
4. The class of the returned object can vary from call to call as a function of the input parameters.
5. Static factory methods can encapsulate all the logic required for pre-constructing fully initialized instances, so they can be used for moving this additional logic out of constructors. This prevents constructors from performing further tasks, others than just initializing fields
6. A sixth advantage of static factories is that the class of the returned object need not exist when the class containing the method is written. 

The main con od providing only static factory methods is that classes without public or protected constructors cannot be subclassed.

A second shortcoming of static factory method is that they are hard for programmers to find.


**Example** <br>
There are plenty of examples of static factory method in JDK: 
```java
String value1 = String.valueOf(1);
String value2 = String.valueOf(1.0L);
String value3 = String.valueOf(true);
String value4 = String.valueOf('a');

Optional<String> value1 = Optional.empty();
Optional<String> value2 = Optional.of("Baeldung");
Optional<String> value3 = Optional.ofNullable(null);


//implementation of static factory method in Boolean class
public static Boolean valueOf(boolean b) {
return b ? Boolean.TRUE : Boolean.FALSE;
}
```

Besides, the above examples what's worth mentioning is I think the most representative example of static factory methods in the JDK which is Collections class. <br>
This is a non-instantiable class that implements only static methods.

Few examples of Collections static factory methods:

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
        return new Client(name, phoneNo, Country.POLAND);
    }

}

}
```

```java
Client client = Client.createWithDefaultCountry("John", 001111111);
```




