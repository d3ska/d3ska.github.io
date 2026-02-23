---
title: "Why business is not eager for refactoring?"
categories:
  - Blog
tags:
  - Refactoring
  - Communication
  - Business
---

### Refactoring in a Business Context

I have often encountered opinions that claim businesses (by businesses, I mean companies) don't allow programming teams to refactor or spend time polishing code.

As the definition suggests, refactoring involves changing existing code without altering observable behavior. Think about it and put yourself in the shoes of a businessperson, who usually doesn't think the same way as programmers do. They might ask:

"Why should we pay for something that we don't see the results of? Did they do something wrong before and now have to fix it? Is it a new feature? What's the benefit of it?" The key to success lies in answering these questions.

Most programmers aren't the best at soft skills, but you must present refactoring as something from which the business will benefit. And that's true.

Keep in mind what you want to optimize. Identify some indicators that measure it and predict how they will change after refactoring.

It could be the time needed for deployment, providing new features more quickly, or having a much faster software, allowing employees to accomplish more work in the same time.

These scenarios are just examples, but I hope you get the point; for all of the above, the business would be happy.

### Concrete metrics that speak business language

To make this tangible, here are metrics I have found effective when presenting a refactoring case to stakeholders:

- **Lead time for changes:** Track how long it takes from a code commit to production deployment. If your team's lead time has crept from days to weeks because the codebase is difficult to work with, that is a number a business person understands. After a targeted refactoring, you can show the trend reversing -- "We used to ship a feature in 3 weeks; after restructuring the payment module, we shipped the next feature in 5 days."

- **Deployment frequency:** A tangled codebase often leads to fewer, riskier deployments. If your team deploys once a month in large, scary batches instead of multiple times per week, that directly translates to slower time-to-market. Presenting "we want to move from monthly releases to weekly releases" is a statement any product owner can appreciate.

- **Change failure rate:** Track the percentage of deployments that cause an incident or require a hotfix. A high change failure rate is a direct cost -- incident response, customer impact, lost trust. A refactoring that reduces this rate from 25% to 5% has a clear dollar value.

- **Bug density in a specific module:** If a particular area of the system generates a disproportionate number of production bugs, present the data. "This module accounts for 40% of our production incidents despite being only 10% of the codebase" makes a compelling case for investment.

These metrics come from the well-known DORA (DevOps Research and Assessment) framework, and they bridge the gap between engineering concerns and business outcomes.

So, to summarize, you need to "map" programming language to business language and present the benefits that will result from refactoring. You don't even need to use the word "refactoring." By clearly communicating the advantages of refactoring in terms that resonate with business stakeholders, you can gain their support and make the process more widely accepted.