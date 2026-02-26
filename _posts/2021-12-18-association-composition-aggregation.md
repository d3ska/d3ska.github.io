---
title: "Composition, Aggregation and Association"
categories:
  - Blog
tags:
  - Composition
  - Association
  - Aggregation
  - OOP
---

#### Introduction

In real life and in programming objects have relationships between each other.
Below we will discuss three of them which are quite often mixed up: composition, aggregation, and association.

<br>

#### Composition:

It is a relationship that is often described in words 'has-a', 'belongs-to' or 'part-of'. It means that one of the objects is a logical structure
which has another object. In other words, one is a part of the other object.

For example class belongs to school or in other words school has a class. 
So as you see, whether we call it “belongs-to” or “has-a” is only a matter of point of view.

**Composition is a restricted form of "has-a" relationship because the containing object owns it.
The members or parts that 'belongs-to' could not exist without the owning object that has them.**
So class could not exist without a school.


**UML**

![img]({{site.url}}/assets/blog_images/2021-12-18-association-composition-aggregation/composition-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2021-12-18-association-composition-aggregation/composition-dark.png){: .dark }


**Code**
```java
public class Room {}

public class School {
    private List<Room> rooms;
}
```

<br>

#### Aggregation:

It is also a 'has-a' relationship, but contrary to composition, the fact is that it doesn't involve owning, so it is a weaker relationship.

For example, imagine a car and its wheels. We can take off the wheels, and they will still exist, independently of the car; moreover they may even be installed in another car.

It is used to assemble the parts to a bigger construct, which is capable of more things than its parts.
    
**UML**

![img]({{site.url}}/assets/blog_images/2021-12-18-association-composition-aggregation/aggregation-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2021-12-18-association-composition-aggregation/aggregation-dark.png){: .dark }

**Code**
```java
public class Wheel {}

public class Car {
    private List<Wheel> wheels;
}
```

<br>

#### Association:

It is the weakest of those three relationships; none of the objects belong to another, **so it isn't a "has-a" relationship.**

**Association only means that the objects “know” each other.** For example, a mother and her child.


**UML**

![img]({{site.url}}/assets/blog_images/2021-12-18-association-composition-aggregation/association-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2021-12-18-association-composition-aggregation/association-dark.png){: .dark }

**Code**
```java
public class Child {}

public class Mother {
    private List<Child> children;
}
```

<br>

#### A Note on Code Similarities

Looking at the three code examples above, you will notice they all look structurally identical -- a class holding a `List<>` of another class. In Java (and most OOP languages), there is no syntax-level keyword that distinguishes composition from aggregation from association. The difference is **semantic**, expressed through lifecycle management and intent:

* **Composition** -- the owning class **creates and destroys** its parts. If `School` is deleted, its `Room` objects cease to exist. In practice, the rooms are typically instantiated inside the constructor or a factory method of `School`, and there is no public setter that allows external code to swap them out.
* **Aggregation** -- the container **references** parts that can exist independently. A `Car` holds references to `Wheel` objects, but those wheels can be removed and attached to a different car. The wheels are usually passed in from outside (constructor injection or setters).
* **Association** -- neither object owns or manages the other's lifecycle. A `Mother` knows about her `Child` objects, and those children can also hold a reference back to the mother. Both exist independently.

The way I enforce these semantics in code is through constructor design, visibility of setters, and documentation (or UML). The language itself won't stop you from violating the contract, so understanding the intent behind each relationship is essential.

<br>

#### Bidirectional vs Unidirectional Association

Associations can be **unidirectional** or **bidirectional**:

* **Unidirectional** -- only one class knows about the other. For example, a `Student` may hold a reference to a `University`, but the `University` class does not store references to individual students.
* **Bidirectional** -- both classes reference each other. A `Mother` knows about her `Child`, and the `Child` also holds a reference to the `Mother`.

In practice I default to unidirectional associations whenever possible, because bidirectional ones introduce tighter coupling and make it harder to reason about object graphs (and can complicate serialization/ORM mappings).

<br>

#### Dependency -- the Weakest Link

There is actually a relationship even weaker than association: **dependency**. A dependency exists when one class *uses* another, but only temporarily -- for example, as a method parameter or a local variable. The dependent class does not hold a long-lived reference to the other.

```java
public class ReportGenerator {
    public void generate(Printer printer) {  // dependency: used, not stored
        printer.print("Report content");
    }
}
```

In UML, dependency is drawn as a dashed arrow. It signals the weakest possible coupling: `ReportGenerator` knows about `Printer` only for the duration of the `generate` call.

The full spectrum from weakest to strongest: **Dependency < Association < Aggregation < Composition**.
