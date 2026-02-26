---
title: "Difference between @Mock and @MockBean"
categories:
  - Blog
tags:
  - Testing
  - Mocks
---

#### Introduction

Both `@Mock` and `@MockBean` create mock objects, but they operate in fundamentally different contexts. `@Mock` is a pure Mockito annotation that works without Spring, while `@MockBean` is a Spring Boot annotation that injects a mock into the Spring application context. Choosing the wrong one leads to either a test that is slower than it needs to be or one that does not test what I intended.

To make the distinction concrete, I will use a simple `OrderService` that depends on an `OrderRepository`:

```java
@Service
public class OrderService {

    private final OrderRepository orderRepository;

    public OrderService(OrderRepository orderRepository) {
        this.orderRepository = orderRepository;
    }

    public Order findById(Long id) {
        return orderRepository.findById(id)
                .orElseThrow(() -> new OrderNotFoundException(id));
    }

    public Order place(Order order) {
        order.setStatus(OrderStatus.PLACED);
        return orderRepository.save(order);
    }
}
```

<br>

#### @Mock

The `@Mock` annotation is interchangeable with the `Mockito.mock()` method. I can mock either a class or an interface. To activate the annotation, I put `@ExtendWith(MockitoExtension.class)` above the test class, or invoke `MockitoAnnotations.openMocks(this)` in the setup block. (Note: the older `initMocks()` method has been deprecated since Mockito 2.x; prefer `openMocks()` instead.)

The benefit of using annotations is readability, and `@Mock` also makes failures easier to diagnose -- the field name appears in the failure message. Combined with `@InjectMocks`, it eliminates most manual wiring.

```java
@ExtendWith(MockitoExtension.class)
class OrderServiceTest {

    @Mock
    private OrderRepository orderRepository;

    @InjectMocks
    private OrderService orderService;

    @Test
    void shouldReturnOrderWhenFound() {
        // given
        Order order = new Order(1L, OrderStatus.PLACED);
        when(orderRepository.findById(1L)).thenReturn(Optional.of(order));

        // when
        Order result = orderService.findById(1L);

        // then
        assertThat(result.getId()).isEqualTo(1L);
        assertThat(result.getStatus()).isEqualTo(OrderStatus.PLACED);
    }

    @Test
    void shouldThrowWhenOrderNotFound() {
        // given
        when(orderRepository.findById(99L)).thenReturn(Optional.empty());

        // when & then
        assertThrows(OrderNotFoundException.class,
                () -> orderService.findById(99L));
    }
}
```

No Spring context is loaded here. Mockito instantiates `OrderService` directly, injecting the mock `OrderRepository` through the constructor. The test runs in milliseconds.

<br>

#### @MockBean

`@MockBean` is a Spring Boot annotation (provided by the `spring-boot-test` module, not core Spring Framework). It replaces an existing bean of the same type in the Spring application context with a Mockito mock, or adds a new one if none of that type exists.

This annotation is useful in integration tests where I want the full Spring wiring -- transaction management, AOP proxies, security filters -- but need to replace one specific dependency.

```java
@SpringBootTest
class OrderServiceIntegrationTest {

    @Autowired
    private OrderService orderService;

    @MockBean
    private OrderRepository orderRepository;

    @Test
    void shouldReturnOrderWhenFound() {
        // given
        Order order = new Order(1L, OrderStatus.PLACED);
        when(orderRepository.findById(1L)).thenReturn(Optional.of(order));

        // when
        Order result = orderService.findById(1L);

        // then
        assertThat(result.getId()).isEqualTo(1L);
        assertThat(result.getStatus()).isEqualTo(OrderStatus.PLACED);
    }

    @Test
    void shouldThrowWhenOrderNotFound() {
        // given
        when(orderRepository.findById(99L)).thenReturn(Optional.empty());

        // when & then
        assertThrows(OrderNotFoundException.class,
                () -> orderService.findById(99L));
    }
}
```

The test logic looks almost identical, but the execution is very different. Spring boots the entire application context, creates `OrderService` through its normal bean lifecycle, and the `@MockBean` annotation swaps the real `OrderRepository` with a mock inside that context. This means Spring's `@Transactional` boundaries, validation, and any AOP advice around `OrderService` are all active.

Keep in mind that `@MockBean` causes the Spring application context to be reloaded (or a new context to be created) when the set of mocked beans differs between test classes. In large projects, this can noticeably slow down the test suite. Where possible, I try to keep the same `@MockBean` configuration across test classes to maximize context caching.

<br>

#### Side-by-Side Comparison

| | `@Mock` | `@MockBean` |
|---|---------|-------------|
| **Library** | Mockito (`mockito-junit-jupiter`) | Spring Boot (`spring-boot-test`) |
| **Requires** | `@ExtendWith(MockitoExtension.class)` | `@SpringBootTest` (or `@WebMvcTest`, etc.) |
| **Application context** | None -- plain Java | Full Spring context loaded |
| **Bean replacement** | No context, no beans | Replaces or adds a bean in the context |
| **Speed** | Milliseconds | Seconds (context startup) |
| **Wiring** | `@InjectMocks` (constructor injection) | Spring's dependency injection |
| **AOP / Transactions** | Not active | Active |
| **Best for** | Unit tests | Integration tests |

<br>

#### When to Use Which

**Use `@Mock` when:**
- I am testing a single class in isolation (unit test)
- I do not need Spring's dependency injection, transactions, or AOP
- I want the fastest possible test execution
- I am following the London school of unit testing (mock all collaborators)

**Use `@MockBean` when:**
- I need the Spring application context (integration test)
- I want to verify behavior that depends on Spring features (transactions, caching, security, etc.)
- I need to replace a specific bean while keeping the rest of the real application wiring
- I am testing a `@RestController` with `@WebMvcTest` and need to mock a service layer bean

**A common mistake** I have seen is using `@MockBean` in tests that do not actually need the Spring context. This turns what should be a fast unit test into a slow integration test for no benefit. As a rule of thumb: if the test class does not have `@SpringBootTest`, `@WebMvcTest`, `@DataJpaTest`, or a similar Spring test annotation, I use `@Mock`.

<br>

\* Both annotations are available in JUnit 5. `@ExtendWith(MockitoExtension.class)` comes from the Mockito library (`mockito-junit-jupiter`), while `@ExtendWith(SpringExtension.class)` comes from the Spring Test module. `@SpringBootTest` already includes `@ExtendWith(SpringExtension.class)` implicitly.




