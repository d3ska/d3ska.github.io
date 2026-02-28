---
title: "Observer Design Pattern"
categories:
  - Design Patterns
tags:
  - Design Patterns
  - Observer Pattern
  - Behavioral Patterns
  - SOLID
---

Business objects often need to react when something else changes. A newsletter publishes new articles and multiple mail clients need to know about it. An order changes status and the inventory, billing, and notification services all care. When these dependencies are wired together directly, every new consumer means modifying the producer. The **Observer pattern** breaks this coupling by letting an object notify an open-ended set of dependents without knowing who they are.

### The Problem: Direct Notification

Imagine a `Newsletter` class that sends email articles to users. A first attempt might call each subscriber type directly:

```java
public class Newsletter {

    private List<EmailArticle> emails;
    private final GmailUser gmailUser;
    private final OutlookUser outlookUser;

    public Newsletter(GmailUser gmailUser, OutlookUser outlookUser) {
        this.gmailUser = gmailUser;
        this.outlookUser = outlookUser;
    }

    public void setEmails(List<EmailArticle> emails) {
        this.emails = emails;
        gmailUser.receive(this.emails);
        outlookUser.receive(this.emails);
    }
}
```

This works for two subscribers, but adding a `YahooUser` means modifying `Newsletter`. The producer is tightly coupled to every consumer, violating the Open-Closed Principle. You also cannot add or remove subscribers at runtime.

### The Pattern

The Observer pattern defines two roles. An **Observer** declares the callback that gets invoked on state changes. An **EventManager** maintains a list of observers and handles registration and notification. The concrete publisher does not inherit from anything. Instead, it *composes* an `EventManager` and delegates subscription management to it.

```java
public interface Observer<T> {

    void update(T event);
}
```

```java
public class EventManager<T> {

    private final List<Observer<T>> observers = new ArrayList<>();

    public void subscribe(Observer<T> observer) {
        observers.add(observer);
    }

    public void unsubscribe(Observer<T> observer) {
        observers.remove(observer);
    }

    public void notifyObservers(T event) {
        for (Observer<T> observer : observers) {
            observer.update(event);
        }
    }
}
```

The generic parameter `T` represents the event payload. Observers register with an `EventManager`, and when `notifyObservers` is called, every registered observer receives the event. The `EventManager` is a standalone, reusable component. The concrete publisher composes it rather than extending it, which avoids locking the publisher into a particular inheritance hierarchy.

### Applying It: Newsletter and Subscribers

With the generic building blocks in place, the newsletter problem becomes straightforward. The `Newsletter` holds an `EventManager` as a field and delegates subscription methods to it:

```java
public class Newsletter {

    private final EventManager<List<EmailArticle>> events = new EventManager<>();
    private List<EmailArticle> emails;

    public void subscribe(Observer<List<EmailArticle>> observer) {
        events.subscribe(observer);
    }

    public void unsubscribe(Observer<List<EmailArticle>> observer) {
        events.unsubscribe(observer);
    }

    public void setEmails(List<EmailArticle> emails) {
        this.emails = emails;
        events.notifyObservers(this.emails);
    }
}
```

Each mail client implements `Observer` with the matching type:

```java
public class GmailUser implements Observer<List<EmailArticle>> {

    private List<EmailArticle> emails;

    @Override
    public void update(List<EmailArticle> emails) {
        this.emails = new ArrayList<>(emails);
    }
}
```

The `Newsletter` no longer knows about `GmailUser`, `OutlookUser`, or any other concrete subscriber. Adding a new subscriber type means creating a new class that implements `Observer<List<EmailArticle>>`. The `Newsletter` never changes.

```java
Newsletter newsletter = new Newsletter();
GmailUser gmail = new GmailUser();

newsletter.subscribe(gmail);
```

Observers can be added or removed at runtime, which makes the relationship fully dynamic. When `setEmails` is called, the `Newsletter` delegates to the `EventManager`, which iterates through all registered observers and pushes the update.

This is the **push model**: the publisher sends the event data directly to observers as a method argument. The alternative is the **pull model**, where the publisher passes a reference to itself and observers query for the data they need. The push model is simpler and more common for cases where all observers need the same data.

### Java's Built-in Support and Modern Alternatives

Java historically provided `java.util.Observable` and `java.util.Observer` out of the box, but these were deprecated in Java 9. The `Observable` class had significant limitations: it was a class rather than an interface (preventing use with inheritance hierarchies), and it made the `setChanged()` method protected, restricting who could trigger notifications.

Today, several modern alternatives handle the same publish-subscribe need:

* **`PropertyChangeListener`** (java.beans): still in the JDK, works well for simple property change notifications.
* **Event buses** (e.g., Guava's `EventBus`, Spring's `ApplicationEventPublisher`): decouple publishers and subscribers through a central dispatcher, removing the need for direct references between them. In distributed systems, message brokers like Apache Kafka and RabbitMQ apply the same publish-subscribe idea across service boundaries, with durable messaging, partitioning, and delivery guarantees on top.
* **Reactive Streams** (Project Reactor, RxJava): take the observer concept further by adding backpressure, composable operators, and asynchronous scheduling. If I am building anything with high-throughput event flows, this is the direction I reach for.

### Pros

* **Open/Closed Principle.** New subscriber types can be introduced without changing the publisher's code.
* **Runtime flexibility.** Observers can register and unregister dynamically, so relationships between objects are established at runtime rather than at compile time.
* **Decoupling.** The publisher depends only on the `Observer` interface, not on concrete subscriber classes. This makes both sides independently testable and replaceable.

### Cons

* **Unspecified notification order.** Subscribers are notified in an implementation-dependent order. If logic depends on a particular sequence, subtle bugs will follow.
* **Debugging difficulty.** The flow of control jumps from the publisher to potentially many observers, making stack traces harder to follow and the order of side effects non-obvious.

The Observer pattern fits naturally when the set of interested parties changes at runtime, when consumers are expected to grow over time, or in event-driven architectures where decoupling producers from consumers is a core requirement. If your scenario involves a fixed, small set of consumers that are unlikely to change, direct method calls are simpler and more debuggable.

### References

* [Observer - Refactoring Guru](https://refactoring.guru/design-patterns/observer)
* [Observer in Java - Refactoring Guru](https://refactoring.guru/design-patterns/observer/java/example)
