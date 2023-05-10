---
title: "Defining Software Architecture"
categories:
  - System Design
tags:
  - System Design
  - Software Architecture
---

The software community has been engaged in a longstanding debate over how to define "architecture." Some believe it refers to the underlying structure of a system or how the most important components are interconnected at a high level.

Martin Fowler shared the details of the conversation with Ralph Johnson.
> ... who questioned this phrasing, arguing that there was no objective way to define what was fundamental, or high level and that a better view of architecture was the shared understanding that the expert developers have of the system design.
> A second common style of definition for architecture is that it's “the design decisions that need to be made early in a project”, but Ralph complained about this too, saying that it was more like the decisions you wish you could get right early in a project.
> His conclusion was that "Architecture is about the important stuff. Whatever that is."
> <br>
> <cite>The article can be found [here](https://martinfowler.com/architecture/).</cite>


However, we can try to make this topic less abstract and define what a software architecture consists of. <br>
Architecture consists of the structure combined with architecture characteristics ("-ilities"), architecture decisions, and design principles.
<br>

### Structure

The term "architecture" in software development refers to the particular style or styles of architecture utilized in a system, such as microservices, monolith, layered, or microkernel. 
However, solely describing the architecture in terms of its structure does not provide a complete understanding of it. 
For instance, if someone states that a system is built on a microservices architecture, they are only referring to the system's structure, rather than its entire architecture, which encompasses also factors such as architecture characteristics, design principles, and architecture decisions. 
A full understanding of a system's architecture requires knowledge of these additional elements.
<br>

### Architecture characteristics 

Another way to define software architecture is through non-functional requirements, known as architecture characteristics. 
These characteristics outline the criteria for the success of a system, which are independent of its functionality. 
Notably, the characteristics do not rely on knowledge of the system's functionality but are essential for its proper operation. 
Examples of architecture characteristics include:
- Availability 
- Reliability
- Testability
- Scalability
- Security
- Agility
- Fault Tolerance
- Elasticity 
- Recoverability
- Performance
- Deployability
- Learnability

So basically the "-ilities".
<br>

### Architecture decisions

Architecture decisions establish the guidelines for constructing a system. An architect may, for instance, specify that only the business and services layers in a layered architecture can access the database or mandate specific data formats. 
These decisions impose limitations on the system and guide development teams on what is permissible and what is not. 
Nonetheless, a variance mechanism exists that allows for the deviation from a particular architecture decision if it cannot be implemented in a certain part of the system due to various constraints or conditions.
<br>

### Design principles

Design principles are not as strict as architecture decisions, as they serve as guidelines rather than strict rules. 
A design principle might recommend that development teams use asynchronous messaging to improve performance when implementing microservices architecture. 
In contrast, architecture decisions (rules) cannot possibly account for every circumstance or possibility when communicating between services, so a design principles can provide guidance on the preferred approach, enabling developers to make more informed decisions.
<br>

The topics described above have been comprehensively covered in the book
**"Fundamentals of Software Architecture by Mark Richards and Neal Ford (O'Reilly)."**
