---
title: "Best Practices for Writing Effective and Reliable Tests"
categories:
  - Testing
  - System Quality
tags:
  - Testing
---

Many teams face a common challenge after introducing the practice of writing tests: builds take up to half an hour, refactoring tests prove to be difficult, and testing consumes too much of the team's time and energy. However, this problem can be solved by adhering to good testability practices.

### The FIRST Principle

The FIRST principle is a set of criteria that good unit tests should meet:

* **Fast**: Quick tests lead to quicker project building times. However, the execution time can vary depending on the type of test. Typically, unit tests are faster than integration tests, which in turn are faster than end-to-end (E2E) tests.

* **Independent**: Tests should not depend on each other. Each test should be able to run alone and in any order. This makes it easier to pinpoint problems when a test fails.

* **Repeatable**: Tests should give the same result each time, no matter how often they are run.

* **Self-Validating**: The test should have a clear result, either positive or negative. There's no need for any additional activities to interpret the result.

* **Timely**: Ideally, tests should be written just before the production code that makes them pass (Test-Driven Development - TDD).


### Clarity in Tests

In the realm of software testing, readability and clarity are crucial to understanding the purpose of each test, the functionality being tested, and the expected outcome. Let's consider an initial version of a test that lacks these elements:

```java
void testVerification() throws IOException {
    Customer correctCustomer = CustomerBuilder.create().build();
    HttpPost httpPost = new HttpPost(LOAN_ORDERS_URI);
    httpPost.setEntity(new StringEntity(objectMapper.writeValueAsString(new LoanOrder(correctCustomer))));
    httpPost.setHeader("Content-type", "application/json");
    HttpResponse postResponse = httpClient.execute(httpPost);
    assertThat(postResponse.getStatusLine().getStatusCode()).isEqualTo(200);
    String loanOrderId = EntityUtils.toString(postResponse.getEntity()).replaceAll("data:", "").trim();
    HttpGet httpGet = new HttpGet(LOAN_ORDERS_URI + "/" + loanOrderId);
    httpGet.setHeader("Content-type", "application/json");
    HttpResponse getResponse = httpClient.execute(httpGet);
    LoanOrder loanOrder = objectMapper.readValue(EntityUtils.toString(getResponse.getEntity()).replaceAll("data:", ""), LoanOrder.class);
    assertThat(loanOrder.getStatus()).isEqualTo(VERIFIED);
}
```

While this test might work perfectly, it's somewhat hard to reason about. It's not immediately clear what the test setup is, what operation is being tested, and what the expected outcome is. This can become problematic as the codebase grows and as more developers interact with the tests.

To address these issues, we can refactor the test to be more readable and clear. **The Given-When-Then practice (Arrange-Act-Assert)**, commonly used in behavior-driven development (BDD), is a brilliant approach to make tests more comprehensible. It separates the test method into three well-defined sections:

* **Given (set up)**: Outlines the preconditions.
* **When (execution)**: Describes the action to be tested.
* **Then (verification)**: Verifies the outcome.

Let's refactor our original test to adhere to the Given-When-Then pattern:

```java
@Test
void testVerification() throws IOException {
    // Given
    Customer correctCustomer = CustomerBuilder.create().build();
    LoanOrder loanOrderRequest = new LoanOrder(correctCustomer);
    HttpPost httpPost = createHttpPost(LOAN_ORDERS_URI, loanOrderRequest);
    
    // When
    HttpResponse postResponse = httpClient.execute(httpPost);
    
    // Then
    assertThat(postResponse.getStatusLine().getStatusCode()).isEqualTo(200);
    
    // Extract loanOrderId from the response
    String loanOrderId = parseLoanOrderIdFromResponse(postResponse);
    
    // When
    HttpResponse getResponse = executeHttpGet(LOAN_ORDERS_URI, loanOrderId);
    
    // Then
    LoanOrder loanOrder = parseLoanOrderFromResponse(getResponse);
    assertThat(loanOrder.getStatus()).isEqualTo(VERIFIED);
}

```

In the realm of software testing, readability and clarity are crucial elements. By breaking down tests into smaller, well-named methods and adhering to the Given-When-Then pattern, we significantly improve the readability and maintenance of our code. This pattern is very effective in breaking down the test into clear, understandable stages. The test itself becomes more focused, making it easier to understand the functionality that's being tested, the initial state, the operation being tested, and the expected outcome.

However, the **Given-When-Then (Arrange-Act-Assert)** approach is just one of many techniques we can use to improve our tests. Other testing patterns and methodologies can provide additional clarity and efficiency, and they are worth considering as part of a robust testing strategy. Here are a few worth mentioning:

* **Assertion Class**: This involves creating a separate class to handle assertions. This makes tests easier to read and allows for better reuse of common assertions.

* **Fixture Setup**: A fixed setup (or 'fixture') is a context where tests run. This can be anything from input data to a mock object or a test double. Clear and consistent setup of fixtures can make tests more reliable and easier to understand.

* **Custom Assertions**: Developing custom assertions for your tests can improve readability and reduce code duplication. This might involve creating a method that takes in parameters and performs an assertion based on those parameters.

