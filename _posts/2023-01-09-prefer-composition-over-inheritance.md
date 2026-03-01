---
title: "Prefer Composition over Inheritance"
categories:
  - Software Design
tags:
  - Inheritance
  - OOP
  - Composition
  - Design Principles
---

You've probably heard this statement before. It shows up in design books, code reviews, and conference talks. But it is easy to nod along without thinking about *why* composition wins in most situations, or what a properly composed design actually looks like. I've written an article that explains what composition is [here](/posts/association-composition-aggregation/).

## Introduction

Both concepts revolve around reusing pieces of code.

In general, it is widely considered good design to favor composition over inheritance. This preference arises because composition allows for greater flexibility and code reuse. It enables you to combine simple objects to achieve more complex behavior without the need to create intricate inheritance hierarchies.

The Gang of Four (GoF) captured this principle succinctly in *Design Patterns*: **"Program to an interface, not an implementation."** By depending on abstractions rather than concrete classes, you naturally gravitate toward composition. Your objects collaborate through well-defined interfaces, and you can swap implementations freely. Rigid inheritance hierarchies make that difficult.

## Composition vs. Inheritance

Inheritance can be inflexible because it involves creating a subclass that is tightly coupled to its superclass. This means that any changes to the superclass may necessitate changes to the subclass as well. This can become challenging to manage and maintain as the codebase expands. Moreover, inheritance can complicate code reuse because distinguishing between behavior inherited from the superclass and behavior specifically implemented in the subclass can be tricky.

In Java there is an additional constraint: a class can extend only one other class. If your `Car` already extends `Vehicle`, it cannot also extend `Loggable` or `Serializable` (as classes). Composition has no such limit. A class can hold references to as many collaborators as it needs, combining their capabilities without being locked into a single hierarchy.

On the other hand, composition empowers you to construct complex behavior by assembling simple objects. This makes it easier to reuse code and make changes to your codebase, as you can readily substitute different objects to attain the desired behavior.

Let's illustrate these concepts with Java examples.

## Examples in Java

### Composition Example

Consider a `Car` class that depends on an `Engine`. The key is that `Engine` is an interface, and `Car` receives it through its constructor rather than creating a concrete instance itself:

```java
interface Engine {
    void start();
    void stop();
}

class GasolineEngine implements Engine {
    @Override
    public void start() {
        // Fuel injection, ignition sequence
    }

    @Override
    public void stop() {
        // Cut fuel supply
    }
}

class ElectricEngine implements Engine {
    @Override
    public void start() {
        // Activate electric motor
    }

    @Override
    public void stop() {
        // Disengage motor
    }
}

class Car {
    private final Engine engine;

    Car(Engine engine) {
        this.engine = engine;
    }

    void drive() {
        engine.start();
    }
}
```

Now `Car` does not know or care what kind of engine it has. You can swap implementations without touching the `Car` class:

```java
Car gasCar = new Car(new GasolineEngine());
Car electricCar = new Car(new ElectricEngine());
```

This is exactly what the GoF meant by "program to an interface, not an implementation." The `Car` depends on the `Engine` abstraction, so adding a `HybridEngine` or a `HydrogenEngine` tomorrow requires zero changes to `Car`.

Notice that the choice of engine is made at runtime, not compile time. An inheritance hierarchy locks the relationship the moment you write `class GasCar extends Vehicle`. With composition, the same `Car` class can power a gasoline vehicle in one context and an electric vehicle in another, decided by whoever constructs it.

This also makes testing straightforward. When `Car` inherits from a superclass, testing it means testing the entire parent-child chain together. With composition, you inject a test double that implements `Engine` and verify `Car` in complete isolation. No real engine, no side effects, no fragile setup.

If the composition example had used `new GasolineEngine()` directly inside the `Car` constructor, we would have lost all of this flexibility. The `Car` would be permanently welded to one engine type, and at that point composition would offer little advantage over inheritance.

### Inheritance Example

Now consider a notification system where inheritance seems like a natural fit at first. You have a base `Notifier` class, and you want to support different channels:

```java
class Notifier {
    void send(String message) {
        // Send via email by default
        sendEmail(message);
    }

    private void sendEmail(String message) {
        // Email-sending logic
    }
}

class SmsNotifier extends Notifier {
    @Override
    void send(String message) {
        super.send(message);
        // Also send via SMS
        sendSms(message);
    }

    private void sendSms(String message) {
        // SMS-sending logic
    }
}
```

This works, but the problems surface quickly. What if you also need Slack notifications? You create `SlackNotifier extends Notifier`. Now what about SMS *and* Slack together? You cannot extend both `SmsNotifier` and `SlackNotifier`. You either create yet another subclass (`SmsAndSlackNotifier`) or start duplicating logic. Every new channel doubles the number of possible combinations, and the hierarchy becomes unmanageable.

