---
title: "SOLID: The First 5 Principles of Object Oriented Design"
categories:
  - Software Design
tags:
  - SOLID
  - OOP
  - Design Principles
  - Clean Code
---

**SOLID** is an acronym for five object-oriented design principles introduced by Robert C. Martin (Uncle Bob). Together they guide developers toward code that is easier to understand, extend, and maintain.

* **S** - Single-Responsibility Principle
* **O** - Open-Closed Principle
* **L** - Liskov Substitution Principle
* **I** - Interface Segregation Principle
* **D** - Dependency Inversion Principle

### Single-Responsibility Principle

The **Single-Responsibility Principle** (SRP) states that a class should have one and only one reason to change, meaning that a class should have only one job. Let's look at a classic example with a simple book class:

```java
public class Book {

    private String name;
    private String author;
    private String text;

    //constructor, getters and setters
}
```

This class stores data about a book. It has one clear reason to change: if the data model of a book changes. Now suppose someone adds printing responsibilities directly into this class.

```java
public class Book {

    private String name;
    private String author;
    private String text;

    void printTextToConsole(String text) {
        //our code for formatting and printing text
    }

    void printTextToAnotherMedium(String text) {
        // code for writing to any other location..
    }
}
```

Now `Book` has two reasons to change: changes to the book data and changes to how we print. If we switch from console output to a REST API response, we have to touch the same class that holds our domain model. That is a violation of SRP.

To fix this, we extract the printing concern into its own class.

```java
public class BookPrinter {

    void printTextToConsole(String text) {
        // our code for formatting and printing text
    }

    void printTextToAnotherMedium(String text) {
        // code for writing to any other location...
    }
}
```

Now `Book` stays focused on book data, and `BookPrinter` handles output. The two change for completely independent reasons and can evolve separately. If we later need to add PDF export, we modify (or extend) `BookPrinter` without touching `Book` at all.

This principle applies broadly: whether it's logging, validating, emailing, or anything else, we should have separate classes dedicated to a single concern. Don't worry about having more classes; this approach makes it easier to read, refactor, and maintain. Trust me, you wouldn't want to work with classes with thousands of lines.

> **Takeaway:** Every class should have exactly one reason to change. If you find yourself modifying a class for two unrelated reasons, split it.

### Open-Closed Principle

The **Open-Closed Principle** (OCP) states that objects or entities should be open for extension but closed for modification.

Think about what happens when a new requirement arrives. If adding that feature means opening up existing, tested classes and editing their internals, every change carries the risk of breaking something that already works. The OCP says we should structure code so that new behavior is added by writing new code, not by modifying old code.

Design patterns like Strategy and Template Method are popular tools for achieving this. Here is a concrete example. Suppose we need a logging system that can send messages to different destinations. We start with an abstraction:

```java
public interface MessageLogger {

    void log(String message) throws Exception;
}
```

Each destination gets its own class that implements this interface. The first one logs messages to the console.

```java
public class ConsoleLogger implements MessageLogger {
    @Override
    public void log(String message) throws Exception {
        System.out.println(message);
    }
}
```

The second one writes to a file.

```java
public class FileLogger implements MessageLogger {
    @Override
    public void log(String message) throws Exception {
        Files.write(Paths.get("file.log"),
                Collections.singletonList(message));
    }
}
```

If we need to log to a database tomorrow, we add a new `DatabaseLogger` class that implements `MessageLogger`. The existing `ConsoleLogger` and `FileLogger` remain untouched. Any code that depends on the `MessageLogger` interface works with the new implementation automatically:

```java
MessageLogger logger = new ConsoleLogger();
logger.log("Application started");

// Later, switch to file logging without changing any calling code
logger = new FileLogger();
logger.log("Application started");
```

> **Takeaway:** Design your modules so that adding new behavior means writing new code (a new class or implementation), never editing existing, tested code.

### Liskov Substitution Principle

