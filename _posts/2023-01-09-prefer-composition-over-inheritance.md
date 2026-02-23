---
title: "Prefer Composition over Inheritance"
categories:
  - Blog
tags:
  - Inheritance
  - OOP
  - Composition
---

# Prefer Composition over Inheritance

You've probably heard this statement before, as it's quite popular. Let's dive deeper into the subject. I've written an article that explains what composition is [here](/posts/association-composition-aggregation/).

## 1. Introduction

Both concepts revolve around reusing pieces of code.

In general, it is widely considered good design to favor composition over inheritance. This preference arises because composition allows for greater flexibility and code reuse. It enables you to combine simple objects to achieve more complex behavior without the need to create intricate inheritance hierarchies.

## 2. Composition vs. Inheritance

Inheritance can be inflexible because it involves creating a subclass that is tightly coupled to its superclass. This means that any changes to the superclass may necessitate changes to the subclass as well. This can become challenging to manage and maintain as the codebase expands. Moreover, inheritance can complicate code reuse because distinguishing between behavior inherited from the superclass and behavior specifically implemented in the subclass can be tricky.

On the other hand, composition empowers you to construct complex behavior by assembling simple objects. This makes it easier to reuse code and make changes to your codebase, as you can readily substitute different objects to attain the desired behavior.

Let's illustrate these concepts with Java examples.

## 3. Examples in Java

### Composition Example

Consider a simple example of composition in Java where we have a `Car` class composed of an `Engine` and `Wheels`:

```java
class Engine {
    void start() {
        // Implementation for starting the engine
    }
}

class Wheels {
    void rotate() {
        // Implementation for rotating the wheels
    }
}

class Car {
    private Engine engine;
    private Wheels wheels;

    public Car() {
        this.engine = new Engine();
        this.wheels = new Wheels();
    }

    void drive() {
        engine.start();
        wheels.rotate();
        // Additional car-specific functionality
    }
}
```

In this example, the Car class uses composition to combine the functionality of an Engine and Wheels to create a fully functional Car.

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

## 3. Caveats with the `equals` Method (New Section)

When using inheritance, especially in complex class hierarchies, you must exercise caution with the `equals` method. This method is essential for comparing objects for equality, but it can be challenging to implement correctly when multiple classes are involved.

The potential caveats with the `equals` method in inheritance hierarchies include:

- **Violation of Transitive Rule:** Inheritance hierarchies can lead to violations of the transitive rule of equality. If not carefully implemented, you might encounter situations where object A equals object B, and object B equals object C, but object A does not equal object C. This can lead to subtle bugs and unexpected behavior in your code.

- **Complexity and Consistency:** Inheritance hierarchies can introduce complexity in determining which attributes and criteria should be considered for equality comparison. Ensuring that the `equals` method is consistent across all classes in the hierarchy can be challenging.

To delve deeper into these issues and learn best practices for implementing the `equals` method in inheritance hierarchies, refer to our comprehensive article on [equals and hashCode](/posts/java-equals-and-hashcode-contract/).


## 4. Violation of encapsulation

Subclasses rely on the implementation details of their superclasses for correct operation. The implementation within a superclass may vary across different releases, potentially leading to a breakdown in the subclass, despite no changes to its own code.
Consequently, a subclass necessitates co-evolution with its superclass, unless the superclass has been clearly designed and documented by its authors for the purpose of extension.

## 5. Recommendations and Conclusion

In conclusion, while composition is generally favored over inheritance for its flexibility and code reuse benefits, there are situations where inheritance can be useful, especially for reducing code duplication in common parent classes. However, always approach inheritance with caution, taking into consideration the potential pitfalls of the `equals` method and other complexities it may introduce.

Remember that design decisions should be made based on the specific needs of your project. Consider the advantages and disadvantages of both composition and inheritance in the context of your application, and strive for a balance that aligns with your project's requirements.

By following best practices and staying informed about potential caveats, you can make informed decisions about when to use inheritance and when to prefer composition in your Java projects.

