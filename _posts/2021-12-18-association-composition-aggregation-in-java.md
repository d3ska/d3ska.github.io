---
title: "Composition, Aggregation and Association in Java"
categories:
  - Blog
tags:
  - Composition
  - Association
  - Aggregation
  - OOP
---

#### Introduce

In real life and in programming objects have relationships between each other. 
Below we will discuss three of them which are quite often mixed up composition, aggregation, and association.

<br>

#### Composition:

Its relationship that is often describe in words 'has-a', 'belongs-to' or 'part-of'. It means that one of the object is a logically structure
which has another object. In other words it is a part of the other object.

For example class belongs to school or in other words school has a class. 
So as you see, whether we call it “belongs-to” or “has-a” is only a matter of point of view.

**Composition is a restricted form of "has-a" relationship because the containing object owns it.
The members or parts that 'belongs-to' could not exist without the owning object that has them.**
So class could not exist without a school.


**UML**

![img]({{site.url}}/assets/blog_images/2021-12-18-association-composition-aggregation-in-java/composition.png)


**Code**
```java
public class Room {}

public class School {
    private List<Room> rooms;
}
```

<br>

#### Aggregation:

It is also 'has-a' relationship, but in contrary to composition is fact, that it doesn't involve owning, so its weaker relationship.

For example, imagine a car and its wheels. We can take of the wheels, and they will still exist, independently of the car, moreover they may be even install in another car.

It is used to assemble the parts to a bigger construct, which is capable of more things than its parts.
    
**UML**

![img]({{site.url}}/assets/blog_images/2021-12-18-association-composition-aggregation-in-java/aggregation.png)

**Code**
```java
public class Wheel {}

public class Car {
    private List<Wheel> wheels;
}
```

<br>

#### Association:

It is the weakest of those three relationship, none of the objects belongs to another, **so it isn't a "has-a" relationship.**

**Association only means that the objects “know” each other.** For example, a mother and her child.


**UML**

![img]({{site.url}}/assets/blog_images/2021-12-18-association-composition-aggregation-in-java/association.png)

**Code**
```java
public class Child {}

public class Mother {
    private List<Child> children;
}
```