The **Liskov Substitution Principle** (LSP) is often the least intuitive of the five, but it is arguably the most important for writing reliable polymorphic code.

Robert C. Martin summarizes it: subtypes must be substitutable for their base types.

Barbara Liskov introduced the concept in a 1987 keynote and later formalized it with Jeannette Wing in 1994: if for each object o1 of type S there is an object o2 of type T such that for all programs P defined in terms of T, the behavior of P is unchanged when o1 is substituted for o2, then S is a subtype of T.

In plain terms: if your code works with a base type, it should continue to work correctly with any subtype, no surprises, no special cases.

#### Why It Matters

Consider what happens when LSP is violated. A caller writes code against a base class, trusts its contract, and then at runtime receives a subclass that behaves differently. The result is defensive `instanceof` checks scattered throughout the codebase, or worse, silent bugs.

Let's look at a **bad example**.

```java
public class Bird {
    public void fly() {}
}

class Hawk extends Bird {}
```

The hawk can fly because it is a bird, but what about this:

```java
class Ostrich extends Bird {}
```

An ostrich is a bird, but it cannot fly. The `Ostrich` class is a subtype of `Bird`, but it should not be able to use the `fly` method. This means we are breaking LSP.

The real problem shows up in the calling code. Imagine a method that accepts any `Bird`:

```java
void makeBirdFly(Bird bird) {
    bird.fly(); // What happens when bird is an Ostrich?
}
```

This code trusts the contract that every `Bird` can `fly()`. When an `Ostrich` is passed in, we have a few bad options: throw an exception (surprising the caller), silently do nothing (hiding a logic error), or force the caller to check `instanceof` before calling `fly()`. All of these defeat the purpose of polymorphism.

Now let's look at a **good example**.

```java
public class Bird {}

class FlyingBird extends Bird {
    public void fly() {}
}

class Hawk extends FlyingBird {}
class Ostrich extends Bird {}
```

Why is this better? Because the `fly()` method now only exists on `FlyingBird`, and every subclass of `FlyingBird` genuinely can fly. A method that needs flying behavior asks for a `FlyingBird`, not a generic `Bird`:

```java
void makeBirdFly(FlyingBird bird) {
    bird.fly(); // Safe: every FlyingBird honors this contract
}
```

The compiler itself prevents us from passing an `Ostrich` to `makeBirdFly()`. We have moved the constraint from runtime (where it causes bugs) to compile time (where it causes a build error). That is the payoff of respecting LSP: the type system works for you rather than against you.

> **Takeaway:** Any subclass must honor the contract of its parent. Callers should never need to know which subclass they are working with. If substituting a subclass changes behavior in unexpected ways, the hierarchy needs rethinking.

### Interface Segregation Principle

The **Interface Segregation Principle** (ISP) states that no client should be forced to depend on methods it does not use. This problem typically arises when a single interface grows to cover too many responsibilities, and implementors end up with empty method bodies or `UnsupportedOperationException` throws just to satisfy the compiler.

Consider an `Athlete` interface that tries to capture everything an athlete might do:

```java
public interface Athlete {

    void compete();

    void swim();

    void highJump();

    void longJump();
}
```

This interface bundles together competing, swimming, and jumping. Suppose John Smith is a professional swimmer. By implementing the `Athlete` interface, we are forced to implement methods that John will never use, like high jump and long jump. Notice the empty method bodies:

```java
public class JohnSmith implements Athlete {
    @Override
    public void compete() {
        System.out.println("John Smith started competing");
    }

    @Override
    public void swim() {
        System.out.println("John Smith started swimming");
    }

    @Override
    public void highJump() {
    }

    @Override
    public void longJump() {
    }
}
```

Those empty `highJump()` and `longJump()` methods are a code smell. They signal that the interface is too broad. To fix this, we split the original interface along behavioral lines:

```java
public interface Athlete {

    void compete();
}
```

Then we will create two other interfaces, one for jumping athletes and one for swimming athletes.

