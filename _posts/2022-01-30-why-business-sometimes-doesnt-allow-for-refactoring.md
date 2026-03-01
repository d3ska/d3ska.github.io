---
title: "Why the Business Is Not Eager for Refactoring"
categories:
  - Software Design
tags:
  - Refactoring
  - Communication
  - Technical Debt
  - Software Engineering
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

#### A before/after example

To make this more concrete, consider a team I worked with that maintained an order processing service. The codebase had grown organically over three years with no significant cleanup.

**Before refactoring:**
- Deployment frequency: once every 3 weeks
- Lead time for changes: 14 days on average
- Change failure rate: 30%
- Time to restore service after an incident: 4 hours

The team proposed a two-sprint investment to extract a tangled pricing engine into a clean, well-tested module. They presented these numbers to their product owner alongside the projected improvements.

**After refactoring:**
- Deployment frequency: twice per week
- Lead time for changes: 3 days on average
- Change failure rate: 8%
- Time to restore service: 45 minutes

The improvement in deployment frequency alone meant the business could ship features to customers roughly six times faster. The reduction in change failure rate translated to fewer weekend incidents and lower support costs. Numbers like these are difficult for any stakeholder to argue against.

### When the business still says no

Sometimes you present the data, make a solid case, and the answer is still no. This happens. Budgets are tight, deadlines are real, and not every organization is ready to invest in long-term code health. That does not mean you are powerless.

**The Boy Scout Rule** is the most practical tool here: leave the code a little better than you found it. You don't need permission to rename a confusing variable, extract a small method, or add a missing test while you are already working in that area. These micro-improvements add up over time without requiring a dedicated refactoring sprint.

**Refactor as part of feature work.** When you pick up a new feature, include the necessary cleanup in your estimate. If a feature touches a messy module, cleaning that module is part of delivering the feature well. You are not asking for extra time; you are being honest about what good delivery looks like.

**Build trust through small wins.** Start with a low-risk, high-visibility improvement. Maybe it is a flaky test suite that wastes 30 minutes of everyone's day, or a slow build pipeline. Fix it, measure the improvement, and share it. Once the business sees that your technical proposals deliver real results, they become more willing to fund larger efforts.

### Prevention is better than cure

The best refactoring is the one you never have to do. Large-scale refactoring efforts are usually a symptom of quality degradation that happened gradually over months or years. A few continuous practices can prevent this:

- **Test-Driven Development (TDD)** keeps design pressure constant. Writing tests first forces you to think about interfaces and responsibilities before implementation, which naturally leads to cleaner code.
- **Pair programming** catches design problems in real time. Two people are far less likely to take shortcuts that create future technical debt.
- **Code review** acts as a quality gate. A good review process catches not just bugs but structural issues before they become entrenched in the codebase.

None of these practices eliminate the need for refactoring entirely, but they keep the codebase healthy enough that refactoring stays small and routine rather than becoming a large, scary project that needs executive approval.

So, to summarize, you need to "map" programming language to business language and present the benefits that will result from refactoring. You don't even need to use the word "refactoring." By clearly communicating the advantages of refactoring in terms that resonate with business stakeholders, you can gain their support and make the process more widely accepted.

> **Related posts**: [How to Think and Approach Refactoring?](/posts/general-path-of-refactoring/), [Why Are Tests Essential?](/posts/why-are-tests-essential/), [The Economics of Software Testing](/posts/economy-of-testing/)