With composition, the same problem stays simple. Each channel implements a `NotificationChannel` interface, and the notifier holds a list of channels:

```java
interface NotificationChannel {
    void send(String message);
}

class CompositeNotifier {
    private final List<NotificationChannel> channels;

    CompositeNotifier(List<NotificationChannel> channels) {
        this.channels = channels;
    }

    void send(String message) {
        channels.forEach(ch -> ch.send(message));
    }
}
```

Adding a new channel means writing one small class. Combining channels means passing a different list to the constructor. No combinatorial explosion, no deep hierarchies.

### Violation of Encapsulation

Subclasses rely on the implementation details of their superclasses for correct operation. The implementation within a superclass may vary across different releases, potentially leading to a breakdown in the subclass, despite no changes to its own code.
Consequently, a subclass necessitates co-evolution with its superclass, unless the superclass has been clearly designed and documented by its authors for the purpose of extension.

The classic example (from Joshua Bloch's *Effective Java*) is an `InstrumentedHashSet` that tries to count how many elements have been added:

```java
class InstrumentedHashSet<E> extends HashSet<E> {
    private int addCount = 0;

    @Override
    public boolean add(E e) {
        addCount++;
        return super.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return super.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }
}
```

This looks correct, but it is broken. `HashSet.addAll()` internally delegates to `add()` for each element. When you call `addAll(List.of("a", "b", "c"))`, the count goes up by 3 inside your `addAll` override, and then by 3 *again* as `super.addAll()` calls your overridden `add()` for each element. The result is 6, not 3.

The subclass did nothing wrong. It simply could not know how `HashSet` implements `addAll()` internally, because that is an implementation detail the superclass never promised to keep stable. This is exactly what "violation of encapsulation" means in practice: your subclass depends on internal behavior that can change without warning.

A composition-based wrapper avoids this entirely:

```java
class InstrumentedSet<E> implements Set<E> {
    private final Set<E> delegate;
    private int addCount = 0;

    InstrumentedSet(Set<E> delegate) {
        this.delegate = delegate;
    }

    @Override
    public boolean add(E e) {
        addCount++;
        return delegate.add(e);
    }

    @Override
    public boolean addAll(Collection<? extends E> c) {
        addCount += c.size();
        return delegate.addAll(c);
    }

    public int getAddCount() {
        return addCount;
    }

    // Remaining Set methods delegate to 'delegate'
    // ...
}
```

Now `delegate.addAll()` calls `delegate.add()` internally, not your `add()`. The count stays accurate regardless of how the underlying `Set` implementation works.

## When to Use Each

Composition is the safer default. It keeps your classes small, loosely coupled, and easy to test. When you depend on interfaces and inject collaborators through constructors, you get flexibility that inheritance simply cannot match.

That said, inheritance is not evil. It works well when there is a genuine "is-a" relationship and the superclass is explicitly designed for extension. The problems start when inheritance is used for code reuse alone, without a real type hierarchy behind it.

### When Inheritance Is the Right Choice

The **template method pattern** is one place where inheritance shines. The superclass defines the skeleton of an algorithm, leaving specific steps for subclasses to fill in. The JDK's `AbstractList` is a good example: it implements almost all of the `List` interface for you, requiring only that you override `get(int index)` and `size()`:

```java
class ReadOnlyNumberList extends AbstractList<Integer> {
    private final int[] data;

    ReadOnlyNumberList(int[] data) {
        this.data = data.clone();
    }

    @Override
    public Integer get(int index) {
        return data[index];
    }

    @Override
    public int size() {
        return data.length;
    }
}
```

This is inheritance used well. `AbstractList` was explicitly designed for extension, its documentation tells you exactly which methods to override, and the "is-a" relationship is genuine: your class truly *is* a `List`.

Other cases where inheritance makes sense:

- **Framework hook points** designed for subclassing, like `HttpServlet` where you override `doGet()` or `doPost()`
- **Sealed hierarchies** where you control both the superclass and all subclasses (e.g., an AST node type with a fixed set of variants)
- **Shared identity**, not just shared behavior. If external code needs to treat your class as an instance of the parent type, and that relationship is stable, inheritance is natural

A practical rule of thumb: if you catch yourself extending a class just to borrow a few methods, stop and compose instead. Create an interface, inject the dependency, and keep your options open.

### References

* Erich Gamma et al., *Design Patterns: Elements of Reusable Object-Oriented Software* (Addison-Wesley, 1994)
* Joshua Bloch, *Effective Java* (Addison-Wesley, 3rd edition, 2018), Item 18: "Favor composition over inheritance"

> **Related posts**: [Core concepts behind OOP](/posts/core-concepts-behind-oop/), [SOLID: The First 5 Principles of Object Oriented Design](/posts/solid-the-first-5-principles-of-object-oriented-design/), [Composition, Aggregation and Association](/posts/association-composition-aggregation/)

