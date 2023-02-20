---
title: "Is Java Pass by Value or by Reference?"
categories:
  - Blog
tags:
  - Java
  - Memory
---

Let me answer that question right away, Java is ALWAYS pass by value.
<br>

The reason for the confusion about these terms is typically related to the concept of object reference in Java. Technically speaking, Java always uses pass-by-value, because although a variable can hold a reference to an object, that reference is actually a value that indicates the object's memory location. Therefore, object references are passed by value in Java.
<br>

It may sound a bit confusing but let's take a look on the example.

```java

class Point {
    int x;
    int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }
}
```
```java
class Test {
    public static void main(String[] args) {
        Point point = new Point(2, 2);
        changeXToGivenValue(3, 3, point);
        System.out.println(String.format("X: {}", point.x));
    }
    
    public static void changePointCoordinatesTo(int x, Point givenPoint){
        givenPoint.x = x;
    }
}
```

How do you think, what will be the output? 
```console
 X: 3
```

Since Java is passed by value and the value of this 'point' variable is the address of a certain object in memory, what it does is copies that value, which is memory address.
Because it copies that value, memory address, 'givenPoint' variable ends up pointing to the exact same Point object in memory as 'point' variable.
That's why Java sometimes may look like it's pass by reference, because it seems like we were passing reference to that object but what it's actually doing 
is passing in the value of the memory address of that object.

![img]({{site.url}}/assets/blog_images/2023-02-20-is-java-pass-by-value-or-by-reference/value-reference-1.jpg)

<br> 

But what would happen in the example below? 

```java
class Test {
    public static void main(String[] args) {
        Point point = new Point(2, 2);
        changeXToGivenValue(5, point);
        System.out.println(String.format("X: {}", point.x));
    }
    
    public static void changeXToGivenValue(int x, Point givenPoint){
        givenPoint = new Point(0, 0);
        givenPoint.x = x;
    }
}
```

Based on the previous example you may think that the output of the example above will be 5... but actually it will not. 

```console
 X: 2
```

The way the variable is passed in is still exactly the same as in the previous example. It's still passing value of the address in memory. 

```console
givenPoint = new Point(0, 0);
```

The key difference is that we're setting 'givenPoint' variable to be a brand-new Point object. 
So when the method starts 'givenPoint' variable has the exactly same address in the memory as 'point' variable, but when we 
reassign to this variable a new object, it starts pointing to a new object in memory, meaning the variable will have different address value.

![img]({{site.url}}/assets/blog_images/2023-02-20-is-java-pass-by-value-or-by-reference/value-reference-2.jpg)

<br>

In case of the primitive types, variables does not store the memory reference, but the values itself, so what it does is copying the given value.  


