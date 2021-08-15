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

The last four scopes mentioned, request, session, application and websocket, are only available in a web-aware application.

<p>&nbsp;</p>

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

