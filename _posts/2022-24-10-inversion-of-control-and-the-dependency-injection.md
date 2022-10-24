---
title: "Inversion of Control and Dependency Injection"
categories:
  - Blog
tags:
  - IoC and DI
  - Design 
  - Application Architecture
---

#### Inversion of Control and Dependency Injection

I believe that many of you, as me also learn the fastest through an examples. 
Let's consider a simple example. Imagine a program that is managing songs, you can search for some, read details and some interesting facts about it.
There would probably a class that would search for those songs in some repository or ask some external provider for example.

```java
class SongLister {
    
    public final SpotifySongsProvider provider;
    
    public Collection<Song> songsByArtist(String artistName){
       return provider.findSongsByArtist(artistName);
    }
}
```
But the above implementation is pretty naive. Why it's so?
We are using concrete implementation of some finder to get songs. 
In current state, it's highly coupled, and it's not good.

I want the method songsByArtist to be completely independent of the provider.
In other words, 'WHAT' should not be limited by 'HOW' we are doing something. 
It should be just an inner implementation detail, an abstraction. 

To achieve it, I will extract the interface, which defined proper API contract.

```java 
public interface SongsProvider {
    List<Song> findSongsByArtist(String artistName);
}
```

Now it is maybe decoupled, thanks to the interface. 
But still we need to somehow make a concrete implementation to work with SongLister class.
Firstly let's try with creating a concrete implementation in the constructor of SongLister class.

```java
class SongLister {

    private SongsProvider provider;

    public MovieLister() {
        provider = new SpotifyProvider();
    }
}
```
Now it is configurabish at most I would say, as now we depend on both, the concrete implementation and a SongsProvider interface.

![img]({{site.url}}/assets/blog_images/2022-24-10-inversion-of-control-and-the-dependency-injection/conrete-impl-constructor-initializing.jpg)

The desire situation is where we would depend just on the interface.
It's highly possible that we would have several SongsProvider implementation which we would like to use with but independent of the MovieLister class, 
we may sign a contract with different provider, or just take those songs from some hosted file, for purpose of the local development.

We would like to have them just to be a dependencies/ plugins. But by direct instantiating it we are loosing that possibility, to have finder as a dependency/ plugin. 

<br>

#### What is Inversion of Control?

Inversion of Control is a principle in software engineering which transfers the control of objects or portions of a program to a container or framework. 
This give as a few advantages such as:

* decoupling between 'WHAT' and 'HOW' something is achieved
* possibility of configuring our dependencies
* easier testing 
* greater modularity of a program


IoC can be achieved in few ways, so actually IoC is a concept which is implemented by mechanisms such as: Strategy design pattern, Factory pattern, Service Locator pattern and finally Dependency Injection (DI).

<br>

### Dependency Injection

One of most known implementation of IoC, which is based on idea to have a separate object, an assembler, which is responsible for connecting objects with each other, or rather "injecting" them into other objects. 

Dependency diagram with our songs example:
![img]({{site.url}}/assets/blog_images/2022-24-10-inversion-of-control-and-the-dependency-injection/di-architecture.jpg)


<br>

Our code should look like below. Now we are expecting a provider in a constructor, or rather an SongsProvider interface, not a concrete implementation as you noticed.
What are a pros of that? This give as a huge possibility to use different providers without breaking changes, thanks to the fact that it's decoupled.
For me personally more suitable would be Configurable Dependency rather than Dependency Injection, but I hope that you got the idea.

```java
class SongLister {

    private SongsProvider provider;

    public MovieLister(SongsProvider provider) {
        this.provider = provider;
    }
}
```

#### Types of Dependency Injection 

**Dependency Injection in Spring can be done through constructors, setters or fields.**

<br> 

1. Constructor-Based Dependency Injection
   The container will invoke a constructor with argument representing a dependency we want to set.
   An example of that approach is above, it's recommended for mandatory dependencies, but to be honest it used most of the time.

2. Setter-Based Dependency Injection
   The container will call setter method after initiating a bean by invoking a no-argument constructor or no-argument static factory method.
   Spring documentation recommends setter-based injection for optional ones.
    
3. Field-Based Dependency Injection
    If there is no constructor or setter method to inject the Item bean, the container will use reflection to inject our desire dependency. 
    It's not recommended to use this approach because of following reasons:
    * It's bound us highly with Spring framework
    * This is done by reflection, which is costlier than constructor-based or setter-based injection and is also a reason for first point.
    * It's easy to add more and more dependencies, especially when using this approach and violate SRP. Constructor/setter-based injections would make us think earlier that we have too many dependencies. 



##### Recommended and used articles:

Baeldung: 
* [Intro to Inversion of Control and Dependency Injection with Spring](https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring) 

Martin Fowler's articles:
* [Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/bliki/InversionOfControl.html) 
* [Inversion of Control](https://martinfowler.com/articles/injection.html) 