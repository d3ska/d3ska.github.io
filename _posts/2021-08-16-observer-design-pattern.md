---
title: "Observer Design Patter"
categories:
  - Blog
tags:
  - Design patterns
  - Observer pattern
  - SOLID
---

The observer pattern is a software design pattern in which an object, called the subject, maintains a list of its dependents, known as observers, and notifies them automatically of any state changes, usually by calling one of their methods.

**Problem**

Imagine you have two types of objects: Subscriber and Newsletter. A user is very interested in a new email article from a particular blog that should be added soon. The user could visit the email inbox every day and check the article's availability. However, while the article is still not sent, most of these checks would be pointless.

On the other hand, the blog could send tons of emails to all its users each time a new post becomes available. This would save time for some users but would upset others who aren't interested in the new post.

Let's see how we can implement it ourselves. First, let's define the Newsletter class:

![img]({{site.url}}/assets/blog_images/2021-08-16-observer-design-pattern/observer1.png)

Now let's see how the observer, the GmailUser class, can look like. It should have the update() method, which is invoked when the state of Newsletter changes:

![img]({{site.url}}/assets/blog_images/2021-08-16-observer-design-pattern/observer2.png)

The Subscriber interface has only one method:

![img]({{site.url}}/assets/blog_images/2021-08-16-observer-design-pattern/observer3.png)

**Example**

![img]({{site.url}}/assets/blog_images/2021-08-16-observer-design-pattern/observer4.png)

The Newsletter is an observable, and when news gets updated, the state of Newsletter changes. When the change happens, the Newsletter notifies the observers about this fact by calling their update() method. To be able to do that, the observable object needs to keep references to the observers, and in our case, it's the users variable.

**Pros**

* Open/Closed Principle: You can introduce new subscriber classes without having to change the publisher's code (and vice versa if there's a publisher interface).
<br>
* You can establish relations between objects at runtime.
<br>

**Cons**

* Subscribers are notified in random order
