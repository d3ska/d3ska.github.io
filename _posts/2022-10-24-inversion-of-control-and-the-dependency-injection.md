---
title: "Inversion of Control and Dependency Injection"
categories:
  - Software Design
tags:
  - IoC
  - Dependency Injection
  - Design Principles
  - Software Architecture
---

Consider a program that manages songs: you can search them, read details, and discover interesting facts about each. There would probably be a class that searches for those songs by asking some external provider.

### The Problem: Hidden Dependencies

A first attempt might look like this:

```java
class SongLister {

    public final SpotifySongsProvider provider = new SpotifySongsProvider();

    public Collection<Song> songsByArtist(String artistName) {
        return provider.findSongsByArtist(artistName);
    }
}
```

This implementation has several problems. The field is `public`, so anyone can replace or misuse the provider directly. It uses a concrete class (`SpotifySongsProvider`) instead of an abstraction, so the lister is tightly coupled to Spotify. And the provider is instantiated inline, so there is no way to swap it for a different implementation or a test double without modifying the class.

I want `songsByArtist` to be completely independent of the provider. In other words, *what* we do should not be limited by *how* we do it. The source of songs should be an implementation detail behind an abstraction.

The first step is extracting an interface that defines the contract:

```java
public interface SongsProvider {
    List<Song> findSongsByArtist(String artistName);
}
```

Now let's try using the interface, but still creating the concrete implementation inside the constructor:

```java
class SongLister {

    private final SongsProvider provider;

    public SongLister() {
        this.provider = new SpotifyProvider();
    }
}
```

![img]({{site.url}}/assets/blog_images/2022-24-10-inversion-of-control-and-the-dependency-injection/concrete-implementation-constructor-initializing-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2022-24-10-inversion-of-control-and-the-dependency-injection/concrete-implementation-constructor-initializing-dark.png){: .dark }

This is better, but `SongLister` still depends on `SpotifyProvider`. If we want to use a different source of songs (say, a local file for development or a different streaming service in production), we have to change the class. We would like providers to be interchangeable plugins, but by directly instantiating one, we lose that flexibility.

### What Is Inversion of Control?

**Inversion of Control** (IoC) is a principle that transfers control over object creation and wiring from the objects themselves to an external entity: a framework, a container, or simply the calling code.

Without IoC, the `SongLister` decides which provider to create. With IoC, something outside the `SongLister` makes that decision and supplies the provider. The key advantages:

* Decoupling between *what* a class does and *how* its dependencies are provided
* Components become configurable and easily extendable
* Easier testing (dependencies can be replaced with test doubles)
* Greater modularity and separation of concerns

IoC is a broad concept. It can be achieved through several mechanisms: the Strategy pattern, the Factory pattern, the Service Locator pattern, and Dependency Injection (DI).

Worth noting: while the Service Locator pattern technically achieves IoC, it is widely considered an anti-pattern in modern applications. The key issue is that it hides class dependencies instead of making them explicit, which makes the code harder to reason about and test. With a Service Locator, you cannot tell from a class's constructor what it depends on. You only discover missing dependencies at runtime. Dependency Injection, by contrast, makes every dependency visible and explicit.

### Dependency Injection

**Dependency Injection** is the most widely used form of IoC. Instead of a class creating or locating its own dependencies, an external assembler "injects" them. The class declares what it needs (through a constructor parameter, a setter, or a field), and someone else provides it.

![img]({{site.url}}/assets/blog_images/2022-24-10-inversion-of-control-and-the-dependency-injection/di-architecture-light.png){: .light }
![img]({{site.url}}/assets/blog_images/2022-24-10-inversion-of-control-and-the-dependency-injection/di-architecture-dark.png){: .dark }

Applied to our example, the `SongLister` now expects a `SongsProvider` in its constructor instead of creating one:

```java
class SongLister {

    private final SongsProvider provider;

    public SongLister(SongsProvider provider) {
        this.provider = provider;
    }

    public Collection<Song> songsByArtist(String artistName) {
        return provider.findSongsByArtist(artistName);
    }
}
```

The caller decides which implementation to use:

```java
SongLister lister = new SongLister(new SpotifyProvider());
// or for local development:
SongLister lister = new SongLister(new LocalFileProvider());
```

This gives us the flexibility to swap providers without any changes to `SongLister`. For me, a more descriptive name for this concept would be "configurable dependency" rather than "dependency injection."

### Types of Dependency Injection

There are three common ways to inject dependencies.

**Constructor injection** passes dependencies through the constructor. This is the recommended approach for required dependencies because it makes them explicit, enforces that the object is fully initialized before use, and allows fields to be `final`.

```java
class SongLister {

    private final SongsProvider provider;

    public SongLister(SongsProvider provider) {
        this.provider = provider;
    }
}
```

**Setter injection** provides dependencies through setter methods after construction. This is suitable for optional dependencies that have reasonable defaults.

```java
class SongLister {

    private SongsProvider provider = new DefaultProvider();

    public void setProvider(SongsProvider provider) {
        this.provider = provider;
    }
}
```

**Field injection** sets dependencies directly on fields, typically using reflection. This approach is generally discouraged outside of tests for several reasons:

* It hides dependencies, making it impossible to tell what a class needs by looking at its constructor
* It prevents fields from being `final`, since the field must be writable after construction
* It relies on reflection, which bypasses Java's access control mechanisms
* It makes unit testing harder without a DI framework
* It makes it easy to keep adding dependencies without noticing that the class is doing too much, violating the Single Responsibility Principle

```java
class SongLister {

    @Inject
    private SongsProvider provider;
}
```

### Dependency Injection in Spring

Spring is the most widely used DI framework in the Java ecosystem. It manages object creation and lifecycle through its **application context** (the IoC container) and injects dependencies based on configuration or annotations.

Spring supports all three injection types. For constructor injection, if a class has a single constructor, Spring uses it automatically without requiring the `@Autowired` annotation (since Spring 4.3). This further reinforces constructor-based injection as the idiomatic default in Spring applications.

```java
@Service
class SongLister {

    private final SongsProvider provider;

    public SongLister(SongsProvider provider) {
        this.provider = provider;
    }
}
```

For setter injection, Spring calls the annotated setter after constructing the bean:

```java
@Service
class SongLister {

    private SongsProvider provider;

    @Autowired
    public void setProvider(SongsProvider provider) {
        this.provider = provider;
    }
}
```

Spring documentation recommends setter-based injection for optional dependencies.

For field injection, Spring uses reflection to set the field directly:

```java
@Service
class SongLister {

    @Autowired
    private SongsProvider provider;
}
```

Field injection is the most concise syntax, but it carries all the disadvantages described earlier. The only scenario where it is generally acceptable is in test classes, where brevity outweighs the design concerns.

> **Related post:** [SOLID: The First 5 Principles of Object Oriented Design](/posts/solid-the-first-5-principles-of-object-oriented-design/)

### References

* Martin Fowler, [Inversion of Control](https://martinfowler.com/bliki/InversionOfControl.html)
* Martin Fowler, [Inversion of Control Containers and the Dependency Injection pattern](https://martinfowler.com/articles/injection.html)
* Baeldung, [Intro to Inversion of Control and Dependency Injection with Spring](https://www.baeldung.com/inversion-control-and-dependency-injection-in-spring)
