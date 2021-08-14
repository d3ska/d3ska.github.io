---
title: "Strategy Design Patter"
categories:
  - Blog 
tags:
  - Design patterns
  - Strategy pattern
  - SOLID
---

#### Strategy is a behavioral design pattern that lets you define a family of algorithms, put each of them into a separate class, and make their objects interchangeable.

It makes it easier for us to solve the same problem in different ways.
A very important element is the intention, which is what we want to do, not how.

However, each strategy is simply a way of doing what we want to accomplish.

For example:

**Problem**: No payment for the order

**Intention**:  The possibility of paying for the order

Now, think for a moment how many ways do you know for payment?

We can pay by using for example:
* Visa Card,
* Mastercard Card,
* Credit Card
* PayPal,
* Cash,
* Cryptocurrencies

From the point of view of the person ordering a given functionality or the fragment of code that is responsible for the payment, it does not matter how it will be implemented, the effect is important, what counts is that the payment will be settled.

### **Implementation**

First of all we will create the interface for our strategy pattern example, in our case to pay the amount passed as argument.

**PayStrategy.java**

![img]({{site.url}}/assets/blog_images/2021-12-08-strategy-design-pattern/strategy1.png)

Now we have to create concrete implementation of algorithms for payment using one of above metho, e.g. PayPal, Credit Card or Cryptocurrencies.

**PayByPayPal.java**

![img]({{site.url}}/assets/blog_images/2021-12-08-strategy-design-pattern/strategy2.png)

**PayByCreditCard.java**

![img]({{site.url}}/assets/blog_images/2021-12-08-strategy-design-pattern/strategy4.png)

**CreditCard.java**

![img]({{site.url}}/assets/blog_images/2021-12-08-strategy-design-pattern/strategy5.png)

**Order.java**

![img]({{site.url}}/assets/blog_images/2021-12-08-strategy-design-pattern/strategy6.png)

Now we can use any strategy that implements our interface, whatsmore in Strategy pattern, a class behavior or its algorithm can be changed at run time. This type of design pattern comes under behavior pattern.
	 