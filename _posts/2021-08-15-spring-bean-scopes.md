---
title: "Spring Bean Scopes"
categories:
  - Blog
tags:
  - Spring
  - Spring Bean
  - Context
  - Bean Scopes
---

#### Bean Scopes Define the Runtime Context Availability

The Spring framework defines 6 types of bean scopes:

* singleton.
* prototype.
* request.
* session.
* application.
* websocket.

The last four scopes mentioned (request, session, application, and websocket) are only available in a web-aware application.

Additionally, there is the possibility to create custom bean scopes.

### Singleton

When a bean is defined with the singleton scope, the container creates an instance of the bean only once. The framework returns that instance each time the bean is requested by your application code. Singleton is also the default bean scope in Spring if no other scope is defined.

```java
@Bean
public ConnectionFactory connectionFactory(){
    return new ConnectionFactory();
}
```

This would work exactly the same as:

```java
@Bean
@Scope("singleton")
public ConnectionFactory connectionFactory(){
    return new ConnectionFactory();
}
```

We can also use a constant instead of a string value as follows:

```java
@Scope(value = ConfigurableBeanFactory.SCOPE_SINGLETON)
```

Singleton scope is typically used for:
* database connections
* connections with devices using serial port communication
* libraries operating throughout the life of the program, e.g., Log4j
* etc.

### Prototype

When a bean is created with the prototype scope, the framework returns a different instance every time it is requested from the container. The prototype scope is defined by setting the value prototype to the @Scope annotation in the bean definition:

```java
@Bean
@Scope("prototype")
public Car carSingleton(){
    return new Car();
}
```

We can also use a constant like we did for the singleton scope:

```java
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)
```

Prototype scope is typically used for:
* stateful beans that hold conversation or per-request data
* objects that are expensive to keep around and should be garbage-collected after use
* beans injected into different contexts where each consumer needs its own isolated instance

### Request

The request scope defines a single bean with a lifecycle tied to a single HTTP request. For every HTTP request, a new instance of the bean is created from the single bean definition. This scope is only valid in the context of a web-aware Spring ApplicationContext.

```java
@Bean
@Scope(value = WebApplicationContext.SCOPE_REQUEST,
        proxyMode = ScopedProxyMode.TARGET_CLASS)
public UserData requestScopeUserData() {
    return new UserData();
}
```


The **proxyMode** attribute is necessary because there is no active request at the time of the web application context instantiation. Spring creates a proxy to be injected as a dependency and instantiates the target bean when needed in a request.

We can also use a **@RequestScope** composed annotation as a shortcut for the above definition:

```java
@Bean
@RequestScope
public UserData requestScopeUserData() {
    return new UserData();
}
```

Now, let's create a controller that references this bean:

```java
public class SecurityInterceptor {
    @Resource(name = "requestScopeUserData")
    private UserData userData;

}
```

Request scope is typically used for:
* storing per-request audit or logging metadata
* holding user-specific input validation state during a single request
* accumulating request-scoped diagnostics or tracing information

### Session

The session scope defines a single bean with a lifecycle tied to an HTTP Session. Like the request scope, the session scope is applicable to beans in web applications.

```java
@Bean
@Scope(value = WebApplicationContext.SCOPE_SESSION,
        proxyMode = ScopedProxyMode.TARGET_CLASS)
public UserData sessionScopedBean() {
    return new UserData();
}
```

We can define the bean with the session scope in a similar manner:

```java
@Bean
@SessionScope
public UserData sessionScopedBean() {
    return new UserData();
}
```

Controller that references this bean:

```java
public class ScopesController {
    @Resource(name = "sessionScopedBean")
    private UserData userData;

}
```

Session scope is typically used for:
* shopping carts in e-commerce applications
* user preferences or settings that persist across multiple requests within a session
* multi-step form wizards where data must survive between page navigations


### Application

The application scope creates the bean instance for the lifecycle of a ServletContext. This is similar to the singleton scope, but there is an important difference concerning the scope of the bean. It is a singleton per ServletContext, not per Spring ApplicationContext (of which there may be several in any given web application). It is also exposed and therefore visible as a ServletContext attribute.

```java
@Bean
@Scope(value = WebApplicationContext.SCOPE_APPLICATION,
        proxyMode = ScopedProxyMode.TARGET_CLASS)
public UserData applicationScopedBean() {
    return new UserData();
}
```

Or use a special annotation:

```java
@Bean
@ApplicationScope
public UserData applicationScopedBean() {
    return new UserData();
}
```

Controller that references this bean:

```java
public class ScopesController {
    @Resource(name = "applicationScopedBean")
    private UserData userData;

}
```

Application scope is typically used for:
* global configuration or feature flags shared across the entire web application
* application-wide counters or statistics
* shared resources that must be accessible by multiple Spring ApplicationContexts in the same ServletContext


### Websocket

Finally, let's create the bean with the websocket scope:

```java
@Bean
@Scope(scopeName = "websocket",
        proxyMode = ScopedProxyMode.TARGET_CLASS)
public UserData applicationScopedBean() {
    return new UserData();
}
```


The Spring Framework provides a WebSocket API that you can use to write client- and server-side applications that handle WebSocket messages. You can declare a Spring-managed bean in the websocket scope. Those beans live as long as the WebSocket session they belong to.

A new bean instance is created for each WebSocket session, and the bean is destroyed when the session ends.

Websocket scope is typically used for:
* per-connection chat state or message history
* user-specific notification queues tied to a live connection
* real-time collaboration data such as cursor positions or document edits

### Creating Custom Bean Scopes

In addition to the built-in bean scopes provided by Spring, it is also possible to create your own custom bean scopes. This can be useful when you need to manage bean instances according to specific requirements that are not covered by the existing scopes.

To create a custom bean scope, you need to implement the `org.springframework.beans.factory.config.Scope` interface. This interface defines the contract for managing the lifecycle of a bean in the custom scope.

Here is a minimal example of a custom scope that stores beans in a thread-local context:

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

Once the scope class is ready, we need to register it with the Spring container:

```java
@Configuration
public class CustomScopeConfig implements BeanFactoryPostProcessor {

    @Override
    public void postProcessBeanFactory(ConfigurableListableBeanFactory factory) {
        factory.registerScope("thread", new ThreadScope());
    }
}
```

After registration, beans can use the custom scope like any built-in one:

```java
@Component
@Scope("thread")
public class MyThreadScopedBean {
    // ...
}
```
