---
title: "Availability"
categories:
  - System Design
tags:
  - Availability
---

#### What is availability?

You can think of availability in the couple of ways. One way to think about it is how resistant a system is to failures
For instance, what happens if a server in your system fails? What happens if your database fails?
Is your system going to be completely go down or is your system still going to be operational?
And this is often described as a system's fault tolerance, how fault-tolerant is a system.

Another way to think about availability is as the percentage of time in a given period of time, like a month or a year
are at least operational enough such that all of its primary functions are satisfied. 

So like I just mentioned availability is a very important thing to think about when you are evaluating a system.
In this day and age especially, most system almost have an implied guarantee of availability.


Imagine that we are dealing with a system that supported airplane software, the software that allows an airplane to function properly.
You could imagine that if that system were to ever go down, when an airplane were flying, that would be absolutely unacceptable. 

Another example may be stock or crypto exchange system, imagine what would happen and what would be a reaction of the customer that 
entrusted that company their money.

So in that types of system, you would expect an extremely high amount of availability, any amount of complete downtime of the system would just be unacceptable.
By the way, here we don't necessarily have to reach so far into life or death examples like airplane software , or into money loosing like trading systems.

You cna take a step back and look at platforms like YouTube. If Twitter ever goes down, that would be terrible, because hundreds of millions of people use Twitter every day.

Or take cloud providers for an instance like AWS, Azure or GCP, if parts of there cloud providers systems ever go down, that can be unfortunate because then it affect all the businesses and customers 
that rely on their services for their own platforms.


As an example, in the summer of 2019, in June or July, Google Cloud Platform had a really bad outage that affected a bunch of its products and services.
This outage lasted at least of a couple of hours. Many or all of the businesses that relied on Google Cloud Platform were affected, e.g. Vimeo.


All that to say that availability matters a lot.

---

#### How to measure availability?

Availability usually is measured as the percentage of a system's uptime in a given year.

So, for instance, if a system is up and operational fo r half an entire year,
then we acn say that that system has 50% availability. Which is really bad. 

When we're dealing with availability, we're usually dealing with very high percentages.

In the industry most services or most systems, aim for really high availability, so we often end up 
measuring availability, not exactly in percentages but in what we call nines. 

Nines are effectively percentage, but they are specifically percentages with the number nine.  For example, if we have a system that has 99% availability,
then we can say that that system has two nines of availability, because literally the number nine appears two times in this percentage, 99,9% would be three nines availability and so forth. 

That's a standard way that people talk about availability in the industry.


Here is the char from Wikipedia, where we can see the bunch of popular availability percentages.

<table style="width:100%">
  <tr>
    <th>Availability</th>
    <th>Downtime per year</th>
    <th>Downtime per month</th>
    <th>Downtime per week</th>
    <th>Downtime per day</th>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>  
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
  <tr>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
    <td></td>
  </tr>
</table>


As you can see, even that 99% availability sounds really great, being down for three and half days, or even more per year,
is still really bad and borderline unacceptable.  
If your system supports life and death scenarios, then this is definitely unacceptable but even for services like Facebook or YouTube, which serves for a billion of users, that downtime is too high. 

Five nines of availability (99.999%) is typically regarded as the gold standard of availability.
If your system has five nines of availability, we would say that it's really highly available system.