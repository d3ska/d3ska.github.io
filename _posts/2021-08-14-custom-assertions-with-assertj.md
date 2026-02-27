---
title: "Custom Assertions with AssertJ"
categories:
  - Testing
tags:
  - Testing
  - AssertJ
  - Custom Assertions
  - Domain Driven Design
  - DDD
  - Java
---

You might have encountered tests that are difficult to read, especially when the failure messages are unclear. In such cases, consider creating your own specific assertions using the domain model vocabulary. This approach not only makes them easier to read but also helps apply Domain Driven Design's ubiquitous language in tests.

> **Note:** AssertJ is typically already available as a transitive dependency through `spring-boot-starter-test`. If you are not using Spring Boot, add it explicitly. For Maven: `assertj-core` in `org.assertj` group. For Gradle: `testImplementation 'org.assertj:assertj-core:3.x.x'`.

**Implementation**

Let's start with the class we want to test:

```java
public class Person {
    private String fullName;
    private int age;
    private List<String> nicknames;

    public Person(String fullName, int age) {
        this.fullName = fullName;
        this.age = age;
        this.nicknames = new ArrayList<>();
    }

    public void addNickname(String nickname) {
        nicknames.add(nickname);
    }
}
```

First, create a class that extends `AbstractAssert`. The two generic type parameters are the assertion class itself (in this case, `PersonAsserts`) and the class under test. A constructor as shown below is also needed:

```java
public class PersonAssert extends AbstractAssert<PersonAssert, Person> {

    public PersonAssert(Person actual) {
        super(actual, PersonAssert.class);
    }

    // assertion methods described later
}
```

Now, we can add our custom assertions along with personalized failure messages:

```java
public PersonAssert isAdult() {
    isNotNull();
    if (actual.getAge() < 18) {
        failWithMessage("Expected person to be adult");
    }
    return this;
}

public PersonAssert hasFullName(String fullName) {
    isNotNull();
    if (!actual.getFullName().equals(fullName)) {
        failWithMessage("Expected person to have" +
                " full name %s but was %s",
                fullName, actual.getFullName());
    }
    return this;
}

public PersonAssert hasNickName(String nickName) {
    isNotNull();
    if (!actual.getNicknames().contains(nickName)) {
        failWithMessage("Expected person to have nickname %s",
                nickName);
    }
    return this;
}
```

First, check if the object we are testing is not null. Then, implement assertions as shown above. In the failWithMessage method, pass our custom message, making it more readable.

We're almost ready to use our custom assertions. If we have more than one custom assertion class, we can wrap all assertThat methods in a class, providing a static factory method for each of the assertion classes:

```java
public class ProjectAssertions {

    public static PersonAssert assertThat(Person actual) {
        return new PersonAssert(actual);
    }

}
```

<br>

Now, let's look at some examples of tests:

```java
@Test
public void whenPersonAgeLessThanEighteen_thenNotAdult() {
    Person person = new Person("Jane Roe", 16);

    // assertion fails
    assertThat(person).isAdult();
}

@Test
public void whenPersonNameMatches_thenCorrect() {
    Person person = new Person("John Doe", 20);
    assertThat(person)
            .hasFullName("John Doe");
}

@Test
public void whenPersonDoesNotHaveAMatchingNickname_thenIncorrect() {
    Person person = new Person("John Doe", 20);
    person.addNickname("Nick");

    // assertion will fail
    assertThat(person)
            .hasNickName("John");
}
```

> **Tip:** For larger projects, the `assertj-assertions-generator` Maven plugin can automatically generate custom assertion classes from your domain model. This can save a significant amount of boilerplate when you have many domain objects.

Writing custom assertions takes a bit of upfront effort, but the payoff is substantial: tests become self-documenting, failure messages speak the language of the domain, and the overall test suite is far easier for the team to maintain and extend.
