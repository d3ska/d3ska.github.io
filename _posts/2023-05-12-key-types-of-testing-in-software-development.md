---
title: "Key Types of Testing in Software Development"
categories:
  - Testing
tags:
  - Testing
  - Unit Testing
  - Integration Testing
  - E2E Testing
---

You have a codebase with hundreds of tests, yet bugs still reach production. Meanwhile, the CI pipeline takes 45 minutes because the E2E suite keeps timing out on flaky browser tests. Something is off, and it is probably not the number of tests. It is the balance between them.

Each type of test serves a different purpose, and understanding when to reach for which one matters far more than memorizing definitions. In this post I will walk through the core test types, the testing pyramid that guides how to distribute them, and common anti-patterns that undermine even well-intentioned test suites.

### The Testing Pyramid

The **testing pyramid** is the most widely referenced model for thinking about test distribution. Picture a triangle divided into three horizontal layers:

- **Base (Unit tests)** -- The widest layer. These are fast, cheap, and numerous. They run in milliseconds and give immediate feedback.
- **Middle (Integration tests)** -- Fewer in number. They verify that components work together correctly, but take longer to run and require more infrastructure.
- **Top (E2E tests)** -- The narrowest layer. These simulate real user flows across the entire system. They are slow, expensive, and prone to flakiness.

The rationale is straightforward: the higher you go, the more expensive each test becomes in terms of execution time, maintenance cost, and debugging difficulty. A failing unit test points you to a specific function. A failing E2E test might take an hour to diagnose.

A commonly cited guideline for distribution is roughly **70% unit tests, 20% integration tests, and 10% E2E tests**. These numbers are not rules. They are a starting point. A CRUD-heavy web application that mostly shuffles data between a database and an API might benefit from a heavier investment in integration tests. A math library with pure functions and no external dependencies might be almost entirely unit tests. The right ratio depends on where the risk lives in your particular application.

#### The Testing Trophy

Kent C. Dodds popularized an alternative model called the **Testing Trophy**, which argues that integration tests give the best return on investment for most web applications. The trophy shape puts integration tests as the largest band, with unit tests below and E2E tests above. The idea is that integration tests catch the bugs that actually matter (components failing to work together) while still being reasonably fast to run. It is worth knowing about both models and choosing the balance that fits your system.

### Unit Tests

**Unit tests** operate at the lowest level, targeting individual methods, classes, or components. They run in isolation: no database, no network, no external dependencies. According to Google's testing taxonomy, these are "small tests" that run in a single process.

Unit tests are fast, cheap, and give immediate feedback. They use simple objects, stubs, and mocks, without requiring an application framework to be running.

```java
@Test
void verificationShouldPassForAgeBetween18And99() {
    // Given
    AgeVerification verification = new AgeVerification(22);
    // When
    boolean passes = verification.passes();
    // Then
    assertThat(passes).isTrue();
}
```

Unit tests shine when testing pure business logic, calculations, and decision branches. If a function takes inputs and returns outputs with no side effects, a unit test is almost always the right choice.

### Integration Tests

**Integration tests** verify that different modules work together correctly. They test the interaction between components, including integration with infrastructure like databases, message brokers, or external APIs. Google classifies these as "medium tests" that run on a single machine.

```java
@Test
void shouldFailWithConnectionResetByPeer() {
    // Given
    WireMock.stubFor(WireMock.get("/18210116954")
        .willReturn(WireMock.aResponse().withFault(Fault.CONNECTION_RESET_BY_PEER)));

    // When / Then
    BDDAssertions.thenThrownBy(() -> service.verify(zbigniew()))
        .hasRootCauseInstanceOf(IOException.class);
}
```

Integration tests catch an entire category of bugs that unit tests cannot: serialization mismatches, incorrect SQL queries, misconfigured HTTP clients, broken message formats. The trade-off is that they are slower and require infrastructure (a test database, a WireMock server, a Docker container).

### End-to-End Tests

**End-to-end (E2E) tests** simulate the complete flow of a process from beginning to end, mimicking real user scenarios. They verify entire business processes, data consistency across services, and communication with external systems. Google classifies these as "large tests" that often require dedicated environments with databases, queues, and network setups similar to production.

E2E tests are best reserved for critical user journeys where failure would have serious business impact:

```java
@Test
void userSignsUpAndCanLogIn() {
    // Given -- user navigates to signup page
    open("/signup");
    $("#email").setValue("newuser@example.com");
    $("#password").setValue("SecureP@ss123");
    $("#confirm-password").setValue("SecureP@ss123");

    // When -- user submits the registration form
    $("#signup-button").click();
    $(byText("Check your email to confirm your account")).shouldBe(visible);

    // Then -- after confirming email, user can log in
    open(getConfirmationLink("newuser@example.com"));
    $(byText("Email confirmed")).shouldBe(visible);

    open("/login");
    $("#email").setValue("newuser@example.com");
    $("#password").setValue("SecureP@ss123");
    $("#login-button").click();
    $(byText("Welcome")).shouldBe(visible);
    $(".user-dashboard").shouldBe(visible);
}
```

This test covers the full signup flow: form submission, email confirmation, and login. It exercises the frontend, backend, email service, and database together. That breadth is exactly why E2E tests are valuable for critical paths, and exactly why they are expensive. A flaky email service or a slow database migration can break this test even when the application code is perfectly correct.

### Contract Tests

**Contract tests** deserve special attention because they solve a problem that other test types handle poorly: verifying communication between services that are developed and deployed independently.

In a microservice architecture, Service A (the consumer) depends on an API provided by Service B (the provider). Unit tests for each service pass individually. Integration tests within each service pass too. But when Service B changes its response format from `{ "userName": "..." }` to `{ "user_name": "..." }`, Service A breaks in production because nobody caught the mismatch.

Contract tests solve this by establishing a shared agreement between consumer and provider. The consumer defines what it expects from the provider (the contract). The provider verifies that it still satisfies that contract. If a change breaks the contract, the provider's build fails before the change is deployed.

The **Pact** framework is the most widely used tool for this. Spring Cloud Contract is another popular option in the Java ecosystem. The key idea is that both sides run tests against the same contract definition, so breaking changes are caught at build time rather than in production.

Contract tests are not a replacement for integration tests. They specifically target the boundary between services, making sure both sides agree on the shape of their communication.

### Other Test Types

Beyond the core categories above, several other test types appear regularly in practice:

* **Manual tests.** Performed by people who understand the business domain, verifying user scenarios in dedicated testing environments.
* **Performance tests.** Verify the system's behavior under load using tools like JMeter or Gatling. The load profile is based on current and anticipated traffic patterns.
* **User tests.** Potential end-users carry out specific tasks while a UX specialist observes and identifies problematic areas.
* **Smoke tests.** A quick, shallow pass over the most critical functionality to verify that the system starts up and responds at all. A sanity check before running the full suite.
* **Regression tests.** Confirm that previously working functionality has not been broken by recent changes. In practice, any test suite that runs on every build acts as a regression safety net.
* **Security tests.** Verify resilience against common attack vectors (SQL injection, XSS, unauthorized access) using tools like OWASP ZAP or Burp Suite.

### When to Write Which Type of Test

Choosing the right test type is not always obvious, but a few guidelines help:

- **Pure business logic with no dependencies** -- unit test. A pricing calculator, a validation rule, a sorting algorithm. These are fast to write, fast to run, and give precise feedback.
- **Code that interacts with a database or external API** -- integration test. Repository queries, HTTP client behavior, message serialization. You need the real infrastructure (or a realistic fake) to catch the bugs that matter here.
- **Critical user journeys** -- E2E test. Checkout, signup, payment processing. These are the flows where a production bug costs real money or loses real users.
- **Communication between independently deployed services** -- contract test. If two teams own different sides of an API, a contract test ensures they stay in sync without requiring a full integrated environment.

When in doubt, start with the fastest test that can catch the bug you are worried about. If a unit test can verify the behavior, write a unit test. Only reach for a slower, more expensive test type when the faster option cannot cover the scenario.

### Anti-Patterns to Avoid

A test suite can look impressive by the numbers and still be counterproductive. Here are three patterns I have seen cause the most damage.

#### The Ice Cream Cone

The **ice cream cone** is the testing pyramid flipped upside down: lots of E2E tests at the top, a handful of integration tests in the middle, and barely any unit tests at the base. This happens naturally when teams write tests after the fact, starting with "let's just verify the whole feature works" instead of building up from focused unit tests.

The result is slow builds, flaky pipelines, and tests that are painful to debug. A single failing E2E test might take 30 minutes to reproduce locally. Teams start ignoring failures, and the suite loses its value as a safety net.

#### Testing Implementation Details

When tests are tightly coupled to how the code works internally (which private methods are called, the exact order of operations, the specific data structures used), every refactor breaks the tests even when the behavior has not changed. This creates a vicious cycle: developers stop refactoring because it means rewriting tests, and the codebase gradually rots.

