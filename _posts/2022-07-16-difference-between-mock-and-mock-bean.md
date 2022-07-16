---
title: "Difference between @Mock and @MockBean"
categories:
  - Blog
tags:
  - Testing
  - Mocks
---

#### Introduce

Some people may wonder what's the difference and when to use which, there is short definition of both.

<br>

#### @Mock

Mock annotation is an interchangeable with Mockito.mock() method, we can mock either class or interface.
But in case of annotation, we need to let spring know to initialize those by putting @ExtendWith(MockitoExtension.class)*
above testing class or invoking MockitoAnnotations.initMocks(this) method for example in setUp block.

The benefit of using annotations is definitely a readability and also @Mock makes it easier to find the issue as in case of a failure, the name of the field will be displayed in the failure message.

Furthermore, it works great with @InjectMocks and can reduce the amount of setup code significantly
\
<br>
<br>

#### @MockBean

MockBean annotation can be used to add mock object to the Spring context. 
The bean will replace any existing bean of the same type in the application context or will add new one, if none of that type will be not found.

This annotation can be useful in integration test where we want to mock some part of our logic, some bean like service or repository. 

To use this annotation, we have to use add also @ExtendWith(SpringExtension.class)* to run the test. 


<br>
<br>
* Mentioned annotations are available in JUnit5





