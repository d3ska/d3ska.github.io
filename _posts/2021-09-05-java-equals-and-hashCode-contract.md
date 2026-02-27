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

It is one of the methods available in the Object class.
Due to the fact that every object in Java has this class in its inheritance hierarchy, we can call this method on every object.
In most cases, the default implementation of the equals method is not appropriate, therefore the programmer creating a new object must implement this method if they want to check if the instances of this class are equal.
There is a set of guidelines that the equals method should meet in order to be properly implemented.<br>
**The equals() method must be:**

* reflexive: an object must equal itself
* symmetric: x.equals(y) must return the same result as y.equals(x)
* transitive: if x.equals(y) and y.equals(z) then also x.equals(z)
* consistent: the value of equals() should change only if a property that is contained in equals() changes (no randomness allowed)
* non-null: for any non-null reference x, `x.equals(null)` must return `false`

**instanceof vs getClass() in equals**

When implementing `equals()`, there are two common strategies for the type check: using `instanceof` or using `getClass()`. The `instanceof` approach (`if (!(obj instanceof MyClass))`) allows subclasses to be equal to parent instances, which supports the Liskov Substitution Principle. The `getClass()` approach (`if (obj == null || getClass() != obj.getClass())`) is stricter and only considers objects of the exact same runtime class as potentially equal. I generally prefer the `instanceof` approach for open class hierarchies and `getClass()` when strict type matching is required.

### hashCode() method

As with equals, hashCode is implemented in the Object class. Whenever the programmer implements the hashCode method, they should also implement the equals method.

This method returns a number of the int type, which is used to assign a given object to a group. Thanks to the hashCode method, we are able to divide all possible instances of a given class into separate groups. Each of these groups is represented by a number returned by the hashCode method.

**The hashCode method is used by collections such as HashSet and HashMap to ensure data consistency. Without overriding both equals and hashCode, these collections wouldn't behave properly.**


### The contract between equals and hashCode

The hashCode and equals methods are related, and their implementation should be consistent. This relationship is defined by the contract between hashCode and equals.

* If `X.equals(Y) == true` then it is required that `X.hashCode() == Y.hashCode()`.
* Multiple calls of the hashCode method on the same object, which has not been modified between calls, must return the same value.
* If `X.hashCode() == Y.hashCode()` then it is not required that `X.equals(Y) == true`.


<br>

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
But let's see how the HashSet will behave with these objects.

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
Let's check it again after overriding the hashCode() method.

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

<br>

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

**Conclusion**

In order to achieve a fully working custom equality mechanism, it is mandatory to override hashCode() each time you override equals().

* If two objects are equal, they MUST have the same hash code.
* If two objects have the same hash code, it doesn't mean that they are equal.
* Overriding equals() alone will make your business fail with hashing data structures like: HashSet, HashMap, HashTable ... etc.
* Overriding hashCode() alone doesn't force Java to ignore memory addresses when comparing two objects.
