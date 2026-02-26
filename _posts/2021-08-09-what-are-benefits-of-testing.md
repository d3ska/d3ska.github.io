---
title: "What are benefits of testing?"
categories:
  - Blog
tags:
  - Test
  - Test Driven Development
  - TDD
---

Do I really need tests? Isn't it a waste of time? These are questions you might have asked once or still find yourself asking. However, the answer is that tests are not a waste of time, and they provide numerous benefits. Let me explain why.

> **Related posts**: I later wrote a more comprehensive overview covering legacy code, CI/CD, and industry statistics in [Why Are Tests Essential?]({{site.url}}/posts/why-are-tests-essential-comprehensive-look-at-software-testing/). For the business and ROI perspective on testing, see [The Economics of Software Testing]({{site.url}}/posts/economy-of-testing/).

### TDD - Test Driven Development

Test-Driven Development (TDD) is a software development process that relies on software requirements being converted into test cases before the software is fully developed. First, write tests that fail, then code to cover them and make the tests pass, and finally refactor the code if needed.

![img]({{site.url}}/assets/blog_images/2021-09-09-is-it-worth-to-writh-tests/test-driven-development-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2021-09-09-is-it-worth-to-writh-tests/test-driven-development-dark.png){: .dark }

TDD naturally reinforces every benefit listed below. By writing tests first, I am forced to think about design upfront, which leads to cleaner interfaces. The tests themselves become living documentation of the intended behavior. And because TDD catches issues the moment they are introduced, it amplifies the cost savings of shift-left testing.

### Benefits of having tests

* **Better program design and higher code quality**: Tests help reduce code complexity and lead to fewer, more thoughtful dependencies between classes. This results in smaller classes and functions that are easier to read and understand. For instance, when I write a test for a service class and find myself needing to set up ten dependencies, that is an immediate signal that the class is doing too much and should be split. Tests act as the first consumer of my API, so if the test setup feels painful, the production code likely needs a redesign.

* **Detailed project documentation**: Tests serve as excellent documentation for a project. They help new team members dive into the project and understand it more quickly. In many cases, tests may be the only up-to-date source of documentation. A well-named test like `shouldRejectOrderWhenInventoryIsInsufficient` tells a new developer exactly what the business rule is, without them having to read through layers of production code. Unlike wiki pages or README files, tests stay current because they fail when behavior changes.

* **Code flexibility and easier maintenance**: Having tests in place makes it easier to confidently refactor code or add or delete features/parts of the code. This approach helps teams stay focused on Continuous Delivery. I have personally refactored entire modules, renamed classes, and restructured packages knowing that a green test suite meant nothing was broken. Without that safety net, even a small rename can feel risky enough to postpone indefinitely.

* **Save project costs in the long run**: At first, it may seem like tests are a waste of time because they take time to write. However, investing time in testing reduces the chances of future bugs and issues during software development. If issues do occur, they will be discovered more quickly, saving both time and money, and potentially preventing more costly bugs. This approach is called "Shift-left testing," and it allows teams to find and prevent defects early in the software delivery process. A bug caught during development might take minutes to fix, while the same bug found in production can require hours of debugging, hotfix deployments, and customer communication.

![img]({{site.url}}/assets/blog_images/2021-09-09-is-it-worth-to-writh-tests/shift-left-testing-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2021-09-09-is-it-worth-to-writh-tests/shift-left-testing-dark.png){: .dark }

In conclusion, tests are not a waste of time but rather a crucial aspect of software development that leads to better design, documentation, flexibility, and cost savings.


