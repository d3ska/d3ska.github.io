---
title: "Best Practices for Writing Effective and Reliable Tests"
categories:
  - Testing
tags:
  - Testing
  - Best Practices
  - Test Quality
  - Clean Code
---

Many teams introduce testing and then hit a wall: builds take thirty minutes, tests break on every refactor, and the team spends more time fixing tests than writing features. These problems are not caused by testing itself but by how tests are written. The practices below target exactly these pain points, from keeping tests fast and independent, through making them readable, to ensuring they actually catch bugs.


### The FIRST Principles

The **FIRST** acronym captures five properties that well-written tests should have. Rather than listing them in isolation, I want to show how violating even one of them creates real problems.

**Fast.** Slow tests kill feedback loops. Unit tests should run in milliseconds, integration tests in seconds. When a full suite takes minutes, developers stop running it locally and bugs slip through to CI. The most common culprits are unnecessary database calls, network I/O, and sleeping threads in tests that could use deterministic scheduling instead.

**Independent.** Tests that depend on each other are a debugging nightmare. If test B only passes when test A runs first, a failure in B tells you nothing useful. Each test should set up its own state and tear it down afterwards.

**Repeatable.** A test that passes on Monday and fails on Tuesday without any code change is worse than no test at all. The usual offender is hidden dependency on something external: the current date, a random number, a running service. Consider this example where a discount is tied to the current date:

```java
// Broken: depends on the system clock
public class DiscountService {
    public BigDecimal calculateDiscount(Order order) {
        LocalDate today = LocalDate.now();
        if (today.getMonthValue() == 12) {
            return order.getTotal().multiply(new BigDecimal("0.10"));
        }
        return BigDecimal.ZERO;
    }
}

// This test passes only in December
@Test
void shouldApplyDecemberDiscount() {
    // Given
    Order order = new Order(new BigDecimal("100.00"));
    DiscountService service = new DiscountService();

    // When
    BigDecimal discount = service.calculateDiscount(order);

    // Then
    assertThat(discount).isEqualByComparingTo("10.00");
}
```

The fix is to inject a `Clock` so the test controls time:

```java
public class DiscountService {
    private final Clock clock;

    public DiscountService(Clock clock) {
        this.clock = clock;
    }

    public BigDecimal calculateDiscount(Order order) {
        LocalDate today = LocalDate.now(clock);
        if (today.getMonthValue() == 12) {
            return order.getTotal().multiply(new BigDecimal("0.10"));
        }
        return BigDecimal.ZERO;
    }
}

@Test
void shouldApplyDecemberDiscount() {
    // Given
    Clock decemberClock = Clock.fixed(
        LocalDate.of(2024, 12, 15).atStartOfDay(ZoneId.systemDefault()).toInstant(),
        ZoneId.systemDefault()
    );
    Order order = new Order(new BigDecimal("100.00"));
    DiscountService service = new DiscountService(decemberClock);

    // When
    BigDecimal discount = service.calculateDiscount(order);

    // Then
    assertThat(discount).isEqualByComparingTo("10.00");
}
```

Now the test passes in any month, on any machine.

**Self-Validating.** A test should produce a clear pass or fail. If someone has to open a log file or check a database row manually to decide whether the test passed, it is not a real test.

**Timely.** Writing tests close to (or before) the production code they verify keeps the design testable. Code that is written first and tested later often turns out to be hard to test, which leads to tests being skipped entirely.

These five properties are a useful checklist, but they are not enough on their own. The rest of this post covers the practices that turn these principles into concrete, day-to-day habits.


### Structuring Tests with Given-When-Then

Once tests are fast, independent, and repeatable, the next challenge is making them readable. A test that nobody can understand is a test nobody trusts. **Given-When-Then** (also called Arrange-Act-Assert) is the simplest way to bring structure to a test method. It splits every test into three sections:

* **Given** -- set up the preconditions.
* **When** -- execute the action under test.
* **Then** -- verify the outcome.

Consider this test that is hard to follow at a glance:

```java
@Test
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

The setup, action, and assertion are tangled together. Applying Given-When-Then properly means one cycle per test. Here the original test has two distinct behaviors (creating a loan order and verifying its status), so it should be two tests:

```java
@Test
void shouldCreateLoanOrder() throws IOException {
    // Given
    Customer customer = CustomerBuilder.create().build();
    LoanOrder loanOrderRequest = new LoanOrder(customer);

    // When
    HttpResponse response = postLoanOrder(loanOrderRequest);

    // Then
    assertThat(response.getStatusLine().getStatusCode()).isEqualTo(200);
    assertThat(parseLoanOrderIdFromResponse(response)).isNotBlank();
}