```java
public interface SwimmingAthlete extends Athlete {

    void swim();
}
```

```java
public interface JumpingAthlete extends Athlete {

    void highJump();

    void longJump();
}
```

Now John Smith only implements the interface that matches his abilities, with no dead code:

```java
public class JohnSmith implements SwimmingAthlete {

    @Override
    public void compete() {
        System.out.println("John Smith started competing");
    }

    @Override
    public void swim() {
        System.out.println("John Smith started swimming");
    }
}
```

> **Takeaway:** Prefer many small, focused interfaces over one large "god" interface. Clients should only see the methods they actually need.

### Dependency Inversion Principle

To understand the motivation behind the DIP, let's start with its formal definition, given by Robert C. Martin in his book *Agile Software Development: Principles, Patterns, and Practices*:

* High-level modules should not depend on low-level modules. Both should depend on abstractions.
* Abstractions should not depend on details. Details should depend on abstractions.

At its core, the DIP is about inverting the classic dependency between high-level and low-level components by abstracting away the interaction between them. In traditional software development, high-level components depend on low-level ones, making it hard to reuse the high-level components.

Let's look at a **bad example**.

Consider an electric switch that turns a light bulb on or off. We can create two classes to make it work. First, the LightBulb class.

```java
public class LightBulb {
    public void turnOn() {
        System.out.println("LightBulb: Bulb turned on...");
    }
    public void turnOff() {
        System.out.println("LightBulb: Bulb turned off...");
    }
}
```

In the class above, we wrote two methods, turnOn() and turnOff().

The second class is ElectricPowerSwitch.

```java
public class ElectricPowerSwitch {

    private LightBulb lightBulb;
    private boolean on;

    public ElectricPowerSwitch(LightBulb lightBulb) {
        this.lightBulb = lightBulb;
        this.on = false;
    }

    public boolean isOn() {
        return this.on;
    }

    public void press() {
        boolean checkOn = isOn();
        if (checkOn) {
            lightBulb.turnOff();
            this.on = false;
        } else {
            lightBulb.turnOn();
            this.on = true;
        }
    }
}
```

Our switch is now ready to use, to turn the light bulb on and off. But the mistake we made is apparent. Our high-level ElectricPowerSwitch class is directly dependent on the low-level LightBulb class. The LightBulb class is hardcoded in ElectricPowerSwitch. However, a switch should not be tied to a bulb. It should be able to turn on and off other devices too, such as a fan, an AC, or the entire lighting in our backyard.

Now let's change it to conform to the DIP principle. To do that, we need an abstraction that both the ElectricPowerSwitch and LightBulb classes will depend on.

First, create an interface for switches.

```java
public interface Switch {
    boolean isOn();
    void press();
}
```

We wrote an interface for switches with the isOn() and press() methods. This interface will give us the flexibility to plug in other types of switches, say a remote control switch later on, if required.

Next, we will write the abstraction in the form of an interface called Switchable with the turnOn() and turnOff() methods.

```java
public interface Switchable {
    void turnOn();
    void turnOff();
}
```

That interface allows any switchable device in the application to implement it and provide its own functionality. Our ElectricPowerSwitch class will also depend on this interface, as shown below:

```java
public class ElectricPowerSwitch implements Switch {

    private Switchable client;
    private boolean on;

    public ElectricPowerSwitch(Switchable client) {
        this.client = client;
        this.on = false;
    }

    public boolean isOn() {
        return this.on;
    }

    public void press() {
        boolean checkOn = isOn();
        if (checkOn) {
            client.turnOff();
            this.on = false;
        } else {
            client.turnOn();
            this.on = true;
        }
    }
}
```

In the ElectricPowerSwitch class, we implemented the Switch interface and referred to the Switchable interface instead of any specific class in the field. We then called the interface's turnOn() and turnOff() methods, which will be called at runtime on the object passed to the constructor. Now we can add low-level switchable classes without having to worry about modifying the ElectricPowerSwitch class (remember the Open-Closed Principle). Let's add two such classes.

