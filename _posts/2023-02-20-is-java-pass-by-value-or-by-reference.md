---
title: "Is Java Pass by Value or by Reference?"
categories:
  - Java
tags:
  - Java
  - Memory
  - JVM
  - Pass by Value
---

Java is **always** pass by value. No exceptions.

The confusion arises because when you pass an object to a method, what gets passed is the *value* of the reference (the memory address of the object), not the object itself. The method receives its own copy of that reference. Both the original variable and the parameter point to the same object in memory, which makes it look like pass by reference, but it is not.

### Modifying an Object Through a Copied Reference

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
        changeX(3, point);
        System.out.println("X: " + point.x);
    }

    public static void changeX(int x, Point givenPoint) {
        givenPoint.x = x;
    }
}
```

Output:

```
X: 3
```

The method `changeX` receives a copy of the reference that `point` holds. Both `point` and `givenPoint` point to the same `Point` object in memory, so modifying `givenPoint.x` changes the same object the caller sees.

![img]({{site.url}}/assets/blog_images/2023-02-20-is-java-pass-by-value-or-by-reference/value-reference-1-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2023-02-20-is-java-pass-by-value-or-by-reference/value-reference-1-dark.png){: .dark }

### Reassigning the Reference Inside a Method

```java
class Test {
    public static void main(String[] args) {
        Point point = new Point(2, 2);
        replaceAndChangeX(5, point);
        System.out.println("X: " + point.x);
    }

    public static void replaceAndChangeX(int x, Point givenPoint) {
        givenPoint = new Point(0, 0);
        givenPoint.x = x;
    }
}
```

You might expect the output to be 5, but it is:

```
X: 2
```

When the method starts, `givenPoint` holds the same address as `point`. But the line `givenPoint = new Point(0, 0)` reassigns the local copy to a brand-new object. From that point on, `givenPoint` and `point` refer to different objects. Setting `givenPoint.x = 5` modifies the new object, which the caller never sees.

![img]({{site.url}}/assets/blog_images/2023-02-20-is-java-pass-by-value-or-by-reference/value-reference-2-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2023-02-20-is-java-pass-by-value-or-by-reference/value-reference-2-dark.png){: .dark }

This is the definitive proof that Java is pass by value. If it were pass by reference, reassigning `givenPoint` inside the method would also change `point` in the caller. It does not.

### Primitives

For primitive types (`int`, `double`, `boolean`, etc.), the situation is simpler. Variables hold the values directly, not references. Passing a primitive to a method copies the value itself. Changes inside the method have no effect on the caller's variable:

```java
public static void tryToChange(int x) {
    x = 99;
}

public static void main(String[] args) {
    int value = 10;
    tryToChange(value);
    System.out.println(value); // 10
}
```

### Arrays

The same pass-by-value rule applies to arrays. When you pass an array to a method, Java copies the reference to the array, not the array itself. The method can modify the array's elements (because both references point to the same array object), but reassigning the parameter to a new array will not affect the original:

```java
public static void modifyArray(int[] arr) {
    arr[0] = 99;
}

public static void replaceArray(int[] arr) {
    arr = new int[] {10, 20, 30};
}

public static void main(String[] args) {
    int[] numbers = {1, 2, 3};
    modifyArray(numbers);
    System.out.println(numbers[0]); // 99 (same array object was modified)

    replaceArray(numbers);
    System.out.println(numbers[0]); // 99 (reassignment inside the method had no effect)
}
```

### Summary

| What is passed | What gets copied | Can the method modify the original? |
|---|---|---|
| Primitive | The value itself | No |
| Object reference | The address (reference value) | It can modify the object's fields, but it cannot make the caller's variable point to a different object |
