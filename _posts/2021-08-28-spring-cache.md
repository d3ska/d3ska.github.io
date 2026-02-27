---
title: "Spring Cache"
categories:
 - Java
tags:
 - Caching
 - Spring
 - Spring Cache
 - Java
---

### What is Spring Cache? How to configure it in your project and how to use it? How does Spring Cache work? How to add an external provider that allows flexible management of our cache?

Spring Cache is a mechanism that allows you to temporarily store frequently accessed data in memory, avoiding repeated expensive computations or database calls.
The purpose of this is to increase the speed of access to this information when it is potentially needed in the near future.
This mechanism allows you to limit the number of queries to the database, or the number of external calls to the services we use.
The cache is used in virtually all systems, from hardware-level CPU caches all the way up to application-level data stores. The creators of Spring made sure that the caching abstraction they provide is safe to use in multithreaded applications.

Spring-supported CacheManagers can be divided into:
* using Spring's internal mechanisms, e.g. ConcurrentMapCacheManager, SimpleCacheManager.
* enabling the configuration of external providers, e.g. CaffeineCacheManager, EhCacheCacheManager, JCacheCacheManager, RedisCacheManager.

**How to configure and use Spring Cache?**

First we need to add a dependency:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```


If our application uses Spring only, we should use:

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.1.8.RELEASE</version>
</dependency>
```

To enable caching we need to add an annotation, much like enabling any other configuration level feature in the framework.
We can do it by adding the @EnableCaching annotation to any of the configuration classes.

```java
@Configuration
@EnableCaching
public class CacheConfiguration {

    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("books");
    }

}
```

After we enable caching, for the minimal setup, we must register a cacheManager.

When using Spring Boot, the mere presence of the `spring-boot-starter-cache` starter dependency on the classpath, combined with the @EnableCaching annotation, will register the same ConcurrentMapCacheManager. So there is no need for a separate bean declaration.
In addition, we can customize an automatically configured CacheManager with one or more CacheManagerCustomizer<T> beans:

```java
@Component
public class SimpleCacheCustomizer implements CacheManagerCustomizer<ConcurrentMapCacheManager> {

    @Override
    public void customize(ConcurrentMapCacheManager cacheManager) {
        cacheManager.setCacheNames(asList("users", "addresses"));
    }

}
```

The simplest way to enable caching for a method is to annotate it with `@Cacheable`. This annotation is used for methods that retrieve data but do not modify it. It also has parameters that we can and even have to set.

```java
@Cacheable("addresses")
public String getAddress(Customer customer) {
    return addressService.getCustomerAddresses(customer.getId());
}
```

The getCustomerAddresses() call will first check the cache addresses before actually invoking the method and then caching the result.

Now, what would be the problem with making all methods @Cacheable?

The problem is size. We don't want to populate the cache with values that we don't need often.
Caches can grow quite large, quite fast, and we could be holding on to a lot of stale or unused data.

**Conditional Caching**

Spring provides two attributes on `@Cacheable` for fine-grained control over when caching should occur:

* `condition` -- evaluated **before** the method executes. If the expression evaluates to `false`, the cache is neither checked nor populated. For example: `@Cacheable(value = "addresses", condition = "#customerId > 10")`.
* `unless` -- evaluated **after** the method executes. If it evaluates to `true`, the result is **not** stored in the cache, but the cache is still checked before invocation. For example: `@Cacheable(value = "addresses", unless = "#result.size() == 0")`.

Combining both attributes lets us cache only the results that truly matter, keeping the cache lean and effective.

**@CachePut**

This annotation is added on methods that modify the data. It allows for the fact that when the method with this annotation is called, it updates the given cache entry, so that we always have the most recent changes that we can use. It is important here to add parameters as in the @Cacheable annotation. So we need to set cacheName.
In this example, we want our key to be id, so we need to extract it from result (this is the object that the method returns to us, i.e. the returned object = result):

```java
@CachePut(value = "addresses", key = "#result.id")
public Address editAddress(Address newAddress, Long idOfOldAddress){
    return addressService.editCustomerAddress(newAddress, idOfOldAddress);
}
```

**@CacheEvict**

That annotation allows us to clear the cache. We add it on methods that delete data.
Thanks to this, in the case of calling the method that removes some data, our cache will also be cleared from this entry.
This frees space in the cache memory and helps maintain consistency between the main data source and the cache.
As usual, it is obligatory to add cacheName and key, but in this case we can omit the key, because the key will be id, which is in the method parameter:

```java
@CacheEvict("addresses")
public void deleteAddress(Long addressId) {
    addressService.deleteCustomerAddress(addressId);
}
```

The `@CacheEvict` annotation also supports the `allEntries` attribute. When set to `true`, it evicts all entries from the specified cache rather than just the one matching the key. This is useful when an operation invalidates an entire dataset:

```java
@CacheEvict(value = "addresses", allEntries = true)
public void refreshAllAddresses() {
    // After this method completes, the entire "addresses" cache is cleared
}
```

**@Caching and @CacheConfig**

When a single method needs multiple cache operations at once, Spring provides the `@Caching` annotation. It groups several `@Cacheable`, `@CachePut`, and `@CacheEvict` annotations together:

```java
@Caching(
    evict = {
        @CacheEvict(value = "addresses", key = "#customer.id"),
        @CacheEvict(value = "customerDirectory", key = "#customer.id")
    }
)
public void removeCustomer(Customer customer) {
    // Both cache entries are evicted in a single operation
}
```

If every method in a class operates on the same cache, we can extract shared configuration to the class level with `@CacheConfig`:

```java
@Service
@CacheConfig(cacheNames = "addresses")
public class AddressService {

    @Cacheable
    public List<Address> getAddresses(Long customerId) { ... }

    @CacheEvict
    public void removeAddress(Long customerId) { ... }
}
```

**TTL, Eviction Policies, and Cache Sizes**

The default `ConcurrentMapCacheManager` provided by Spring does not support Time-To-Live (TTL), maximum size limits, or eviction policies. For production workloads, I strongly recommend using an external cache provider such as Caffeine or Redis.

With Caffeine, for example, TTL and maximum cache size can be configured directly:

```java
@Bean
public CacheManager cacheManager() {
    CaffeineCacheManager manager = new CaffeineCacheManager("addresses");
    manager.setCaffeine(Caffeine.newBuilder()
        .expireAfterWrite(10, TimeUnit.MINUTES)
        .maximumSize(500));
    return manager;
}
```

Redis-backed caches offer similar configuration through `RedisCacheConfiguration`, including TTL per cache region. Choosing the right eviction policy (LRU, LFU, size-based) depends on the access patterns of your application.

As you can see, using the cache is important in the application when we want it to be fast and efficient. The cache is a very important part of the system and we shouldn't skip it.
