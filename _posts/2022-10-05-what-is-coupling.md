---
title: "What is coupling?"
categories:
  - Blog
tags:
  - Metrics
  - Design
  - Coupling
---

In the object-oriented paradigm, coupling refers to the degree of direct knowledge that one element has of another. We generally divide coupling into **tight** and **loose**.

<br>
We can refer to monolith architecture as an example: a classic monolith is typically tightly coupled, whereas a modular monolith is loosely coupled (assuming the modules are separated correctly). <br>
But let's think about a less abstract example -- a phone and a remote control. What would you do if the battery in your new iPhone dies? You would probably buy a new phone because replacing the battery is difficult and expensive.
On the other hand, what would you do if your TV remote stopped working? You would simply get new batteries and replace them in seconds.

The phone represents tight coupling -- the battery is sealed inside, inseparable from the device. The remote represents loose coupling -- the batteries are an interchangeable component behind a simple interface (the battery compartment). Well-designed software components behave like the remote: you can swap out their dependencies without dismantling the whole system.

<br>
<br>

![img]({{site.url}}/assets/blog_images/2022-10-05-what-is-coupling/coupling-ilustration.png)

<br>

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

##### A Different Look at Coupling

But tight and loose coupling -- what does that mean in the context of our code? <br>
I gained a much better understanding of this topic after watching a conference talk by Lukasz Szydlo,
which can be found [here](https://www.youtube.com/watch?v=Jy6eS9QHJOM).


Let's briefly walk through that concept. <br>
We can separate coupling in our code into four levels, ranging from tightly coupled to loosely coupled. We can
determine which level our code is at by answering the following questions in the context of the piece of code we
are working on:

**How** is it done? <br>
**Where** does it take place? <br>
**Who** is doing it? <br>
**What** happened? <br>

We should strive for a situation where our part of the system cannot answer at least three of those questions (How? Where? Who?).


<table style="width:100%">
  <caption>Coupling levels</caption>
  <tr>
    <th></th>
    <th>Local method</th>
    <th>Local instance</th>
    <th>External instance</th>
    <th>Configurable instance (DI)</th>
    <th>Notification</th>
  </tr>
  <tr>
    <td>How?</td>
    <td style="text-align:center; color: green">V</td>
    <td style="text-align:center; color: red">X</td>
    <td style="text-align:center; color: red">X</td>
    <td style="text-align:center; color: red">X</td>
    <td style="text-align:center; color: red">X</td>
  </tr>
  <tr>
    <td>Where?</td>
    <td style="text-align:center; color: green">V</td>
    <td style="text-align:center; color: green">V</td>
    <td style="text-align:center; color: red">X</td>
    <td style="text-align:center; color: red">X</td>
    <td style="text-align:center; color: red">X</td>
  </tr>
  <tr>
   <td>Who?</td>
    <td style="text-align:center; color: green">V</td>
    <td style="text-align:center; color: green">V</td>
    <td style="text-align:center; color: green">V</td>
    <td style="text-align:center; color: red">X</td>
    <td style="text-align:center; color: red">X</td>
  </tr>
  <tr>
    <td>What?</td>
    <td style="text-align:center; color: green">V</td>
    <td style="text-align:center; color: green">V</td>
    <td style="text-align:center; color: green">V</td>
    <td style="text-align:center; color: green">V</td>
    <td style="text-align:center; color: red">X</td>
   </tr>
</table>

<br>
<br>

###### Local method
```java
public class SaleService {

    void sendNotification(Notification notification){
        //...
    }
}
```
This is the highest level of coupling -- the local method. SaleService knows everything about the other operation. <br>
**HOW** - implementation of the method <br>
**WHERE** - in my instance <br>
**WHO** - 'me' - EmailSender <br>
**WHAT** - email will be sent


<br>

###### Local instance
```java
public class SaleService {

    private ConcreteSenderImpl concreteSender = new ConcreteSenderImpl();

    void sendNotification(PurchaseDetails purchaseDetails){
        concreteSender.send(purchaseDetails);
    }
}
```

~~**HOW** - implementation of the method~~ <br>
**WHERE** - in my instance <br>
**WHO** - 'me' - EmailSender <br>
**WHAT** - email will be sent

We are not aware of how it will be done, since it is handled by a different class.

<br>

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

~~**HOW** - implementation of the method~~ <br>
~~**WHERE** - in my instance~~ <br>
**WHO** - 'me' - EmailSender <br>
**WHAT** - email will be sent

We decreased coupling further, and "where" is also gone. We no longer contain the object itself -- we hold only a reference to it. It lives somewhere, but we don't know where.

<br>

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

~~**HOW** - implementation of the method~~ <br>
~~**WHERE** - in my instance~~ <br>
~~**WHO** - 'me' - EmailSender~~ <br>
**WHAT** - email will be sent

This example also uses dependency injection, but the key difference is that we are injecting an interface rather than a concrete implementation. Because of that, we eliminate the knowledge of *who* will perform the sending.
It might be this library or another, or we might be dispatching an event to an external service. We know an email will be sent, but we don't know by whom.
This level of coupling is the most common, and in most cases it is perfectly acceptable.
But we can make it even weaker...


<br>

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

~~**HOW** - implementation of the method~~ <br>
~~**WHERE** - in my instance~~ <br>
~~**WHO** - 'me' - EmailSender~~ <br>
~~**WHAT** - email will be sent~~

With notification, we are not telling the system what to do -- we are telling it what happened.
We have no awareness of what the reaction to our event will be.
Perhaps the client will receive an SMS, or an email, or a push notification, or all of them.
We simply announce what happened and let the subscribers decide how to handle it.
