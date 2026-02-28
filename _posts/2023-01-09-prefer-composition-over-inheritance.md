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

# Prefer Composition over Inheritance

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

Now, let's look at an example involving inheritance. Consider a class hierarchy representing different types of vehicles:

```java
class Vehicle {
    void start() {
        // Implementation for starting a generic vehicle
    }
}

class Car extends Vehicle {
    void drive() {
        start();
        // Additional car-specific functionality
    }
}

class Bicycle extends Vehicle {
    void pedal() {
        start();
        // Additional bicycle-specific functionality
    }
}
```

In this example, the Car and Bicycle classes inherit the start method from the Vehicle class. While this demonstrates inheritance, it's essential to note that changes to the start method in the Vehicle class can impact all subclasses, potentially leading to maintenance challenges.

## Violation of Encapsulation

Subclasses rely on the implementation details of their superclasses for correct operation. The implementation within a superclass may vary across different releases, potentially leading to a breakdown in the subclass, despite no changes to its own code.
Consequently, a subclass necessitates co-evolution with its superclass, unless the superclass has been clearly designed and documented by its authors for the purpose of extension.

## When to Use Each

Composition is the safer default. It keeps your classes small, loosely coupled, and easy to test. When you depend on interfaces and inject collaborators through constructors, you get flexibility that inheritance simply cannot match.

That said, inheritance is not evil. It works well when there is a genuine "is-a" relationship and the superclass is explicitly designed for extension (think `AbstractList` in the JDK). The problems start when inheritance is used for code reuse alone, without a real type hierarchy behind it.

A practical rule of thumb: if you catch yourself extending a class just to borrow a few methods, stop and compose instead. Create an interface, inject the dependency, and keep your options open.

