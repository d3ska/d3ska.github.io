---
title: "Prefer Composition over Inheritance"
categories:

- Blog
  tags:
- Inheritance
- OOP
- Composition

---

You probably already thought about this, as its quite popular statement.
Let's delve deeper into the subject.
I wrote article that explain what is
composition [here](https://matthewonsoftware.com/blog/association-composition-aggregation/), and about
inheritance [here](https://matthewonsoftware.com/blog/core-concepts-behind-oop/).

<br>

Both concepts are about reusing piece of code. <br><br>
In general, it is generally considered good design to prefer composition over inheritance. This is because composition allows for greater flexibility and reuse of code, as it allows you to combine simple objects to achieve more complex behavior, rather than having to create complex inheritance hierarchies.
<br>

Inheritance can be inflexible because it involves creating a subclass that is tightly coupled to its superclass. This means that any changes to the superclass may require changes to the subclass as well, which can be difficult to manage and maintain as the codebase grows. Additionally, inheritance can make it difficult to reuse code because it can be hard to know what behavior is inherited from the superclass and what behavior is implemented specifically in the subclass.

On the other hand, composition allows you to build complex behavior by composing simple objects, rather than having to create complex inheritance hierarchies. This makes it easier to reuse code and make changes to your codebase because you can easily substitute different objects to achieve the desired behavior.

However, it is worth noting that both inheritance and composition have their own strengths and weaknesses, and it is important to choose the right approach for the specific problem at hand. There is no one-size-fits-all answer, and it is important to consider the specific requirements and constraints of your project when deciding which approach to use.

Worth knowing is that inheritance offer polymorphism out of the box, but in case of composition we don't get it for free but instead our classes are not tightly coupled, but with usage of interfaces we can achieve polymorphisms with composition as well, so then we get low coupling and polymorphism.
We can use table below to visualize the differences between those two concepts. 

<table style="width:100%">
  <tr>
    <th></th>
    <th>Parameter</th>
    <th>Type</th>
  </tr>
  <tr>
    <td>Inheritance</td>
    <td>Extending</td>
    <td>Parent Classes</td>
  </tr>  
  <tr>
    <td>Composition</td>
    <td>Using</td>
    <td>Interfaces</td>
  </tr>
</table>


<br>

Keep in mind that it's not that we should use only composition from now on, most of the time? Yes, but not always.



#### The cons of Composition

* Code Repetition in interface implementations to initialize internal types
* Wrapper methods to expose information from internal types


#### The pros of Composition

* Reduces coupling to re-used code
* Adaptable as new requirements come in


#### Tips for Inheritance

* Design the parent class to be inherited
* Avoid protected member variables with direct access
* Create a protected API from children classes to use
* Mark all other methods as final/sealed/private


#### When to use Inheritance

When we have some boilerplate in some classes, we can move it to the parent class, so other classes can inherit it. 
As you see its scenario where we are not expecting that the model would growth.





