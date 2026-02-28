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

Software testing is an integral part of the development process. Each type of testing targets a different scope, and understanding when to use which is more valuable than memorizing definitions.

### Unit Tests

Unit tests operate at the lowest level, targeting individual methods, classes, or components. They run in isolation: no database, no network, no external dependencies. According to Google's testing taxonomy, these are "small tests" that run in a single process.

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

### Integration Tests

Integration tests verify that different modules work together correctly. They test the interaction between components, including integration with infrastructure like databases, message brokers, or external APIs. Google classifies these as "medium tests" that run on a single machine.

```java
@Test
void shouldFailWithConnectionResetByPeer() {
    WireMock.stubFor(WireMock.get("/18210116954")
        .willReturn(WireMock.aResponse().withFault(Fault.CONNECTION_RESET_BY_PEER)));

    BDDAssertions.thenThrownBy(() -> service.verify(zbigniew()))
        .hasRootCauseInstanceOf(IOException.class);
}
```

### End-to-End Tests

End-to-end (E2E) tests simulate the complete flow of a process from beginning to end, mimicking real user scenarios. They verify entire business processes, data consistency across services, and communication with external systems. Google classifies these as "large tests" that often require dedicated environments with databases, queues, and network setups similar to production.

```java
@Test
void shouldDisplayErrorMessage() {
    $(byLinkText("ERROR")).click();
    $(byText("Something happened...")).shouldBe(Condition.visible);
}
```

### Other Test Types

Beyond the core three, several other test types appear regularly in practice:

* **Manual tests.** Performed by people who understand the business domain, verifying user scenarios in dedicated testing environments.
* **Performance tests.** Verify the system's behavior under load using tools like JMeter or Gatling. The load profile is based on current and anticipated traffic patterns.
* **User tests.** Potential end-users carry out specific tasks while a UX specialist observes and identifies problematic areas.
* **Smoke tests.** A quick, shallow pass over the most critical functionality to verify that the system starts up and responds at all. A sanity check before running the full suite.
* **Regression tests.** Confirm that previously working functionality has not been broken by recent changes. In practice, any test suite that runs on every build acts as a regression safety net.
* **Security tests.** Verify resilience against common attack vectors (SQL injection, XSS, unauthorized access) using tools like OWASP ZAP or Burp Suite.
* **Contract tests.** Particularly relevant in microservice architectures, these verify that communication between a consumer and provider adheres to a shared agreement. Frameworks like Pact or Spring Cloud Contract help automate this.

> **Related post**: [Why Are Tests Essential?](/posts/why-are-tests-essential/)

### Testing Approaches

With the test types established, let's look at a different dimension: the **strategies** we use within tests to verify correctness. Regardless of whether a test is a unit test or an integration test, it typically falls into one of three approaches.

### Verifying Results

The most straightforward approach: call a method, check what it returns. The test does not care about internal state or side effects, only the output.

![img]({{site.url}}/assets/blog_images/2023-05-12-key-types-of-testing-in-software-development/result-verification-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2023-05-12-key-types-of-testing-in-software-development/result-verification-dark.png){: .dark }

```java
@Test
void shouldCreateStudentLoan() {
    LoanOrderService loanOrderService = new LoanOrderService();
    Customer student = aStudent();

    LoanOrder loanOrder = loanOrderService.studentLoanOrder(student);

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
    LoanOrder loanOrder = new LoanOrder(LocalDate.now(), aCustomer());
    UUID managerUuid = UUID.randomUUID();

    loanOrder.addManagerDiscount(managerUuid);

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
    customerVerifier.verify(customer);

    verify(eventEmitter, times(3)).emit(argThat(VerificationEvent::passed));
}
```

We use a mock to verify that `customerVerifier` emitted exactly three passing verification events. This approach is the most implementation-coupled of the three, so it should be used as a last resort when result verification and state checking are not feasible.
