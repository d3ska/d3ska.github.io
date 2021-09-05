---
title: "Java equals() and hashCode() contract"
categories:
  - Blog
tags:
  - equals hashCode contract
  - Java
---


### Equals() method

Is one of the methods available in the Object class.
Due to the fact that every object in Java has this class in its inheritance hierarchy, we can call this method on every object.
In most cases, the default implementation of the equals method is not appropriate, therefore the programmer creating a new object must implement this method if he wants to check if the instances of this class are equal.
There is a set of guidelines that the equals method should meet in order to be properly implemented.<br>
**The equals() method must be:**

* reflexive: an object must equal itself
* symmetric: x.equals(y) must return the same result as y.equals(x)
* transitive: if x.equals(y) and y.equals(z) then also x.equals(z)
* consistent: the value of equals() should change only if a property that is contained in equals() changes (no randomness allowed)



### hashCode() method

As with equals, hashCode is implemented in the Object class. Whenever the programmer implements the hashCode method, he should also implement theequals method.

This method returns a number of the int type, which is used to assign a given object to a group. Thanks to the hashCode method, we are able to divide all possible instances of a given class into separate groups. Each of these groups is represented by a number returned by the hashCode method.

**The hashCode method is used by collections such as HashSet, HashMap to ensure data consistency of these data sets. Without overridden those two methods, those collections wouldnt work properly.
Without replacing these two methods, these collections would not behave properly.**


### The contract between the equals and hashCodePermalink methods

The hashCode and equals methods are related, and their implementation should be consistent. This relationship is defined by the contract between hashCode and equals.

* If X.equals (Y) == true then it is required that X.hashCode () == Y.hashCode (),
* Multiple calls of the hashCode method on the same object, which has not been modified between calls, must return the same value, 
* If X.hashCode () == Y.hashCode () then it is not required that X.equals (Y) == true.


<br>

#### Code Example

![img]({{site.url}}/assets/blog_images/2021-09-05-java-equals()-and-hashCode()-contracts/equals1.png)

![img]({{site.url}}/assets/blog_images/2021-09-05-java-equals()-and-hashCode()-contracts/equals2.png)

![img]({{site.url}}/assets/blog_images/2021-09-05-java-equals()-and-hashCode()-contracts/equals3.png)


Comparing two objects without the equals () method doesn't work as it should.

![img]({{site.url}}/assets/blog_images/2021-09-05-java-equals()-and-hashCode()-contracts/equals4.png)

![img]({{site.url}}/assets/blog_images/2021-09-05-java-equals()-and-hashCode()-contracts/equals5.png)

As you can see, the overridden equals () method was enough to check for equality between objects.
But let's see how the HashSet will behave with these objects.

![img]({{site.url}}/assets/blog_images/2021-09-05-java-equals()-and-hashCode()-contracts/equals6.png)

![img]({{site.url}}/assets/blog_images/2021-09-05-java-equals()-and-hashCode()-contracts/equals7.png)

The HashSet should not add an object if the same object already exists in the collection.
Let's check it again after overriding the hashCode () method.

![img]({{site.url}}/assets/blog_images/2021-09-05-java-equals()-and-hashCode()-contracts/equals8.png)

![img]({{site.url}}/assets/blog_images/2021-09-05-java-equals()-and-hashCode()-contracts/equals9.png)

<br>

**Conclusion**

In order to achieve a fully working custom equality mechanism, it is mandatory to override hashcode() each time you override equals().

* If two objects are equal, they MUST have the same hash code. 
* If two objects have the same hash code, it doesn't mean that they are equal.
* Overriding equals() alone will make your business fail with hashing data structures like: HashSet, HashMap, HashTable ... etc.
* Overriding hashcode() alone doesn't force Java to ignore memory addresses when comparing two objects.


