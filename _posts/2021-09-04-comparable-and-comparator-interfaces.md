---
title: "Comparable and Comparator Interfaces"
categories:
  - Java
tags:
  - Comparator
  - Comparable
  - Sorting
  - Java
  - Interfaces
---

Imagine you have a list of `Student` objects and you need to sort them by age. Later, a new requirement comes in: sort them by name instead. Then another: sort by student number. Java provides two interfaces for defining order between objects, `Comparable` and `Comparator`, and choosing the right one depends on whether you need a single default ordering or multiple interchangeable ones.

Both serve the same goal (defining an order between objects), but they differ in where the comparison logic lives and how many sort orders you can define.

### Comparable

The `Comparable<T>` interface is found in the `java.lang` package and contains a single method, `compareTo(T)`. A class that implements `Comparable` defines its own **natural ordering**. It provides a single sorting sequence only, meaning you can sort elements on the basis of one data member (for example, age).

The `compareTo` method returns:

* a positive integer if the current object is greater than the specified object
* a negative integer if the current object is less than the specified object
* zero if the current object is equal to the specified object

An important contract to keep in mind: `compareTo` should be consistent with `equals`. If `a.compareTo(b) == 0`, then `a.equals(b)` should ideally return `true`. Violating this can lead to unexpected behavior in sorted collections like `TreeSet`, which rely on `compareTo` to determine equality.

```java
public class Student implements Comparable<Student> {

    private String name;
    private final int age;
    private int studentNumber;

    public Student(String name, int age, int studentNumber) {
        this.name = name;
        this.age = age;
        this.studentNumber = studentNumber;
    }

    @Override
    public int compareTo(Student student) {
        return Integer.compare(age, student.getAge());
    }
}
```

`Integer.compare` is the recommended way to compare numeric fields. A common shortcut is `return this.age - other.age`, but this is a bug: if the values are large enough, the subtraction overflows and produces a wrong result. `Integer.compare` handles all edge cases correctly.

```java
public class TestSort {

    public static void main(String[] args) {
        List<Student> students = new ArrayList<>();
        students.add(new Student("John", 28, 22176));
        students.add(new Student("Anastasia", 34, 44521));
        students.add(new Student("Json", 20, 69120));

        Collections.sort(students);
        students.forEach(System.out::println);
    }
}
```

```
Name: Json, Age: 20, StudentNumber: 69120
Name: John, Age: 28, StudentNumber: 22176
Name: Anastasia, Age: 34, StudentNumber: 44521
```

### Comparator

Unlike `Comparable`, a `Comparator` is external to the element type being compared. It is a separate class (or lambda) that defines a custom ordering. This means you can create multiple comparators for the same class, each sorting by a different field, without modifying the class itself.

The `Comparator<T>` interface is in the `java.util` package and defines a `compare(T, T)` method.

```java
public class Student {

    String name;
    int age;
    int studentNumber;

    public Student(String name, int age, int studentNumber) {
        this.name = name;
        this.age = age;
        this.studentNumber = studentNumber;
    }
}

class SortByAge implements Comparator<Student> {

    public int compare(Student a, Student b) {
        return Integer.compare(a.age, b.age);
    }
}

class SortByName implements Comparator<Student> {

    public int compare(Student a, Student b) {
        return a.name.compareTo(b.name);
    }
}
```

```java
public class TestSort {

    public static void main(String[] args) {
        List<Student> students = new ArrayList<>();
        students.add(new Student("John", 28, 22176));
        students.add(new Student("Anastasia", 34, 44521));
        students.add(new Student("Json", 20, 69120));

        students.sort(new SortByAge());
        students.forEach(System.out::println);
    }
}
```

```
Name: Json, Age: 20, StudentNumber: 69120
Name: John, Age: 28, StudentNumber: 22176
Name: Anastasia, Age: 34, StudentNumber: 44521
```

### Java 8+ Comparator Features

Java 8 introduced a much more concise way to create comparators using static factory methods and lambdas. Instead of writing a full class that implements `Comparator`, we can use `Comparator.comparing()`:

```java
Comparator<Student> byName = Comparator.comparing(Student::getName);
```

We can chain comparators with `thenComparing()` to define secondary (and further) sort keys:

```java
Comparator<Student> byAgeThenName = Comparator.comparing(Student::getAge)
        .thenComparing(Student::getName);
```

Reversing the order is straightforward with `reversed()`:

```java
Comparator<Student> byAgeDescending = Comparator.comparing(Student::getAge).reversed();
```

When dealing with nullable fields, `Comparator.nullsFirst()` and `Comparator.nullsLast()` wrap an existing comparator to handle null values gracefully:

```java
Comparator<Student> byNameNullsFirst =
        Comparator.comparing(Student::getName, Comparator.nullsFirst(Comparator.naturalOrder()));
```

These additions make sorting code significantly shorter and more readable, and they eliminate the need for dedicated `Comparator` classes in most situations.

### Comparable vs Comparator

| | Comparable | Comparator |
|---|---|---|
| Package | `java.lang` | `java.util` |
| Method | `compareTo(T)` | `compare(T, T)` |
| Sort orders | Single (natural order) | Multiple |
| Modifies the class | Yes (class implements the interface) | No (comparison is external) |
| Typical use | Default ordering for the class | Alternative or ad-hoc orderings |

### When compareTo Is Inconsistent with equals

I mentioned earlier that `compareTo` should be consistent with `equals`. This is more than a theoretical concern. Sorted collections like `TreeSet` use `compareTo` (not `equals`) to determine whether two elements are duplicates. If two objects are different according to `equals` but `compareTo` returns 0, a `TreeSet` will silently drop one of them.

Here is a concrete example. Consider a `Student` class where `compareTo` compares only by age, but `equals` checks both name and age:

```java
public class Student implements Comparable<Student> {

    private final String name;
    private final int age;

    public Student(String name, int age) {
        this.name = name;
        this.age = age;
    }

    @Override
    public int compareTo(Student other) {
        return Integer.compare(this.age, other.age);
    }

    @Override
    public boolean equals(Object o) {
        if (this == o) return true;
        if (!(o instanceof Student)) return false;
        Student student = (Student) o;
        return age == student.age && Objects.equals(name, student.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }

    @Override
    public String toString() {
        return name + " (age " + age + ")";
    }
}
```

Now watch what happens when we add two students with the same age but different names:

```java
Student john = new Student("John", 25);
Student anna = new Student("Anna", 25);

System.out.println(john.equals(anna));       // false
System.out.println(john.compareTo(anna));    // 0

Set<Student> hashSet = new HashSet<>();
hashSet.add(john);
hashSet.add(anna);
System.out.println("HashSet size: " + hashSet.size()); // 2

Set<Student> treeSet = new TreeSet<>();
treeSet.add(john);
treeSet.add(anna);
System.out.println("TreeSet size: " + treeSet.size()); // 1 — Anna is lost!
```

`HashSet` uses `equals` and `hashCode`, so it correctly stores both students. `TreeSet` uses `compareTo`, sees that `john.compareTo(anna) == 0`, and treats Anna as a duplicate.

The fix is to make `compareTo` consistent with `equals` by including all fields that participate in equality. In this case, adding a secondary comparison on `name` solves the problem:

```java
@Override
public int compareTo(Student other) {
    int ageResult = Integer.compare(this.age, other.age);
    if (ageResult != 0) return ageResult;
    return this.name.compareTo(other.name);
}
```

The `TreeSet` Javadoc explicitly warns about this: "the ordering imposed by a comparator should be consistent with equals if it is to correctly implement the Set interface." In my experience, this is one of the more common subtle bugs in Java, because everything appears to work correctly until someone swaps a `HashSet` for a `TreeSet` and data silently disappears.

> **Related posts**: [Java equals and hashCode contract](/posts/java-equals-and-hashCode-contract/), [BigDecimal and BigInteger in Java](/posts/bigdecimal-and-biginteger/)