* **Parametrized Testing**: This pattern involves parameterizing your tests to run them with different inputs, reducing code duplication and making your tests more flexible and comprehensive.

By incorporating these practices, along with the Given-When-Then pattern, we can make our test suite more efficient, effective, and manageable as part of our development process. The key is to consider the specific needs and challenges of our testing environment and to apply the methods and patterns that provide the most benefit in our context.


### Following Consistent Naming Conventions in Your Project

Adhering to a consistent naming convention across your testing suite is a crucial aspect of maintaining clean and comprehensible code. This practice makes it easier for developers to understand the purpose of a test at a glance and allows for more efficient navigation through the test suite.

Let's consider the naming of test methods. A good test name should clearly articulate what functionality is being tested and the expected outcome. One common convention is to structure the test name like so: **'methodName_Condition_ExpectedBehavior'**.

For example, if you are testing the **'withdraw'** method in a **'BankAccount'** class, a test might be named **'withdraw_InsufficientFunds_ThrowsException'**. This clearly communicates that the **'withdraw'** method, when dealing with a condition of insufficient funds, is expected to throw an exception.

A consistent naming convention also applies to the organization of your test files. Tests should be grouped logically and clearly labeled so that it's straightforward to find the tests related to a specific component. If you're using a tool that supports it, consider structuring your tests to mirror the structure of your application code. This makes it even easier to locate the tests for a particular piece of functionality.

Maintaining consistent naming conventions isn't just about cleanliness—it's about communication. Clear, descriptive names make your tests more understandable to others (and your future self), making your codebase easier to maintain and evolve. By establishing and following these conventions in your project, you can create a more effective and efficient testing environment.


### Avoiding False Negatives in Testing

While clarity in tests is key, it's equally important to trust our tests. False negatives can lead to missed bugs and give a false sense of confidence. This is often due to poorly constructed tests or those that pass without the correct logic.

To counter this, we must ensure our tests pass only when the right logic is in place and all assertions hold true. This not only improves readability, but also enhances test reliability, reinforcing our confidence in the test results.

Consider the following example, where a test passes without any implementation:

```java
public class SimpleVerification implements Verification {
    @Override
    public boolean passes(Person person) {
        // TODO
        // use someLogicResolvingToBoolean(person);
        return false;
    }

    private boolean someLogicResolvingToBoolean(Person person) {
        throw new UnsupportedOperationException("Not yet implemented!");
    }
}

// Test that passes even without implementation
@Test
void shouldPassSimpleVerification() {
    // Given
    Customer customer = buildCustomer();
    CustomerVerifier service = new CustomerVerifier(new TestVerificationService(), buildSimpleVerification(), new TestBadServiceWrapper());
    // When
    CustomerVerificationResult result = service.verifyPerson(customer);
    assertThat(result.getStatus())
        .isEqualTo(CustomerVerificationResult.Status.VERIFICATION_FAILED);
}
```

In this example, the test shouldPassSimpleVerification() passes even without the implementation of the method someLogicResolvingToBoolean(). This leads to a false negative, as the test passes despite the lack of implementation.

**Solutions to Avoid False Negatives**

To avoid these kinds of false negatives, consider the following practices:

* **Check if the test works without your logic**: Make sure your tests fail when the logic they're testing is not implemented. This way, you ensure that the tests are indeed checking the correct logic.

* **Reverse assertions to make sure the test fails**: If you reverse your assertions and the test still passes, then the test is not effective. By ensuring the test fails when assertions are reversed, you confirm the test's reliability.

* **Use a build tool to check if tests are being run:** Sometimes, tests may not run due to configuration issues, leading to a false sense of security. Using a build tool can help ensure that all tests are indeed running.


By ensuring your tests are correctly designed and run as expected, you can minimize the risk of false negatives, increasing the reliability of your software testing practices.


### Avoiding High Cyclomatic Complexity

Cyclomatic complexity is a software metric that quantifies the number of linearly independent paths through a program's source code. In simpler terms, it measures how complex your code is, based on the number of different paths execution could take through it. High cyclomatic complexity means that there are many different paths through the code, which can make the code hard to understand, test, and maintain.

Let's take a look at an example. Consider the following piece of code:

```java
public void process(User user, Order order) {
    if(user != null) {
        if(order != null) {
            if(order.isValid()) {
                if(user.hasPermission("process_order")) {
                    // Process the order...
                }
            }
        }
    }
}

```

This piece of code has a cyclomatic complexity of 4 because there are four potential paths through the code. This is not only hard to read, but also hard to test because we need to create tests for each of these paths.

We can reduce the cyclomatic complexity by inverting the conditions and returning early, like so:

```java
public void process(User user, Order order) {
    if(user == null || order == null || !order.isValid() || !user.hasPermission("process_order")) {
        return;
    }

    // Process the order...
}
```

Now, the cyclomatic complexity is 2, making the code easier to understand and test.

It's essential to keep cyclomatic complexity in check because it directly impacts the understandability, testability, and maintainability of your code. As a general rule, a cyclomatic complexity above 10 is considered high and should be avoided.