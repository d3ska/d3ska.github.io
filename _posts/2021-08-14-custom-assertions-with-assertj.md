---
title: "Custom Assertions with AssertJ"
categories:
  - Blog
tags:
  - Test
  - Custom Assertions
  - Domain Driven Design
  - DDD
---

You might have encountered tests that are difficult to read, especially when the failure messages are unclear. In such cases, consider creating your own specific assertions using the domain model vocabulary. This approach not only makes them easier to read but also helps apply Domain Driven Design's ubiquitous language in tests.

**Implementation**

Let's start with the class we want to test:

![img]({{site.url}}/assets/blog_images/2021-14-08-custom-assertions-with-assertj/assertions1.png)

First, create a class that extends the AbstractAssert class. Its generic type must be completed by adding the class that extends AbstractAssert (in this case, PersonAsserts) and the class we want to test. A constructor as shown below is also needed:

![img]({{site.url}}/assets/blog_images/2021-14-08-custom-assertions-with-assertj/assertions2.png)

Now, we can add our custom assertions along with personalized failure messages:

![img]({{site.url}}/assets/blog_images/2021-14-08-custom-assertions-with-assertj/assertions3.png)

First, check if the object we are testing is not null. Then, implement assertions as shown above. In the failWithMessage method, pass our custom message, making it more readable.

We're almost ready to use our custom assertions. If we have more than one custom assertions class, we can wrap all assertThat methods in a class, providing a static factory method for each of the assertion classes:

![img]({{site.url}}/assets/blog_images/2021-14-08-custom-assertions-with-assertj/assertions4.png)

<br>

Now, let's look at some examples of tests:

![img]({{site.url}}/assets/blog_images/2021-14-08-custom-assertions-with-assertj/assertions5.png)