@Test
void shouldVerifyCreatedLoanOrder() throws IOException {
    // Given
    String loanOrderId = createLoanOrderAndReturnId(CustomerBuilder.create().build());

    // When
    HttpResponse response = getLoanOrder(loanOrderId);

    // Then
    LoanOrder loanOrder = parseLoanOrderFromResponse(response);
    assertThat(loanOrder.getStatus()).isEqualTo(VERIFIED);
}
```

Notice that `shouldVerifyCreatedLoanOrder` uses a helper method `createLoanOrderAndReturnId` in its Given section. This is not the same as depending on another test. The helper is called within the test itself, so each test still controls its own setup and can run independently, in any order. The Given section is allowed to be complex. What matters is that each test has exactly one When and one Then.


### Useful Testing Patterns

Given-When-Then handles structure, but there are a few patterns that help with the content of each section.

**Custom Assertions** let you replace low-level field-by-field checks with domain-specific, readable assertions. Instead of writing five `assertThat` calls to verify individual fields of a `LoanOrder`, you write `assertThat(loanOrder).hasStatus(VERIFIED).belongsTo(customer)`. This improves readability and reduces duplication across tests. I covered this in detail, including how to build custom assertion classes with AssertJ, in a [dedicated post](/posts/custom-assertions-with-assertj/).

**Fixture Setup** means extracting common test data construction into helper methods or builders, so the Given section stays focused on what is unique to each test case. The `CustomerBuilder.create().build()` pattern from the examples above is a simple version of this.

**Data-Driven Testing** (parameterized tests in JUnit 5) lets you run the same test logic against many inputs without duplicating the test method. This is especially useful for validation rules, parsers, and anything with clear input/output pairs:

```java
@ParameterizedTest
@CsvSource({
    "100.00, 12, 10.00",
    "100.00,  6,  0.00",
    "250.00, 12, 25.00"
})
void shouldCalculateDiscount(String total, int month, String expectedDiscount) {
    // Given
    Clock clock = clockFixedToMonth(month);
    Order order = new Order(new BigDecimal(total));
    DiscountService service = new DiscountService(clock);

    // When
    BigDecimal discount = service.calculateDiscount(order);

    // Then
    assertThat(discount).isEqualByComparingTo(expectedDiscount);
}
```

Three test cases, zero duplication.


### Following Consistent Naming Conventions

Readable tests need readable names. A good test name answers three questions: what is being tested, under what conditions, and what the expected outcome is. One convention that works well is **methodName_condition_expectedBehavior**:

* `withdraw_insufficientFunds_throwsException`
* `calculateDiscount_decemberOrder_returnsTenPercent`
* `verifyPerson_missingIdentity_returnsVerificationFailed`

The exact format matters less than consistency. Pick a convention, document it, and follow it across the entire project. When every test in a suite follows the same pattern, scanning a list of failures in CI immediately tells you what went wrong and where.

Test file organization follows a similar principle. Mirror the structure of your application code so that finding the tests for a given class is trivial. If `DiscountService` lives in `com.example.billing`, its tests belong in `com.example.billing.DiscountServiceTest`.


### Avoiding False Negatives

A test suite you cannot trust is worse than no test suite at all. A **false negative** occurs when a test passes even though the code under test is broken. The test says "no bug found," but that signal is wrong. False negatives erode confidence quietly: every bug that slips through makes the team trust the suite a little less, until eventually nobody pays attention to green builds.

Consider this example:

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

@Test
void shouldPassSimpleVerification() {
    // Given
    Customer customer = buildCustomer();
    CustomerVerifier service = new CustomerVerifier(
        new TestVerificationService(), buildSimpleVerification(), new TestBadServiceWrapper()
    );

    // When
    CustomerVerificationResult result = service.verifyPerson(customer);

    // Then
    assertThat(result.getStatus())
        .isEqualTo(CustomerVerificationResult.Status.VERIFICATION_FAILED);
}
```

The `passes` method is not implemented. It blindly returns `false`, and the test asserts on `VERIFICATION_FAILED`, which happens to match. The test is green, but it is not testing anything meaningful.

Three practices help prevent this:

* **Check that the test fails without your logic.** Before calling a test "done," comment out or stub the production code. If the test still passes, it is not exercising what you think it is.

* **Reverse your assertions.** Flip `isEqualTo(VERIFIED)` to `isNotEqualTo(VERIFIED)`. If the test still passes, something is wrong.

* **Verify that tests actually run.** Build tool misconfigurations (wrong naming patterns, excluded packages) can silently skip tests. Check your CI logs to confirm the test count matches expectations.


### Keeping Cyclomatic Complexity Low

All of the practices above become harder to apply when the production code itself is tangled. **Cyclomatic complexity** measures the number of independent paths through a piece of code. High complexity means more branches to test, more edge cases to cover, and more chances for a test to miss a path.

Here is a method with deeply nested conditions:

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

This has a cyclomatic complexity of 5. Each nesting level adds another path you need to cover with a test. Inverting the conditions and returning early collapses the structure:

```java
public void process(User user, Order order) {
    if(user == null || order == null || !order.isValid() || !user.hasPermission("process_order")) {
        return;
    }

    // Process the order...
}
```

Now the complexity is 2. The code is easier to read, and it is obvious which test cases you need: one where the guard clause triggers and one where processing happens. As a rule of thumb, a cyclomatic complexity above 10 in a single method is a signal that the method is doing too much and should be broken apart.

Keeping production code simple is not a separate concern from testing. It is part of the same discipline. Simple code is testable code, and testable code tends to be better code.


> **Related posts**: [Custom Assertions with AssertJ](/posts/custom-assertions-with-assertj/) -- [Key Types of Testing in Software Development](/posts/key-types-of-testing-in-software-development/) -- [Schools of Unit Testing](/posts/schools-of-unit-tests/) -- [The Economics of Software Testing](/posts/economy-of-testing/)