Good tests verify **what** the code does (its observable behavior), not **how** it does it. If you can completely rewrite the internals of a class and the tests still pass because the inputs and outputs have not changed, your tests are at the right level of abstraction.

#### No Integration Tests

All unit tests pass. Deployment goes out. The application crashes because the SQL query references a column that was renamed last week, or the JSON serializer uses a different date format than the API expects.

Unit tests verify components in isolation, which means they cannot catch problems at the boundaries between components. A codebase with extensive unit test coverage but zero integration tests is a codebase where individual pieces all work correctly and the system as a whole does not.

### Testing Approaches

With the test types and strategy established, there is a different dimension worth examining: the **strategies** we use within tests to verify correctness. Regardless of whether a test is a unit test or an integration test, it typically falls into one of three approaches.

### Verifying Results

The most straightforward approach: call a method, check what it returns. The test does not care about internal state or side effects, only the output.

![img]({{site.url}}/assets/blog_images/2023-05-12-key-types-of-testing-in-software-development/result-verification-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2023-05-12-key-types-of-testing-in-software-development/result-verification-dark.png){: .dark }

```java
@Test
void shouldCreateStudentLoan() {
    // Given
    LoanOrderService loanOrderService = new LoanOrderService();
    Customer student = aStudent();

    // When
    LoanOrder loanOrder = loanOrderService.studentLoanOrder(student);

    // Then
    assertThat(loanOrder.getPromotions())
        .filteredOn(promotion -> promotion.getName().equals("Student Promo"))
        .size().isEqualTo(1);
}
```

We call `studentLoanOrder` and verify the returned `LoanOrder` contains the expected promotion. This is the preferred approach when possible, because it tests behavior without coupling to implementation details.

### Checking State

When a method returns `void` or the result alone does not capture the full effect, we verify the state of the system after the operation.

![img]({{site.url}}/assets/blog_images/2023-05-12-key-types-of-testing-in-software-development/state-verification-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2023-05-12-key-types-of-testing-in-software-development/state-verification-dark.png){: .dark }

```java
@Test
void shouldAddManagerPromo() {
    // Given
    LoanOrder loanOrder = new LoanOrder(LocalDate.now(), aCustomer());
    UUID managerUuid = UUID.randomUUID();

    // When
    loanOrder.addManagerDiscount(managerUuid);

    // Then
    assertThat(loanOrder.getPromotions()).hasSize(1);
    assertThat(loanOrder.getPromotions().get(0).getName())
        .contains(managerUuid.toString());
    assertThat(loanOrder.getPromotions().get(0).getDiscount())
        .isEqualTo(50);
}
```

The method `addManagerDiscount` returns void, so we inspect the `LoanOrder`'s state after the call. This approach is useful but couples the test more tightly to the object's internal structure.

### Verifying Communication

Sometimes we need to verify that an object sent the right messages to its collaborators, without caring about the return value or internal state.

![img]({{site.url}}/assets/blog_images/2023-05-12-key-types-of-testing-in-software-development/interaction-verification-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2023-05-12-key-types-of-testing-in-software-development/interaction-verification-dark.png){: .dark }

```java
@BeforeEach
void setUp() {
    customer = buildCustomer();
    eventEmitter = mock(EventEmitter.class);
    customerVerifier = new CustomerVerifier(buildVerifications(eventEmitter));
}

@Test
void shouldEmitVerificationEvents() {
    // When
    customerVerifier.verify(customer);

    // Then
    verify(eventEmitter, times(3)).emit(argThat(VerificationEvent::passed));
}
```

We use a mock to verify that `customerVerifier` emitted exactly three passing verification events. This approach is the most implementation-coupled of the three, so it should be used as a last resort when result verification and state checking are not feasible.

### Putting It All Together

No single type of test is sufficient on its own. Unit tests give you speed and precision but miss integration issues. Integration tests catch boundary problems but are too slow to cover every edge case. E2E tests provide the highest confidence that the system works as a whole but are expensive and fragile. Contract tests fill the gap between independently deployed services.

A well-designed testing strategy uses all of these in proportion to where the risk lives. The testing pyramid provides a solid starting point, but the exact balance is something you adjust based on your architecture, your team, and the kinds of bugs that actually reach production. The goal is not to maximize the number of tests. It is to maximize the confidence they provide, at a cost you can sustain.

> **Related posts**: [Why Are Tests Essential?](/posts/why-are-tests-essential/), [The Economics of Software Testing](/posts/economy-of-testing/), [Schools of Unit Testing](/posts/schools-of-unit-tests/), [Best Practices for Writing Effective and Reliable Tests](/posts/best-practices-for-writing-effective-and-reliable-tests/)
