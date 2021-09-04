---
title: "Comparable and Comparator Interfaces"
categories:
  - Blog
tags:
  - Comparator
  - Comparable
  - Sorting
---

### Comparable.

Both interfaces are used to order the objects of the user-defined class.
Compmarable interface is found in java.lang package and contains only one method named compareTo(Object).<br>
**It provides a single sorting sequence only**, i.e., you can sort the elements on the basis of single data member only.
For example, it may be name, age or anything else.

**compareTo(Object obj) method**

public int compareTo(Object obj): It is used to compare the current object with the specified object. It returns

positive integer, if the current object is greater than the specified object. 

negative integer, if the current object is less than the specified object.

zero, if the current object is equal to the specified object.


**Code Example**

We need to implements Comparable interface first and override compareTo method.

![img]({{site.url}}/assets/blog_images/2021-09-04-comparable-and-comparator-interfaces/comparable1.png)

or simpler:

![img]({{site.url}}/assets/blog_images/2021-09-04-comparable-and-comparator-interfaces/comparable2.png)

<p>&nbsp;</p>

![img]({{site.url}}/assets/blog_images/2021-09-04-comparable-and-comparator-interfaces/comparable3.png)

![img]({{site.url}}/assets/blog_images/2021-09-04-comparable-and-comparator-interfaces/comparable4.png)


### Comparator.

Unlike Comparable, Comparator is external to the element type we are comparing. It’s a separate class. We create multiple separate classes (that implement Comparator) to compare by different members.
Collections class has a second sort() method and it takes Comparator which could be found in java.util package. 
The sort() method invokes the compare() to sort objects.
To compare Student by age, we need to do 3 things :

1) Create a class that implements Comparator (and thus the compare() method that does the work previously done by compareTo()).

2) Make an instance of the Comparator class.

3) Call the overloaded sort() method, giving it both the list and the instance of the class that implements Comparator.<br> 
Or you may call sort() method directly on Collection that you want to sort and give it Comparator as an argument.

**Code Example**

![img]({{site.url}}/assets/blog_images/2021-09-04-comparable-and-comparator-interfaces/comparator1.png)

<p>&nbsp;</p>

![img]({{site.url}}/assets/blog_images/2021-09-04-comparable-and-comparator-interfaces/comparator2.png)

![img]({{site.url}}/assets/blog_images/2021-09-04-comparable-and-comparator-interfaces/comparator3.png)

<p>&nbsp;</p>

**Key differences between Comparable vs Comparator in Java.**

 <table style="width:100%">
  <tr>
    <th>Comparable</th>
    <th>Comparator</th>
  </tr>
  <tr>
    <td>It provides single sorting sequences.</td>
    <td>It provides multiple sorting sequences.
</td>
  </tr>
  <tr>
    <td>It affects the original class. i.e., actual class is altered.</td>
    <td>It doesn’t affect the original class, i.e., actual class is not altered.</td>
  </tr>
  <tr>
    <td>This method can sort the data according to the natural sorting order.</td>
    <td>This method sorts the data according to the customized sorting order.</td>
  </tr>
  <tr>
    <td>The class whose objects you want to sort must implement comparable interface.</td>
    <td>Class, whose objects you want to sort, do not need to implement a comparator interface.</td>
  </tr>  
  <tr>
    <td>The logic of sorting must be in the same class whose object you are going to sort.</td>
    <td>The logic of sorting should be in separate class to write different sorting based on different attributes of objects.</td>
  </tr>
  <tr>
    <td>Comparable interface is present in java.lang package.</td>
    <td>Comparator interface is present in java.util package.</td>
  </tr>
  <tr>
    <td>Comparable provides compareTo() method to sort elements in Java.</td>
    <td>Comparator provides compare() method to sort elements in Java.</td>
  </tr>
  <tr>
    <td>Implemented frequently in the API by: Calendar, Wrapper classes, Date, and String.</td>
    <td>It is implemented to sort instances of third-party classes.</td>
  </tr>   
  <tr>
    <td>All wrapper classes and String class implement comparable interface.</td>
    <td>The only implemented classes of Comparator are Collator and RuleBasedCollator.</td>
  </tr>
</table> 