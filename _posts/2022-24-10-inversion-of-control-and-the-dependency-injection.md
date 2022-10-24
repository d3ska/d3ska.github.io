---
title: "Inversion of Control and Dependency Injection"
categories:
  - Blog
tags:
  - IoC and DI
  - Design 
  - Application Architecture
---

Inversion of Control and Dependency Injection


Let's take a look on the example where we have some service which provides list of songs by its author.

```java
class SongLister {
    
    public final SpotifySongsProvider provider;
    
    public Collection<Song> songsByArtist(String artistName){
       return provider.findSongsByArtist(artistName);
    }
}
```
The above implementation is pretty naive. Why so?
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
