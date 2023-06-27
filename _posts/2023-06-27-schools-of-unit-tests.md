---
title: "Unraveling the Schools of Unit Testing: The Classic and London Approaches"
categories:
  - Testing
  - System Quality
tags:
  - Testing
---

There exist two major methodologies when it comes to unit testing. These methodologies guide how much to use mocks, stubs, and the special test implementations known as test doubles. The two prominent schools of thought are the Classic School, also known as the "Detroit School" or the "Classical School," and the London School. These schools primarily differ in the way they define the concept of isolation.

### Classic School
In the Classic School, isolation signifies the separation of individual tests from each other. This school emphasizes running tests in parallel and in-memory, ensuring no shared state, such as databases, where individual tests could affect the state. The use of mocks, stubs, and test doubles should be minimal in this school. They're only employed for dependencies that might introduce a shared state and disrupt this kind of isolation.

The aim here is to use production code wherever possible. For these tests, a unit is defined as a class or even a set of classes. Therefore, we want to test an aggregate collectively without overutilizing mocks, stubs, and test doubles, restricting their usage to dependencies that would change the state.

### London School
In stark contrast, the London School defines a unit as a single class. The goal is to test this class in isolation, unlike the Classic School's approach that aims to test a whole component made up of classes performing a common functionality.

In the London School, we create a test implementation of a class, for instance, CustomerVerifier, that performs customer verification based on different implementations. The test implementation might be "always passed verification" or "always failed". Each verification type, such as by name or PESEL (Polish national identification number), would then be tested separately.

### Which School to Choose?
Both schools come with their distinct advantages and disadvantages. A balanced approach may be the most beneficial. The choice between the Classic School and the London School heavily depends on the specific circumstances in your project and the objects you want to test.

The Classic School's primary strength lies in the fact that its tests reflect the actual operation of the system, offering better coverage of the real system logic. However, the London School is not without its merits. It simplifies the process of isolating and finding errors. Furthermore, writing such tests is often faster and easier as the use of test doubles and mocks simplifies the intricate graphs of collaborators.

Jakub Pilimon proposed a heuristic in [his presentation "Testing – Love, Hate, Love"](https://www.youtube.com/watch?v=GjKYLmimGeE) that may be useful in this context. He recommends examining the class being tested and considering if the dependencies are separate merely for better code readability or if they introduce different responsibilities and layers. If it's solely a matter of readability, using the actual object is preferable. However, if multiple responsibilities and layers emerge, it's better to use a mock.

In conclusion, the key is to avoid being purists. Depending on the complexity of the collaborators' graph and the difficulty of setting up the tests, both schools can be employed. However, caution with mocks is advised. As noted by the authors of the Mockito mocking library, if everything is mocked, are we truly testing the production code at all? They advocate for a mixed approach, underscoring the importance of not being afraid to avoid mocking, suggesting a more balanced strategy rather than strict adherence to one school.