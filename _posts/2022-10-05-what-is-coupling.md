---
title: "What is coupling?"
categories:
  - Software Design
tags:
  - Coupling
  - Modularity
  - Software Architecture
---

In the object-oriented paradigm, coupling refers to the degree of direct knowledge that one element has of another. We generally divide coupling into **tight** and **loose**.

We can refer to monolith architecture as an example: a classic monolith is typically tightly coupled, whereas a modular monolith is loosely coupled (assuming the modules are separated correctly).
But let's think about a less abstract example -- a phone and a remote control. What would you do if the battery in your new iPhone dies? You would probably buy a new phone because replacing the battery is difficult and expensive.
On the other hand, what would you do if your TV remote stopped working? You would simply get new batteries and replace them in seconds.

The phone represents tight coupling -- the battery is sealed inside, inseparable from the device. The remote represents loose coupling -- the batteries are an interchangeable component behind a simple interface (the battery compartment). Well-designed software components behave like the remote: you can swap out their dependencies without dismantling the whole system.

![img]({{site.url}}/assets/blog_images/2022-10-05-what-is-coupling/coupling-ilustration-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2022-10-05-what-is-coupling/coupling-ilustration-dark.png){: .dark }

##### Why is coupling bad most of the time?

* It makes refactoring or any change to *how* we implement something more difficult.
* We become **coupled** to the concrete implementation of *what* our system is doing.
* A change in one place can cause a bug in another place that should be completely independent.
* It makes testing and debugging harder.
* Introducing a new feature will take much longer.
* Code gets duplicated because its parts are hard to reuse.

##### Afferent and Efferent Coupling

Two useful metrics for understanding coupling in a codebase are **afferent coupling (Ca)** and **efferent coupling (Ce)**:

- **Afferent coupling (Ca)** measures how many other modules depend *on* a given module. A high Ca means many consumers rely on this module -- changing it carries high risk.
- **Efferent coupling (Ce)** measures how many other modules a given module depends *on*. A high Ce means the module has many dependencies -- it is fragile because any of those dependencies can force a change.

A module with high Ca and low Ce is stable and widely depended upon (think of a core utility library). A module with low Ca and high Ce is unstable and sensitive to change (think of a high-level orchestration service). Tracking these metrics helps you identify where coupling is concentrated and where refactoring will have the greatest impact.

##### Temporal Coupling

There is another form of coupling worth mentioning: **temporal coupling**. This occurs when two operations must happen in a specific order for the system to work correctly, but that ordering constraint is implicit rather than enforced by the API.

For example, if you must call `init()` before `process()`, and the compiler does not prevent you from calling `process()` first, you have temporal coupling. This is a common source of subtle bugs. The solution is to design APIs so that illegal call sequences become impossible -- for instance, by having `init()` return the object that exposes `process()`.

##### Data Coupling

Another common form of coupling appears when modules communicate by sharing data structures. If two services both read from and write to the same database table, or if multiple consumers depend on the shape of an API response, they are **data-coupled**. The danger is that changing the structure in one place (renaming a column, removing a field) silently breaks every dependent module. This is especially prevalent in systems that share database schemas, Protobuf definitions, or API contracts across team boundaries. The mitigation is to treat shared data structures as public APIs: version them, evolve them carefully, and avoid exposing internal representations.

##### Connascence: A Richer Vocabulary for Coupling

If you want a more precise way to talk about coupling, look into **connascence**. Connascence describes the different ways two components can be coupled, ranked by strength. **Connascence of name** is the weakest form: two components must agree on a name (a method name, a queue name). **Connascence of type** means they must agree on a data type. At the stronger end, **connascence of identity** means two components must reference the exact same object instance. The general rule is the same as with coupling: prefer weaker forms over stronger ones, and limit the scope of any connascence to the smallest boundary possible.

##### A Different Look at Coupling

