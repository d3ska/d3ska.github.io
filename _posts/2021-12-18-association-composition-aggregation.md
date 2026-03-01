---
title: "Composition, Aggregation and Association"
categories:
  - Software Design
tags:
  - Composition
  - Association
  - Aggregation
  - OOP
  - Object Relationships
---

In real life and in programming, objects have relationships between each other. Three of them get mixed up constantly: **composition**, **aggregation**, and **association**. They all describe how objects relate, but they differ in one critical aspect: who owns whom, and what happens when the owner disappears.

> **Related posts**: [What is coupling?](/posts/what-is-coupling/) and [Core concepts behind OOP](/posts/core-concepts-behind-oop/)

### Composition

Composition is a "has-a" or "part-of" relationship where the containing object **owns** its parts. The parts cannot exist independently. If the whole is destroyed, the parts are destroyed with it.

Think of a human body and a heart. A heart has no meaningful existence outside of the human it belongs to. The human creates (or at least fully owns) the heart, and when the human ceases to exist, so does the heart. That is the defining characteristic of composition: the lifecycle of the part is entirely controlled by the whole.

In code, this means the owning class creates its parts internally (typically in the constructor) and does not expose setters that would let external code swap them out.

**UML**

![img]({{site.url}}/assets/blog_images/2021-12-18-association-composition-aggregation/composition-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2021-12-18-association-composition-aggregation/composition-dark.png){: .dark }

**Code**

```java
public class Heart {
    private int beatsPerMinute;

    public Heart(int beatsPerMinute) {
        this.beatsPerMinute = beatsPerMinute;
    }

    public void beat() {
        // ...
    }
}

public class Human {
    private final Heart heart;  // created internally, no setter

    public Human() {
        this.heart = new Heart(72);  // Human owns the Heart's lifecycle
    }

    public void live() {
        heart.beat();
    }
}
```

Notice that `Heart` is created inside the `Human` constructor. There is no way for external code to inject a different heart or remove it. When a `Human` instance is garbage collected, its `Heart` goes with it. That is composition.

### Aggregation

Aggregation is also a "has-a" relationship, but it does not involve ownership. The parts can exist independently of the whole, and they can even be shared or moved between different containers.

A car and its wheels are a good example. You can take the wheels off, and they still exist as perfectly valid objects. You can mount them on a different car. The car assembles wheels into something greater, but it does not own their lifecycle.

In code, aggregated parts are passed in from the outside (constructor injection or setters). The container holds references to them but does not create or destroy them.

**UML**

![img]({{site.url}}/assets/blog_images/2021-12-18-association-composition-aggregation/aggregation-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2021-12-18-association-composition-aggregation/aggregation-dark.png){: .dark }

**Code**

```java
public class Wheel {
    private String brand;

    public Wheel(String brand) {
        this.brand = brand;
    }
}

public class Car {
    private List<Wheel> wheels;

    public Car(List<Wheel> wheels) {  // wheels come from outside
        this.wheels = wheels;
    }

    public void setWheels(List<Wheel> wheels) {  // wheels can be swapped
        this.wheels = wheels;
    }
}
```

The key difference from composition: the `Car` does not create its `Wheel` objects. They are injected from the outside, and if the `Car` is destroyed, the wheels can still be referenced elsewhere. They survive independently.

### Association

Association is the weakest of the three. **It is not a "has-a" relationship at all.** It simply means the objects "know about" each other. Neither object owns or manages the other's lifecycle. Both exist completely independently.

Consider a student and a university. A student knows which university they attend, and the university might have a record of the student. But neither one depends on the other for its existence. The student can transfer, graduate, or drop out, and both the student and the university continue to exist just fine.

**UML**

![img]({{site.url}}/assets/blog_images/2021-12-18-association-composition-aggregation/association-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2021-12-18-association-composition-aggregation/association-dark.png){: .dark }

**Code**

```java
public class University {
    private String name;

    public University(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}

public class Student {
    private String name;
    private University university;  // optional, can change at any time

    public Student(String name) {
        this.name = name;
    }

    public void enrollAt(University university) {
        this.university = university;  // the student "knows about" the university
    }

    public void leave() {
        this.university = null;  // and can stop knowing about it
    }
}
```

The `Student` does not create the `University`, does not receive it as a required constructor argument, and can set or clear the reference at will. The relationship is temporary and optional. That is association.

### A Note on Code Similarities

Looking at the three code examples above, you will notice something important: structurally, they all boil down to a class holding a reference to another class. Java (and most OOP languages) has no syntax-level keyword that distinguishes composition from aggregation from association. The difference is **semantic**, expressed through lifecycle management and intent.

The way I enforce these semantics in code is through three things:

* **Constructor design** -- composition creates parts internally; aggregation receives them from outside; association may not even require them at construction time.
* **Setter visibility** -- composition typically has no public setter for the part. Aggregation may expose a setter for swapping parts. Association freely allows setting and clearing references.
* **Lifecycle responsibility** -- if the owner creates and destroys the part, it is composition. If the owner merely holds a reference, it is aggregation or association.

The language itself will not stop you from violating the contract. I have seen codebases where a "composition" relationship had a public setter that allowed swapping out the part, effectively turning it into aggregation. Understanding the intent behind each relationship matters more than the syntax.

### How to Choose the Right Relationship

When I am modeling a new domain, I use a simple decision heuristic:

- **If the part cannot exist without the whole**, use composition. The whole creates the part, owns it, and destroys it. Example: a `Human` and its `Heart`.
- **If the part can be shared or moved between wholes**, use aggregation. The whole uses the part, but the part has its own lifecycle. Example: a `Car` and its `Wheel` objects.
- **If the objects merely interact but have independent lifecycles**, use association. Neither object owns the other. Example: a `Student` and a `University`.

In practice, the boundary between aggregation and association can be blurry. If you are unsure, ask yourself: "Does object A *contain* object B as a structural part, or does A merely *know about* B?" If it is containment, lean toward aggregation. If it is just a reference, lean toward association.

### Bidirectional vs Unidirectional Association

Associations can be **unidirectional** or **bidirectional**:

* **Unidirectional** -- only one class knows about the other. For example, a `Student` may hold a reference to a `University`, but the `University` class does not store references to individual students.
* **Bidirectional** -- both classes reference each other. A `Patient` knows about their `Doctor`, and the `Doctor` also holds a reference to the `Patient`.

In practice I default to unidirectional associations whenever possible, because bidirectional ones introduce tighter coupling and make it harder to reason about object graphs (and can complicate serialization/ORM mappings).

### Dependency -- the Weakest Link

There is actually a relationship even weaker than association: **dependency**. A dependency exists when one class *uses* another, but only temporarily, for example, as a method parameter or a local variable. The dependent class does not hold a long-lived reference to the other.

```java
public class ReportGenerator {
    public void generate(Printer printer) {  // dependency: used, not stored
        printer.print("Report content");
    }
}
```

In UML, dependency is drawn as a dashed arrow. It signals the weakest possible coupling: `ReportGenerator` knows about `Printer` only for the duration of the `generate` call.

The full spectrum from weakest to strongest: **Dependency < Association < Aggregation < Composition**.
