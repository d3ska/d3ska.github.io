---
title: "Key Types of Testing in Software Development"
categories:
  - Testing
  - System Quality
tags:
  - Testing
---

Software testing is an integral part of the development process. Each type of testing has its specific purpose and helps ensure the highest quality of the final product. Here are the most crucial types of tests used in software development.

### Unit Tests

Unit tests operate at the lowest level, targeting individual methods, classes, components, and modules without any integration with other modules or involving any database communication or simulated HTTP traffic. These are considered "small tests" according to Google's scale and involve a single process.

Unit tests are quick to execute and involve simple objects, structures, stubs, and mocks, without the need for setting up and running application frameworks.

```java
@Test
void verificationShouldPassForAgeBetween18And99() {
    // given
    AgeVerification verification = new AgeVerification(22);
    // when
    boolean passes = verification.passes();
    // then
    assertThat(passes).isTrue();
}
```


### Integration Tests

Integration tests are designed to verify the integration and interaction between different modules. They check whole groups of modules responsible for specific business functionalities. They validate the consistency of returned results concerning business requirements and the interaction between the individual modules within the group.

They also test integration with the infrastructure. According to Google's scale, these tests are considered "medium tests" and are performed on a single machine.

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

End-to-End tests simulate the complete flow of a given process in an application or distributed system from beginning to end. They are intended to mimic real-life scenarios of end-users and verify the correctness of entire business processes, data consistency, integration of different applications that are part of the system, and communication with external systems.

Google's scale classifies these as "large tests" and they often require dedicated testing environments, databases, queues, and network protocols similar to those used in production.

```java
@Test
void shouldDisplayErrorMessage() {
    $(byLinkText("ERROR")).click();
    $(byText("Something happened...")).shouldBe(Condition.visible);
}
```

### Manual Tests

Manual tests are performed by people who have a good understanding of the business side of a given system. They verify entire business processes and user scenarios in dedicated testing environments with data similar to production data.

### Performance Tests

Performance tests verify the system's performance under a given load. Specialized tools like JMeter are used for performance and load testing. The design of the load is based on the current and anticipated system load as well as key performance indicators (KPIs).

### User Testing

User tests are performed by potential end-users who carry out specific tasks or processes in the application. These tests are monitored by a UX specialist to identify problematic areas from a user experience perspective.

### Conclusion

Remember, each type of testing serves its purpose, and their collective use ensures the development of a robust, user-friendly, and efficient software system.

---

### Testing Approaches: Verifying Results, Checking State, and Ensuring Communication

In addition to the various types of testing, such as unit testing, integration testing, and end-to-end testing, there are also different testing approaches that can be employed in software development.

### Verifying Results

One approach to testing involves verifying the results returned by a component after processing specific input data. This type of test focuses on the output of the code and does not require verifying the internal state of the component or any side effects it may have. By validating the returned result against expected values, we can ensure that the component behaves as intended.

![img]({{site.url}}/assets/blog_images/2023-05-12-key-types-of-testing-in-software-development/result-verfication.png)

Consider the following example:

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
In this test, we create a student loan order using the LoanOrderService and verify that the resulting LoanOrder contains a promotion named "Student Promo." By checking the size of the promotions list and filtering it based on the promotion name, we can assert the expected behavior of the code.

This approach to testing focuses on the effectiveness of the tests by directly comparing the actual output with the expected output. It promotes better architectural design as it encourages writing code that produces verifiable results.


### Checking State

Another testing approach involves verifying the state of the system after the completion of an operation. This type of test focuses on the state of the tested component, its collaborators, or external dependencies (e.g., in integration tests). By checking the state of relevant objects, we can ensure that the desired changes have occurred.

![img]({{site.url}}/assets/blog_images/2023-05-12-key-types-of-testing-in-software-development/state-verfications.png)

Consider the following example:

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

In this test, we create a LoanOrder and add a manager discount using the addManagerDiscount method. We then verify that the promotions list has a size of 1, the name of the first promotion contains the manager's UUID, and the discount value is equal to 50. By asserting the expected changes in the state of the LoanOrder object, we can ensure that the addManagerDiscount method works correctly.

This approach is useful when testing methods that return void and when it is necessary to verify the state of the system due to the existing architecture or design decisions.


### Ensuring Communication/Verification

The third testing approach involves verifying the communication between objects. This includes checking the messages sent to other objects or using mocks to verify interactions. By capturing and verifying the expected communication, we can ensure that the correct messages are being exchanged.

![img]({{site.url}}/assets/blog_images/2023-05-12-key-types-of-testing-in-software-development/interaction-verfication.png)

Consider the following example:


```java
@Test
void setUp() {
    customer = buildCustomer();
    eventEmitter = mock(EventEmitter.class);
    customerVerifier = new CustomerVerifier(buildVerifications(eventEmitter));

@Test
void shouldAddManagerPromo() {
    customerVerifier.verify(customer);

    verify(eventEmitter, times(3)).emit(argThat(VerificationEvent::passed));
}
```