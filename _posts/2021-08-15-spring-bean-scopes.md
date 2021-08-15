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

#### The scope defines the runtime context within which the bean instance is available.

The latest version of the Spring framework defines 6 types of scopes:

* singleton.
* prototype.
* request.
* session.
* application.
* websocket.

**The last four scopes mentioned, request, session, application and websocket, are only available in a web-aware application.**

### Singleton 

When you define a bean with the singleton scope, the container will create an instance of the bean only once.

The Framework returns that instance each time the bean is requested by your application code.

It is also a default scope of bean in spring if you didn't define other one.

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/singleton1.png)

So it would work exactly the same as:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/singleton2.png)

We can also use a constant instead of the String value in the following manner:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/singleton3.png)

This solution is especially used for:
* database connections.
* connections with devices that use serial port communication.
* libraries operating throughout the life of the program, e.g. log4net.
* e.t.c.

### Prototype

When you create a bean with the Prototype scope, the framework will return a different instance every time it is requested from the container. It is defined by setting the value prototype to the @Scope annotation in the bean definition:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/prototype1.png)

We can also use a contact like we did for singleton scope:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/prototype2.png)

### Prototype

Scopes a single bean definition to the lifecycle of a single HTTP request; that is each and every HTTP request will have its own instance of a bean created off the back of a single bean definition. Only valid in the context of a web-aware Spring ApplicationContext.

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/request3.png)


The proxyMode attribute is necessary because at the moment of the instantiation of the web application context, there is no active request. Spring creates a proxy to be injected as a dependency, and instantiates the target bean when it is needed in a request.

We can also use a @RequestScope composed annotation that acts as a shortcut for the above definition:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/request1.png)

Now let's create a controller that references this bean:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/request2.png)

### Scope

The session scope defines a single bean definition which lives within the lifecycle of an HTTP Session. Similar to the Request scope, the Session scope is applicable to beans in Web applications.

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/scope1.png)

We can define the bean with the session scope in a similar manner:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/scope2.png)

Controller that references this bean:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/scope3.png)


### Application

The application scope creates the bean instance for the lifecycle of a ServletContext.

This is similar to the singleton scope, but there is a very important difference with regards to the scope of the bean.

It is a singleton per ServletContext, not per Spring 'ApplicationContext' (for which there may be several in any given web application), and it is actually exposed and therefore visible as a ServletContext attribute


![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/application1.png)

Or use special annotation:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/application2.png)

Controller that references this bean:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/application3.png)


### Websocket

Finally, let's create the bean with the websocket scope:

![img]({{site.url}}/assets/blog_images/2021-08-15-spring-bean-scopes/websocket1.png)


The Spring Framework provides a WebSocket API that you can use to write client- and server-side applications that handle WebSocket messages. You can declare a Spring-managed bean in the websocket scope. Those are typically singletons and live longer than any individual WebSocket session.

We can also say that it exhibits singleton behavior, but limited to a WebSocket session only.