But tight and loose coupling -- what does that mean in the context of our code?
I gained a much better understanding of this topic after watching a conference talk by Lukasz Szydlo,
which can be found [here](https://www.youtube.com/watch?v=Jy6eS9QHJOM).


Let's briefly walk through that concept.
We can separate coupling in our code into four levels, ranging from tightly coupled to loosely coupled. We can
determine which level our code is at by answering the following questions in the context of the piece of code we
are working on:

**How** is it done?  
**Where** does it take place?  
**Who** is doing it?  
**What** happened?

We should strive for a situation where our part of the system cannot answer at least three of those questions (How? Where? Who?).


| | Local method | Local instance | External instance | Configurable instance (DI) | Notification |
|---|:---:|:---:|:---:|:---:|:---:|
| How? | **V** | X | X | X | X |
| Where? | **V** | **V** | X | X | X |
| Who? | **V** | **V** | **V** | X | X |
| What? | **V** | **V** | **V** | **V** | X |

###### Local method
```java
public class SaleService {

    void sendNotification(Notification notification){
        //...
    }
}
```
This is the highest level of coupling -- the local method. SaleService knows everything about the other operation.  
**HOW** - implementation of the method  
**WHERE** - in my instance  
**WHO** - 'me' - EmailSender  
**WHAT** - email will be sent

###### Local instance
```java
public class SaleService {

    private ConcreteSenderImpl concreteSender = new ConcreteSenderImpl();

    void sendNotification(PurchaseDetails purchaseDetails){
        concreteSender.send(purchaseDetails);
    }
}
```

~~**HOW** - implementation of the method~~  
**WHERE** - in my instance  
**WHO** - 'me' - EmailSender  
**WHAT** - email will be sent

We are not aware of how it will be done, since it is handled by a different class.

###### External instance
```java
public class SaleService {

    private final ConcreteSenderImpl concreteSender;

    public SaleService(ConcreteSenderImpl concreteSender){
        this.concreteSender = concreteSender;
    }

    void sendNotification(PurchaseDetails purchaseDetails){
        concreteSender.send(purchaseDetails);
    }
}
```

~~**HOW** - implementation of the method~~  
~~**WHERE** - in my instance~~  
**WHO** - 'me' - EmailSender  
**WHAT** - email will be sent

We decreased coupling further, and "where" is also gone. We no longer contain the object itself -- we hold only a reference to it. It lives somewhere, but we don't know where.

###### Configurable instance (Dependency Injection)
```java
public class SaleService {

    private final EmailApiService emailSender;

    public SaleService(EmailApiService emailSender){
        this.emailSender = emailSender;
    }

    void sendNotification(PurchaseDetails purchaseDetails){
        emailSender.send(purchaseDetails);
    }
}
```

~~**HOW** - implementation of the method~~  
~~**WHERE** - in my instance~~  
~~**WHO** - 'me' - EmailSender~~  
**WHAT** - email will be sent

This example also uses dependency injection, but the key difference is that we are injecting an interface rather than a concrete implementation. Because of that, we eliminate the knowledge of *who* will perform the sending.
It might be this library or another, or we might be dispatching an event to an external service. We know an email will be sent, but we don't know by whom.
This level of coupling is the most common, and in most cases it is perfectly acceptable.
But we can make it even weaker...

###### Notification
```java
public class SaleService {

    private final EventDelegator eventDelegator;

    public SaleService(EventDelegator eventDelegator){
        this.eventDelegator = eventDelegator;
    }

    void sendNotification(PurchaseDetails purchaseDetails){
        eventDelegator.sendEvent(new ClientBoughtItem(purchaseDetails));
    }
}
```

~~**HOW** - implementation of the method~~  
~~**WHERE** - in my instance~~  
~~**WHO** - 'me' - EmailSender~~  
~~**WHAT** - email will be sent~~

With notification, we are not telling the system what to do -- we are telling it what happened.
We have no awareness of what the reaction to our event will be.
Perhaps the client will receive an SMS, or an email, or a push notification, or all of them.
We simply announce what happened and let the subscribers decide how to handle it.

> **Related posts**: [Cohesion](/posts/cohesion/), [SOLID: The First 5 Principles of Object Oriented Design](/posts/solid-the-first-5-principles-of-object-oriented-design/)
