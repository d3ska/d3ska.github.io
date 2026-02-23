---
title: "Is Java Pass by Value or by Reference?"
categories:
  - Blog
tags:
  - Java
  - Memory
---

Let me answer that question right away, Java is **ALWAYS** pass by value.
<br>

The reason for the confusion about these terms is typically related to the concept of object reference in Java. Technically speaking, Java always uses pass-by-value, because although a variable can hold a reference to an object, that reference is actually a value that indicates the object's memory location. Therefore, object references are passed by value in Java.
<br>

It may sound a bit confusing, but let's take a look at the example.

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
        changePointCoordinatesTo(3, point);
        System.out.println(String.format("X: %d", point.x));
    }
    
    public static void changePointCoordinatesTo(int x, Point givenPoint){
        givenPoint.x = x;
    }
}
```

What do you think the output will be?
```console
 X: 3
```

Since Java uses pass-by-value and the value of this 'point' variable is the address of a certain object in memory, what it does is copy that value, which is the memory address.
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
        System.out.println(String.format("X: %d", point.x));
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
So when the method starts, the 'givenPoint' variable has exactly the same address in memory as the 'point' variable. But when we reassign a new object to this variable, it starts pointing to a new object in memory, meaning the variable will have a different address value.

![img]({{site.url}}/assets/blog_images/2023-02-20-is-java-pass-by-value-or-by-reference/value-reference-2.jpg)

<br>

In the case of primitive types, variables do not store a memory reference but the values themselves, so what happens is simply copying the given value.

#### What About Arrays?

The same pass-by-value rule applies to arrays. When you pass an array to a method, Java copies the reference to the array -- not the array itself. This means the method receives its own copy of the reference, but that reference still points to the same array object in memory. Any modifications to the array's elements inside the method will be visible to the caller:

```java
public static void modifyArray(int[] arr) {
    arr[0] = 99;
}

public static void main(String[] args) {
    int[] numbers = {1, 2, 3};
    modifyArray(numbers);
    System.out.println(numbers[0]); // 99
}
```

However, reassigning the parameter to a new array inside the method will not affect the original reference, just like with any other object reference.


