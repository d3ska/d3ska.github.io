---
title: "Comparable and Comparator Interfaces"
categories:
  - Blog
tags:
  - Comparator
  - Comparable
  - Sorting
---

### Comparable

Both interfaces are used to order the objects of a user-defined class.
The `Comparable<T>` interface is found in the `java.lang` package and contains only one method named `compareTo(T)`.<br>
**It provides a single sorting sequence only**, i.e., you can sort the elements on the basis of a single data member only.
For example, it may be name, age or anything else.

**compareTo(T obj) method**

`public int compareTo(T obj)`: It is used to compare the current object with the specified object. It returns:

* a positive integer, if the current object is greater than the specified object.
* a negative integer, if the current object is less than the specified object.
* zero, if the current object is equal to the specified object.

An important contract to keep in mind: the `compareTo` method should be consistent with `equals`. If `a.compareTo(b) == 0`, then `a.equals(b)` should ideally return `true`. Violating this can lead to unexpected behavior in sorted collections like `TreeSet`, which rely on `compareTo` to determine equality.

**Code Example**

We need to implement the Comparable interface first and override the compareTo method.

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
        if (age == student.getAge()) {
            return 0;
        } else if (age > student.getAge()) {
            return 1;
        } else {
            return -1;
        }
    }
}
```

or simpler:

```java
@Override
public int compareTo(Student student) {
    return Integer.compare(age, student.getAge());
}
```

<br>

```java
public class TestSort {
    public static void main(String[] args) {
        List<Student> students=new ArrayList<>();
        students.add(new Student("John", 28, 22176));
        students.add(new Student("Anastasia", 34, 44521));
        students.add(new Student("Json", 20, 69120));

        Collections.sort(students);
        students.forEach(System.out::println);
    }
}
```

```
Name: JsonAge: 20, StudentNumber: 69120
Name: JohnAge: 28, StudentNumber: 22176
Name: AnastasiaAge: 34, StudentNumber: 44521
```


### Comparator

Unlike Comparable, Comparator is external to the element type we are comparing. It's a separate class. We create multiple separate classes (that implement Comparator) to compare by different members.
The `Collections` class has a second `sort()` method, and it takes a Comparator which could be found in the `java.util` package.
The sort() method invokes the compare() to sort objects.
**To compare Student by age, we need to do 3 things :**

1) Create a class that implements Comparator (and thus the compare() method that does the work previously done by compareTo()).

2) Make an instance of the Comparator class.

3) Call the overloaded sort() method, giving it both the list and the instance of the class that implements Comparator.<br>
Or you may call sort() method directly on Collection that you want to sort and give it Comparator as an argument.

**Code Example**

```java
import java.util.Comparator;

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
    // Used for sorting in ascending order of age
    public int compare(Student a, Student b) {
        return a.age - b.age;
    }
}

class SortByName implements Comparator<Student> {
    // Used for sorting in ascending order of name
    public int compare(Student a, Student b) {
        return a.name.compareTo(b.name);
    }
}
```

<br>

```java
public class TestSort {
    public static void main(String[] args) {
        List<Student> students = new ArrayList<>();
        students.add(new Student("John", 28, 22176));
        students.add(new Student("Anastasia", 34, 44521));
        students.add(new Student("Json", 20, 69120));

        //Collections.sort(students, new SortByAge());
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

<br>

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

These additions make sorting code significantly shorter and more readable, and they eliminate the need for dedicated Comparator classes in most situations.

<br>

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
    <td>It doesn't affect the original class, i.e., actual class is not altered.</td>
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
    <td>The logic of sorting should be in a separate class to write different sorting based on different attributes of objects.</td>
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
    <td>Some well-known implementations of Comparator include Collator and RuleBasedCollator.</td>
  </tr>
</table>
