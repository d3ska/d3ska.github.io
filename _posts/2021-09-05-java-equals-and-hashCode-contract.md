---
title: "Java equals and hashCode contract"
categories:
  - Java
tags:
  - Equals
  - HashCode
  - Java
  - Effective Java
---


### equals() method

The `equals()` method is one of the fundamental methods available in the `Object` class.
Due to the fact that every object in Java has this class in its inheritance hierarchy, we can call this method on every object.
In most cases, the default implementation of the equals method is not appropriate, therefore the programmer creating a new object must implement this method if they want to check if the instances of this class are equal.
There is a set of guidelines that the equals method should meet in order to be properly implemented.

**The equals() method must be:**

* reflexive: an object must equal itself
* symmetric: x.equals(y) must return the same result as y.equals(x)
* transitive: if x.equals(y) and y.equals(z) then also x.equals(z)
* consistent: the value of equals() should change only if a property that is contained in equals() changes (no randomness allowed)
* non-null: for any non-null reference x, `x.equals(null)` must return `false`

### instanceof vs getClass() in equals

When implementing `equals()`, there are two common strategies for the type check: using **`instanceof`** or using **`getClass()`**. The choice matters more than it might seem at first, especially once inheritance enters the picture.

#### The instanceof approach

The `instanceof` check allows subclass instances to be considered equal to parent instances. This supports the Liskov Substitution Principle: if a `ColorPoint` extends `Point`, a `ColorPoint` at (1, 2) can still be equal to a `Point` at (1, 2).

```java
public class Point {
    private final int x;
    private final int y;

    public Point(int x, int y) {
        this.x = x;
        this.y = y;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Point point)) return false;
        return x == point.x && y == point.y;
    }

    @Override
    public int hashCode() {
        return Objects.hash(x, y);
    }
}
```

```java
Point point = new Point(1, 2);
ColorPoint colorPoint = new ColorPoint(1, 2, Color.RED);

point.equals(colorPoint); // true - subclass treated as equal
```

This works well when subclasses don't add state that should participate in equality. However, it can break the symmetry contract if the subclass also overrides equals and considers the extra fields. If `ColorPoint.equals()` checks color, then `point.equals(colorPoint)` returns `true` but `colorPoint.equals(point)` returns `false`.

#### The getClass() approach

The `getClass()` check is stricter. Only objects of the exact same runtime class can be equal.

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    Point point = (Point) o;
    return x == point.x && y == point.y;
}
```

```java
Point point = new Point(1, 2);
ColorPoint colorPoint = new ColorPoint(1, 2, Color.RED);

point.equals(colorPoint); // false - different classes
```

This guarantees symmetry is never broken by subclass overrides, but it violates the Liskov Substitution Principle: you can no longer use a `ColorPoint` wherever a `Point` is expected and get the same equality behavior.

#### When to use each

I generally prefer `instanceof` for class hierarchies where subclasses don't introduce new fields relevant to equality (value types, DTOs). I reach for `getClass()` when subclasses add state that must participate in equality, because it avoids the symmetry trap entirely. Joshua Bloch's *Effective Java* covers this trade-off in depth and is worth reading if you deal with deep class hierarchies.

### hashCode() method

As with equals, `hashCode()` is implemented in the Object class. Whenever the programmer implements the hashCode method, they should also implement the equals method.

This method returns a number of the int type, which is used to assign a given object to a group. Thanks to the hashCode method, we are able to divide all possible instances of a given class into separate groups. Each of these groups is represented by a number returned by the hashCode method.

**The hashCode method is used by collections such as HashSet and HashMap to ensure data consistency. Without overriding both equals and hashCode, these collections will not behave properly.**


### The contract between equals and hashCode

The hashCode and equals methods are related, and their implementation should be consistent. This relationship is defined by the **contract between hashCode and equals**.

* If `X.equals(Y) == true` then it is required that `X.hashCode() == Y.hashCode()`.
* Multiple calls of the hashCode method on the same object, which has not been modified between calls, must return the same value.
* If `X.hashCode() == Y.hashCode()` then it is not required that `X.equals(Y) == true`.


#### Code Example

```java
@Getter
@Setter
public class Student {

    private String name;
    private int age;

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

}
```

```java
Student student1 = new Student("John", 21);
Student student2 = new Student("John", 21);

