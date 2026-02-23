---
title: "Difference between @Mock and @MockBean"
categories:
  - Blog
tags:
  - Testing
  - Mocks
---

#### Introduction

Some people may wonder what the difference is and when to use which. Here is a short definition of both.

<br>

#### @Mock

The `@Mock` annotation is interchangeable with `Mockito.mock()` method. We can mock either a class or an interface.
When using the annotation, we need to let Mockito know to initialize those mocks by putting `@ExtendWith(MockitoExtension.class)`* above the testing class, or by invoking `MockitoAnnotations.openMocks(this)` in the setUp block. (Note: the older `initMocks()` method has been deprecated since Mockito 2.x; prefer `openMocks()` instead.)

The benefit of using annotations is definitely readability, and also `@Mock` makes it easier to find the issue, as in case of a failure, the name of the field will be displayed in the failure message.

Furthermore, it works great with `@InjectMocks` and can reduce the amount of setup code significantly.
\
<br>
<br>

#### @MockBean

`@MockBean` is a Spring Boot annotation (provided by the `spring-boot-test` module, not core Spring Framework) that can be used to add a mock object to the Spring application context.
The bean will replace any existing bean of the same type in the application context or will add a new one if none of that type is found.

This annotation is useful in integration tests where we want to mock some part of our logic, such as a specific service or repository bean.

To use this annotation, we also need to add `@ExtendWith(SpringExtension.class)`* to run the test.

Keep in mind that `@MockBean` causes the Spring application context to be reloaded (or a new context to be created) when the set of mocked beans differs between test classes. In large projects, this can noticeably slow down the test suite. Where possible, try to keep the same `@MockBean` configuration across test classes to maximize context caching.

<br>
<br>
* Mentioned annotations are available in JUnit 5. Note that `@ExtendWith(MockitoExtension.class)` comes from the Mockito library (mockito-junit-jupiter), while `@ExtendWith(SpringExtension.class)` comes from the Spring Test module.




