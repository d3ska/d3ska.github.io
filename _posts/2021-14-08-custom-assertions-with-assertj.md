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

You may have contact with test that are not too readable, as the communicates when they fail.
In that case you should take in consideration craeting your own specific assertions and use the domain model vocabulary, which will make them easier to read.
It's also a way to apply Domain Driven Design ubiquitous language in tests.

**Implementation**

The class that we would like to test.

![img]({{site.url}}/assets/blog_images/2021-14-08-custom-assertions-with-assertj/assertions1.png)

First, we need to create a class that extends the AbstractAssert class. Its generic type must be completed by adding the class that we created that extends AbstractAssert (in this case PersonAsserts) and the class we want to test.
Constructor as below is also needed.

![img]({{site.url}}/assets/blog_images/2021-14-08-custom-assertions-with-assertj/assertions2.png)

Now we can add our own assertions, with own fail messages.

![img]({{site.url}}/assets/blog_images/2021-14-08-custom-assertions-with-assertj/assertions3.png)

First we checked that object that we are testing is notNull and then we implement assertions which looks like above. in the method failWithMessage we passed our custom message, that will be readable.

It is almost ready to use, if we would have more than one customer assertions class, we may wrap all assertThat methods in a class, providing a static factory method for each of the assertion classes:

![img]({{site.url}}/assets/blog_images/2021-14-08-custom-assertions-with-assertj/assertions4.png)

Lets look at an examples of tests.

![img]({{site.url}}/assets/blog_images/2021-14-08-custom-assertions-with-assertj/assertions5.png)