System.out.println("Student1 hashCode: " + student1.hashCode());
System.out.println("Student2 hashCode: " + student2.hashCode());
System.out.println("Equality between Student1 and Student2: " + student1.equals(student2));
```

```
Student1 hashCode: 1608446010
Student2 hashCode: 914504136
Equality between Student1 and Student2: false
```

Comparing two objects without the equals() method doesn't work as it should.

```java
@Override
public boolean equals(Object o) {
    if (this == o)
        return true;
    if (o == null || getClass() != o.getClass())
        return false;
    Student student = (Student) o;
    return age == student.age && Objects.equals(name, student.name);
}
```

```
Student1 hashCode: 71751691
Student2 hashCode: 71751691
Equality between Student1 and Student2: true
```

As you can see, the overridden equals() method was enough to check for equality between objects.
But how will the HashSet behave with these objects?

```java
Student student1 = new Student("John", 21);
Student student2 = new Student("John", 21);

Set<Student> students = new HashSet<>();
students.add(student1);
students.add(student2);

System.out.println("HashSet size = " + students.size());
System.out.println("HashSet contains student = " + students.contains(new Student("John", 21)));
```

```
HashSet size = 2
HashSet contains student = false
```

The HashSet should not add an object if the same object already exists in the collection.
After overriding the hashCode() method, the behavior is correct:

```java
@Override
public int hashCode() {
    return Objects.hash(name, age);
}
```

```
HashSet size = 1
HashSet contains student = true
```

### What happens with HashMap when the contract is broken

The HashSet example above shows duplicates, but the consequences with **HashMap** can be even more confusing. When you override `equals()` but forget `hashCode()`, objects that are logically equal end up in different hash buckets, so the map cannot find them.

```java
public class StudentKey {
    private final String name;
    private final int age;

    public StudentKey(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (o == null || getClass() != o.getClass()) return false;
        StudentKey that = (StudentKey) o;
        return age == that.age && Objects.equals(name, that.name);
    }

    // hashCode() intentionally NOT overridden
}
```

```java
Map<StudentKey, String> grades = new HashMap<>();
StudentKey key1 = new StudentKey("John", 21);
grades.put(key1, "A+");

StudentKey key2 = new StudentKey("John", 21);
System.out.println("key1.equals(key2): " + key1.equals(key2));
System.out.println("Grade lookup: " + grades.get(key2));
```

```
key1.equals(key2): true
Grade lookup: null
```

The two keys are equal according to `equals()`, but since `hashCode()` is not overridden, each instance inherits the default identity-based hash from `Object`. The HashMap hashes `key2` to a different bucket than `key1`, never finds it, and returns `null`. I have personally debugged this exact issue in production code where a cache was growing unbounded because "duplicate" keys kept getting inserted.

After adding the proper `hashCode()`:

```java
@Override
public int hashCode() {
    return Objects.hash(name, age);
}
```

```
key1.equals(key2): true
Grade lookup: A+
```

### Objects.equals() and Objects.hash()

Java 7 introduced the `Objects` utility class, which provides convenient helper methods for implementing equals and hashCode. `Objects.equals(a, b)` handles null checks automatically, returning `true` if both arguments are null and delegating to `a.equals(b)` otherwise. This eliminates repetitive null-checking boilerplate:

```java
@Override
public boolean equals(Object o) {
    if (this == o) return true;
    if (!(o instanceof Person person)) return false;
    return Objects.equals(name, person.name) && age == person.age;
}
```

Similarly, `Objects.hash(Object... values)` generates a hash code from multiple fields in a single call:

```java
@Override
public int hashCode() {
    return Objects.hash(name, age);
}
```

These utility methods reduce the chance of NullPointerException and make the implementation cleaner and less error-prone.

### Conclusion

In order to achieve a fully working custom equality mechanism, it is mandatory to override `hashCode()` each time you override `equals()`.

* If two objects are equal, they **must** have the same hash code.
* If two objects have the same hash code, it doesn't mean that they are equal.
* Overriding `equals()` alone will break hash-based data structures like HashSet, HashMap, and Hashtable.
* Overriding `hashCode()` alone doesn't force Java to ignore memory addresses when comparing two objects.

In practice, you rarely need to write these methods by hand. Most IDEs (IntelliJ IDEA, Eclipse) can generate correct `equals()` and `hashCode()` implementations for you. Lombok's `@EqualsAndHashCode` annotation does the same with zero boilerplate. If you are on Java 16 or later, **records** give you correct value-based `equals()` and `hashCode()` for free:

```java
public record Student(String name, int age) {}
```

Whichever approach you choose, the important thing is that the contract is never violated. When it is, the bugs tend to be subtle and painful to track down.

> **Related posts**: [Hash Tables](/posts/hash-tables/), [Hashing](/posts/hashing/)
