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

The observer pattern is a software design pattern in which an object, called the subject, maintains a list of its dependents, known as observers, and notifies them automatically of any state changes, usually by calling one of their methods.

**Problem**

Imagine you have two types of objects: Subscriber and Newsletter. A user is very interested in a new email article from a particular blog that should be added soon. The user could visit the email inbox every day and check the article's availability. However, while the article is still not sent, most of these checks would be pointless.

On the other hand, the blog could send tons of emails to all its users each time a new post becomes available. This would save time for some users but would upset others who aren't interested in the new post.

Let's see how I can implement it. First, let's define the Newsletter class:

```java
public class Newsletter {

    private List<EmailArticle> emails;
    private final List<Subscriber> subscribers = new ArrayList<>();

    public void addObserver(Subscriber subscriber) {
        this.subscribers.add(subscriber);
    }

    public void removeObserver(Channel channel) {
        this.subscribers.remove(channel);
    }

    public void setEmails(List<EmailArticle> emails) {
        this.emails = emails;

        for (Subscriber subscriber : this.subscribers) {
            subscriber.update(this.emails);
        }
    }
}
```

The `Newsletter` class acts as the **subject** (observable). It maintains a `List<Subscriber>` and provides `addObserver` / `removeObserver` methods to manage that list. When `setEmails` is called, it iterates through all registered subscribers and invokes their `update()` method, pushing the new email articles to each one.

Now let's see what the observer, the `GmailUser` class, can look like. It should have the `update()` method, which is invoked when the state of Newsletter changes:

```java
public class GmailUser implements Subscriber {

    private List<EmailArticle> emails;

    @Override
    public void update(Collection collection) {
        this.setEmails((List<EmailArticle>) collection);
    }

    public void setEmails(List<EmailArticle> emails) {
        this.emails = emails;
    }
}
```

The `GmailUser` implements the `Subscriber` interface. When notified, it casts the incoming collection to `List<EmailArticle>` and stores it locally.

The Subscriber interface has only one method:

```java
public interface Subscriber {

    public void update(Collection o);
}
```

The `Subscriber` interface declares a single `update(Collection o)` method. Any class that wants to receive notifications from a subject must implement this contract.

**Example**

```java
Newsletter observable  = new Newsletter();
GmailUser observer = new GmailUser();

observable.addObserver(observer);
```

The Newsletter is an observable, and when news gets updated, the state of Newsletter changes. When the change happens, the Newsletter notifies the observers about this fact by calling their `update()` method. To be able to do that, the observable object needs to keep references to the observers, and in our case, it's the `subscribers` variable.

**Java's Built-in Support and Modern Alternatives**

Java historically provided `java.util.Observable` and `java.util.Observer` out of the box, but these were deprecated in Java 9. The `Observable` class had significant limitations: it was a class rather than an interface (preventing use with inheritance hierarchies), and it made the `setChanged()` method protected, restricting who could trigger notifications.

Today, several modern alternatives handle the same publish-subscribe need:

* **`PropertyChangeListener`** (java.beans) -- still in the JDK, works well for simple property change notifications.
* **Event buses** (e.g., Guava's `EventBus`, Spring's `ApplicationEventPublisher`) -- decouple publishers and subscribers through a central dispatcher, removing the need for direct references between them.
* **Reactive Streams** (Project Reactor, RxJava) -- take the observer concept further by adding backpressure, composable operators, and asynchronous scheduling. If I am building anything with high-throughput event flows, this is the direction I reach for.

**Pros**

* Open/Closed Principle: I can introduce new subscriber classes without having to change the publisher's code (and vice versa if there's a publisher interface).
<br>
* I can establish relations between objects at runtime.
<br>

**Cons**

* Subscribers are notified in an unspecified, implementation-dependent order. If my logic depends on a particular notification sequence, the observer pattern will create subtle bugs.
<br>
* **Memory leaks.** If I register an observer and forget to unregister it when it is no longer needed, the subject keeps a strong reference to it, preventing garbage collection. In long-running applications this can quietly consume memory over time. Weak references or explicit lifecycle management (e.g., `close()` / `dispose()`) help mitigate this.
<br>
* **Debugging difficulty.** Because the flow of control jumps from the subject to potentially many observers, tracing the execution path through a debugger becomes harder. Stack traces span across the subject-observer boundary, and the order of side effects can be non-obvious.
<br>
* **Risk of infinite notification loops.** If an observer modifies the subject's state in its `update()` method, the subject may fire another round of notifications, leading to infinite recursion. I always guard against this by either preventing state changes inside `update()` or using a flag to detect re-entrant calls.
