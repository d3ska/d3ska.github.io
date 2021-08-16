---
title: "Observer Design Patter"
categories:
  - Blog
tags:
  - Design patterns
  - Observer pattern
  - SOLID
---

### The observer pattern is a software design pattern in which an object, named the subject, maintains a list of its dependents, called observers, and notifies them automatically of any state changes, usually by calling one of their methods.

**Problem**

Imagine that you have two types of objects e.g: Subsriber and Newsletter. User is very interested in a new email article from a particular blog. Which should be added soon, the user could visit the email box every day and check email article availability. But while the article is still not sent, most of these checks would be pointless.

On the other hand, blog could send a tons of emails to all blog users each time new post becomes available. This would save time for some users, but at the same time, it would upset other users who aren't intrested in new post.

Let's see how we can implement it ourselves.
First, let's define the Newsletter class:

![img]({{site.url}}/assets/blog_images/2021-08-16-observer-design-pattern/observer1.png)

Let's now see how the observer, the GmailUser class, can look like. It should have the update() method which is invoked when the state of Newsletter changes:

![img]({{site.url}}/assets/blog_images/2021-08-16-observer-design-pattern/observer2.png)

The Subscriber interface has only one method:

![img]({{site.url}}/assets/blog_images/2021-08-16-observer-design-pattern/observer3.png)

**Example**

![img]({{site.url}}/assets/blog_images/2021-08-16-observer-design-pattern/observer4.png)

Newsletter is an observable, and when news gets updated, the state of Newsletter changes. When the change happens, Newsletter notifies the observers about this fact by calling their update() method.
To be able to do that, the observable object needs to keep references to the observers, and in our case, it's the users variable.

**Pros and Cons**

<span style="color: green"> Open/Closed Principle. You can introduce new subscriber classes without having to change the publisher’s code (and vice versa if there’s a publisher interface). </span>
<p>&nbsp;</p>
<span style="color: green"> You can establish relations between objects at runtime. </span>
<p>&nbsp;</p>
<span style="color: red">  Subscribers are notified in random order. </span>
<<<<<<< HEAD





=======
>>>>>>> 13e75e90ecbe1e7788d1bd796141b4efb8910dbb