```java
public class LightBulb implements Switchable {
    @Override
    public void turnOn() {
        System.out.println("LightBulb: Bulb turned on...");
    }
    @Override
    public void turnOff() {
        System.out.println("LightBulb: Bulb turned off...");
    }
}
```

```java
public class Fan implements Switchable {
    @Override
    public void turnOn() {
        System.out.println("Fan: Fan turned on...");
    }
    @Override
    public void turnOff() {
        System.out.println("Fan: Fan turned off...");
    }
}
```

Now our code is flexible. The high-level `ElectricPowerSwitch` depends only on the `Switchable` abstraction. We can wire them together like this:

```java
Switchable bulb = new LightBulb();
Switch lightSwitch = new ElectricPowerSwitch(bulb);
lightSwitch.press(); // LightBulb: Bulb turned on...
lightSwitch.press(); // LightBulb: Bulb turned off...

Switchable fan = new Fan();
Switch fanSwitch = new ElectricPowerSwitch(fan);
fanSwitch.press(); // Fan: Fan turned on...
```

The same `ElectricPowerSwitch` class controls both devices. Adding a new device (say, a `GarageDoor`) means creating one new class that implements `Switchable`. The switch, the interface, and every existing device stay untouched.

> **Takeaway:** Depend on abstractions (interfaces), not on concrete implementations. This keeps high-level policy decoupled from low-level details.

In practice, wiring all these abstractions by hand becomes tedious in larger applications. **Dependency Injection** (DI) frameworks such as Spring (Java), Guice (Java), or Dagger (Android/Java) automate this process. They manage object creation and inject the correct implementations at runtime based on configuration or annotations, so you can focus on defining clean interfaces while the framework takes care of the plumbing.

### When SOLID Goes Too Far

SOLID principles are guidelines, not laws. Applied without judgment, they can lead to over-engineering that makes a codebase harder to work with rather than easier.

A few signs that SOLID is being over-applied:

* **Premature abstraction.** Creating an interface for a class that has exactly one implementation and no foreseeable reason to have a second one. This adds indirection without adding value. If you have a `UserRepository` interface backed by a single `PostgresUserRepository`, ask yourself whether there is a realistic second implementation. If the answer is "maybe someday," you probably do not need the interface yet.
* **Explosion of tiny classes.** Taking SRP too literally can scatter related logic across dozens of classes that are hard to navigate. A class with two closely related responsibilities (say, validating and saving a form) might be perfectly fine as a single cohesive unit. SRP is about reasons to change, not about counting methods.
* **Abstraction layers that mirror each other.** When every concrete class has a matching interface, every service has a matching "port," and you need to navigate five files to understand a single operation, the abstractions are not earning their keep.
* **Speculative generality.** Applying OCP by designing extension points that nobody actually needs. Writing a plugin system for something that will only ever have one variant adds complexity today for flexibility that may never be used.

The key question is always: does this abstraction make the code easier to understand, test, or extend right now, or am I adding it "just in case"? I have personally refactored codebases where over-application of SOLID created more problems than it solved, with interfaces that had one implementation, factory factories, and service classes that delegated every call to yet another service class.

A good rule of thumb: apply SOLID when you feel pain (a class is hard to test, a change ripples through many files, a new requirement forces you to modify existing code). Don't apply it preemptively to code that is simple and stable.

### References

* Robert C. Martin, *Agile Software Development: Principles, Patterns, and Practices* (Prentice Hall, 2003)
* Robert C. Martin, [SOLID Relevance](https://blog.cleancoder.com/uncle-bob/2020/10/18/Solid-Relevance.html) (The Clean Code Blog, 2020)
* Barbara Liskov, "Data Abstraction and Hierarchy," SIGPLAN Notices 23, no. 5 (1988)

> **Related posts**: [Inversion of Control and Dependency Injection](/posts/inversion-of-control-and-the-dependency-injection/)
