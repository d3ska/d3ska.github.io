---
title: "Spring Bean Scopes"
categories:
  - Java
tags:
  - Spring
  - Spring Bean
  - Bean Scopes
  - Java
---

Imagine you inject the same bean into two controllers. Do they share state? If one request modifies a field on the bean, does the next request see that change? The answer depends entirely on the bean's **scope**.

Spring creates and manages beans for you, but you need to tell it how long each bean should live and how many instances should exist. That is what scopes are for. Get it wrong and you end up with thread-safety bugs in production, leaked session data between users, or objects that silently share mutable state.

The Spring Framework defines six built-in scopes:

* singleton
* prototype
* request
* session
* application
* websocket

The last four (request, session, application, and websocket) are only available in web-aware application contexts. On top of these, you can also create custom scopes for specialized needs.

> **Related posts**: [Singleton Pattern](/posts/singleton-pattern/) and [Inversion of Control and Dependency Injection](/posts/inversion-of-control-and-the-dependency-injection/)

### Singleton

**Singleton** is the default scope. When a bean is defined with this scope, the container creates exactly one instance and returns that same instance every time the bean is requested. If you don't specify a scope, singleton is what you get.

```java
@Bean
public ConnectionFactory connectionFactory() {
    return new ConnectionFactory();
}
```

This behaves identically to the explicit version:

```java
@Bean
@Scope("singleton")
public ConnectionFactory connectionFactory() {
    return new ConnectionFactory();
}
```

You can also use the constant `ConfigurableBeanFactory.SCOPE_SINGLETON` instead of the string literal:

```java
@Scope(value = ConfigurableBeanFactory.SCOPE_SINGLETON)
```

Because there is only one instance shared across the entire application context, singleton beans must be stateless or thread-safe. This makes them a natural fit for services, repositories, and other objects that don't hold per-request or per-user data.

### Prototype

With the **prototype** scope, Spring creates a brand new instance every time the bean is requested from the container. Once the bean is handed off, Spring no longer manages its lifecycle, so you are responsible for cleanup.

```java
@Bean
@Scope("prototype")
public TaskProcessor taskProcessor() {
    return new TaskProcessor();
}
```

The constant form works the same way:

```java
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
```

Prototype scope is a good choice when each consumer needs its own isolated instance. Think of stateful objects that accumulate per-request data, or short-lived workers that should be garbage-collected after use. Because each injection gets a fresh object, you avoid the thread-safety headaches of shared mutable state.

### Request

The **request** scope ties a bean's lifecycle to a single HTTP request. A fresh instance is created for every incoming request and destroyed when the request completes.

```java
@Bean
@Scope(value = WebApplicationContext.SCOPE_REQUEST,
        proxyMode = ScopedProxyMode.TARGET_CLASS)
public AuditContext requestScopedAuditContext() {
    return new AuditContext();
}
```

#### Why proxyMode is necessary

You might wonder why `proxyMode` is needed here. The issue is a lifecycle mismatch. A singleton controller is created once at application startup. A request-scoped bean, on the other hand, does not exist yet at that point because there is no active HTTP request. If Spring tried to inject a real request-scoped bean into the controller during startup, it would fail with an exception.

The solution is a **proxy**. Spring injects a lightweight stand-in object that looks like the real bean. When your code actually calls a method on it during a request, the proxy delegates to the real request-scoped instance that is bound to the current HTTP request. This way the singleton controller can hold a reference to something, and the right instance is resolved lazily at runtime.

You can simplify the definition with the `@RequestScope` composed annotation, which sets the proxy mode for you:

```java
@Bean
@RequestScope
public AuditContext requestScopedAuditContext() {
    return new AuditContext();
}
```

Injecting it into a controller with constructor injection looks like this:

```java
@RestController
public class OrderController {

    private final AuditContext auditContext;

    @Autowired
    public OrderController(AuditContext auditContext) {
        this.auditContext = auditContext;
    }
}
```

Request scope is typically used for storing per-request audit or logging metadata, holding user-specific validation state during a single request, or accumulating request-scoped diagnostics and tracing information.

### Session

The **session** scope ties a bean to an HTTP session. The same instance is returned for all requests within the same session, and a new instance is created for each new session.

```java
@Bean
@Scope(value = WebApplicationContext.SCOPE_SESSION,
        proxyMode = ScopedProxyMode.TARGET_CLASS)
public ShoppingCart sessionScopedCart() {
    return new ShoppingCart();
}
```

The same lifecycle mismatch applies here. The singleton controller outlives any individual session, so Spring needs a proxy for the same reason described in the request scope section. The shorthand annotation works here too:

```java
@Bean
@SessionScope
public ShoppingCart sessionScopedCart() {
    return new ShoppingCart();
}
```

Injecting it with constructor injection:

```java
@RestController
public class CartController {

    private final ShoppingCart cart;

    @Autowired
    public CartController(ShoppingCart cart) {
        this.cart = cart;
    }
}
```

Session scope is the classic choice for shopping carts, user preferences that persist across multiple requests, and multi-step form wizards where data must survive between page navigations.

### Application

The **application** scope creates one bean instance for the lifecycle of a `ServletContext`. This sounds similar to singleton, but there is an important distinction. A singleton bean is scoped per Spring `ApplicationContext`, and a single web application can contain multiple application contexts. An application-scoped bean is a singleton per `ServletContext`, meaning it is shared across all Spring `ApplicationContext` instances running in the same web application. It is also exposed as a `ServletContext` attribute.

```java
@Bean
@Scope(value = WebApplicationContext.SCOPE_APPLICATION,
        proxyMode = ScopedProxyMode.TARGET_CLASS)
public FeatureFlags applicationScopedFlags() {
    return new FeatureFlags();
}
```

Or with the shorthand annotation:

```java
@Bean
@ApplicationScope
public FeatureFlags applicationScopedFlags() {
    return new FeatureFlags();
}
```

Injecting it:

```java
@RestController
public class FeatureController {

    private final FeatureFlags featureFlags;

    @Autowired
    public FeatureController(FeatureFlags featureFlags) {
        this.featureFlags = featureFlags;
    }
}
```

Application scope is typically used for global configuration, feature flags, or shared resources that need to be accessible across multiple Spring application contexts within the same `ServletContext`.

### WebSocket

The **websocket** scope ties a bean to the lifecycle of a WebSocket session. A new instance is created for each WebSocket connection and destroyed when the session ends.

```java
@Bean
@Scope(scopeName = "websocket",
        proxyMode = ScopedProxyMode.TARGET_CLASS)
public ChatSession websocketScopedChat() {
    return new ChatSession();
}
```

This scope is useful for per-connection chat state, user-specific notification queues tied to a live connection, or real-time collaboration data such as cursor positions or document edits.

### Quick Reference

Here is a summary table to help you pick the right scope:

| Scope | Instances | Thread-safe? | Typical use case |
|---|---|---|---|
| singleton | 1 per ApplicationContext | No (shared) | Stateless services, repositories |
| prototype | New per injection | Yes (not shared) | Stateful, short-lived objects |
| request | 1 per HTTP request | Yes (request-bound) | Per-request audit data, validation state |
| session | 1 per HTTP session | Depends | Shopping carts, user preferences |
| application | 1 per ServletContext | No (shared) | Global config, feature flags |
| websocket | 1 per WebSocket session | Depends | Chat state, connection-specific data |

### Creating Custom Bean Scopes

The six built-in scopes cover the most common scenarios, but sometimes you need something more specific. Spring allows you to define your own scope by implementing the `org.springframework.beans.factory.config.Scope` interface.

Here is a practical example: a **thread scope** that gives each thread its own instance of a bean. This can be useful in batch processing or scheduled tasks where each thread needs isolated state.

```java
public class ThreadScope implements Scope {

    private final ThreadLocal<Map<String, Object>> threadLocal =
            ThreadLocal.withInitial(HashMap::new);

    @Override
    public Object get(String name, ObjectFactory<?> objectFactory) {
        Map<String, Object> scope = threadLocal.get();
        return scope.computeIfAbsent(name, k -> objectFactory.getObject());
    }

    @Override
    public Object remove(String name) {
        return threadLocal.get().remove(name);
    }

    @Override
    public void registerDestructionCallback(String name, Runnable callback) {
        // Registration of destruction callbacks can be implemented if needed
    }

    @Override
    public Object resolveContextualObject(String key) {
        return null;
    }

    @Override
    public String getConversationId() {
        return String.valueOf(Thread.currentThread().getId());
    }
}
```

The `get` method is the heart of it. When Spring needs a bean in this scope, it checks the current thread's local map. If the bean already exists for this thread, it returns it. Otherwise, it calls the `ObjectFactory` to create a new one and stores it.

Once the scope class is ready, you register it with the Spring container using a `BeanFactoryPostProcessor`:

```java
@Configuration
public class CustomScopeConfig implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory factory) {
        factory.registerScope("thread", new ThreadScope());
    }
}
```

After registration, you can use the custom scope like any built-in one:

```java
@Component
@Scope("thread")
public class BatchJobContext {
    // Each thread gets its own instance
}
```
