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

The latest version of the Spring framework defines 6 types of bean scopes:

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

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/singleton1.png)

This would work exactly the same as:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/singleton2.png)

We can also use a constant instead of a string value as follows:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/singleton3.png)

Singleton scope is typically used for:
* database connections
* connections with devices using serial port communication
* libraries operating throughout the life of the program, e.g., log4net
* etc.

### Prototype

When a bean is created with the prototype scope, the framework returns a different instance every time it is requested from the container. The prototype scope is defined by setting the value prototype to the @Scope annotation in the bean definition:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/prototype1.png)

We can also use a constant like we did for the singleton scope:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/prototype2.png)

### Request

The request scope defines a single bean with a lifecycle tied to a single HTTP request. For every HTTP request, a new instance of the bean is created from the single bean definition. This scope is only valid in the context of a web-aware Spring ApplicationContext.

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/request3.png)


The **proxyMode** attribute is necessary because there is no active request at the time of the web application context instantiation. Spring creates a proxy to be injected as a dependency and instantiates the target bean when needed in a request.

We can also use a **@RequestScope** composed annotation as a shortcut for the above definition:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/request1.png)

Now, let's create a controller that references this bean:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/request2.png)

### Session

The session scope defines a single bean with a lifecycle tied to an HTTP Session. Like the request scope, the session scope is applicable to beans in web applications.

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/scope1.png)

We can define the bean with the session scope in a similar manner:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/scope2.png)

Controller that references this bean:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/scope3.png)


### Application

The application scope creates the bean instance for the lifecycle of a ServletContext. This is similar to the singleton scope, but there is an important difference concerning the scope of the bean. It is a singleton per ServletContext, not per Spring 'ApplicationContext' (of which there may be several in any given web application), and it is actually exposed and therefore visible as a ServletContext attribute.

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/application1.png)

Or use a special annotation:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/application2.png)

Controller that references this bean:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/application3.png)


### Websocket

Finally, let's create the bean with the websocket scope:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/websocket1.png)


The Spring Framework provides a WebSocket API that you can use to write client- and server-side applications that handle WebSocket messages. You can declare a Spring-managed bean in the websocket scope. Those beans are typically singletons and live longer than any individual WebSocket session.

In other words, the websocket scope exhibits singleton behavior, but it is limited to a WebSocket session only.

### Creating Custom Bean Scopes

In addition to the built-in bean scopes provided by Spring, it is also possible to create your own custom bean scopes. This can be useful when you need to manage bean instances according to specific requirements that are not covered by the existing scopes.

To create a custom bean scope, you need to implement the org.springframework.beans.factory.config.Scope interface. This interface defines the contract for managing the lifecycle of a bean in the custom scope.








