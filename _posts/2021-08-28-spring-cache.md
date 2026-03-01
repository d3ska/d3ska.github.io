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

A product catalog service queries the database for the same 500 products on every page load. The query takes 200ms and runs thousands of times per hour. Spring Cache lets you store the result in memory and skip the database entirely on subsequent calls, turning that 200ms into microseconds.

Spring's caching abstraction sits on top of a pluggable **CacheManager** layer. You annotate methods, and Spring intercepts calls via proxies, checking the cache before hitting the actual implementation. The abstraction is thread-safe by design and supports both built-in managers (`ConcurrentMapCacheManager`, `SimpleCacheManager`) and external providers (`CaffeineCacheManager`, `EhCacheCacheManager`, `JCacheCacheManager`, `RedisCacheManager`).

### Setup

First, add the dependency. For Spring Boot:

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-cache</artifactId>
</dependency>
```

For plain Spring (without Boot):

```xml
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-context</artifactId>
    <version>5.1.8.RELEASE</version>
</dependency>
```

Then enable caching with `@EnableCaching` and register a `CacheManager`:

```java
@Configuration
@EnableCaching
public class CacheConfiguration {

    @Bean
    public CacheManager cacheManager() {
        return new ConcurrentMapCacheManager("products");
    }
}
```

With Spring Boot, the mere presence of `spring-boot-starter-cache` on the classpath combined with `@EnableCaching` will auto-register a `ConcurrentMapCacheManager`, so the bean declaration above is optional. You can customize the auto-configured manager with a `CacheManagerCustomizer` bean:

```java
@Component
public class ProductCacheCustomizer implements CacheManagerCustomizer<ConcurrentMapCacheManager> {

    @Override
    public void customize(ConcurrentMapCacheManager cacheManager) {
        cacheManager.setCacheNames(asList("products", "productCategories"));
    }
}
```

### @Cacheable

The simplest way to cache a method's return value is to annotate it with `@Cacheable`. Spring checks the cache before invoking the method. If a cached entry exists for the given key, the method body is skipped entirely.

```java
@Service
public class ProductService {

    private final ProductRepository productRepository;

    public ProductService(ProductRepository productRepository) {
        this.productRepository = productRepository;
    }

    @Cacheable("products")
    public Product getProduct(Long productId) {
        return productRepository.findById(productId)
            .orElseThrow(() -> new ProductNotFoundException(productId));
    }
}
```

The first call to `getProduct(42L)` hits the database and stores the result under key `42` in the `products` cache. Every subsequent call with the same argument returns the cached `Product` without touching the database.

Now, the obvious question: why not make every method `@Cacheable`? The problem is size. Caches can grow large, fast. You end up holding stale or rarely accessed data in memory, and the overhead of cache management can outweigh the benefits. Cache selectively, only where the read frequency justifies it.

#### Conditional Caching

Spring provides two attributes on `@Cacheable` for fine-grained control over when caching should occur:

- `condition` -- evaluated **before** the method executes. If the expression evaluates to `false`, the cache is neither checked nor populated. For example: `@Cacheable(value = "products", condition = "#productId > 10")`.
- `unless` -- evaluated **after** the method executes. If it evaluates to `true`, the result is **not** stored in the cache, but the cache is still checked before invocation. For example: `@Cacheable(value = "products", unless = "#result.price == 0")`.

Combining both attributes lets you cache only the results that truly matter, keeping the cache lean and effective.

### @CachePut

When a method modifies data, you want the cache to reflect the change immediately. `@CachePut` always executes the method body and then updates the cache entry with the returned value:

```java
@CachePut(value = "products", key = "#result.id")
public Product updateProduct(Product updatedProduct) {
    return productRepository.save(updatedProduct);
}
```

Notice the `key = "#result.id"` expression. The `#result` variable refers to the object returned by the method. We extract the `id` from it to make sure the cache entry matches the one created by `@Cacheable`.

### @CacheEvict

When data is deleted, the corresponding cache entry should go with it. `@CacheEvict` removes the entry after the method completes:

```java
@CacheEvict("products")
public void deleteProduct(Long productId) {
    productRepository.deleteById(productId);
}
```

Since the method parameter name matches the default key strategy, we don't need an explicit `key` attribute here.

The `@CacheEvict` annotation also supports the `allEntries` attribute. When set to `true`, it evicts all entries from the specified cache rather than just the one matching the key. This is useful when an operation invalidates an entire dataset:

```java
@CacheEvict(value = "products", allEntries = true)
public void reloadProductCatalog() {
    // After this method completes, the entire "products" cache is cleared
}
```

### @Caching and @CacheConfig

When a single method needs multiple cache operations at once, Spring provides the `@Caching` annotation. It groups several `@Cacheable`, `@CachePut`, and `@CacheEvict` annotations together:

```java
@Caching(
    evict = {
        @CacheEvict(value = "products", key = "#product.id"),
        @CacheEvict(value = "productCategories", key = "#product.categoryId")
    }
)
public void removeProduct(Product product) {
    productRepository.delete(product);
}
```

If every method in a class operates on the same cache, you can extract shared configuration to the class level with `@CacheConfig`:

```java
@Service
@CacheConfig(cacheNames = "products")
public class ProductService {

    @Cacheable
    public Product getProduct(Long productId) { ... }

    @CacheEvict
    public void deleteProduct(Long productId) { ... }
}
```

### Cache Key Design

By default, Spring uses the method parameters as the cache key. This works well for single-argument methods, but you need to be deliberate about keys when things get more complex.

**SpEL expressions** give you full control. You can reference method parameters by name or by index:

```java
@Cacheable(value = "products", key = "#category + '-' + #page")
public List<Product> getProductsByCategory(String category, int page) {
    return productRepository.findByCategory(category, PageRequest.of(page, 50));
}
```

For composite keys, Spring also provides a `SimpleKeyGenerator` that combines multiple parameters automatically. If you need custom logic (for example, hashing certain fields), you can implement a `KeyGenerator` bean and reference it with `keyGenerator = "myKeyGenerator"`.

One thing to watch out for: **key collisions**. If two methods share the same cache name but use different parameter types that happen to produce the same key, they will overwrite each other's entries. Either use distinct cache names per method or design your keys to be unambiguous.

### TTL, Eviction Policies, and Cache Sizes

The default `ConcurrentMapCacheManager` does not support Time-To-Live (TTL), maximum size limits, or eviction policies. For production workloads, I strongly recommend using an external cache provider such as Caffeine or Redis.

With Caffeine, TTL and maximum cache size can be configured directly:

```java
@Bean
public CacheManager cacheManager() {
    CaffeineCacheManager manager = new CaffeineCacheManager("products");
    manager.setCaffeine(Caffeine.newBuilder()
        .expireAfterWrite(10, TimeUnit.MINUTES)
        .maximumSize(500));
    return manager;
}
```

Redis-backed caches offer similar configuration through `RedisCacheConfiguration`, including TTL per cache region. Choosing the right eviction policy (LRU, LFU, size-based) depends on the access patterns of your application.

### Common Pitfalls

#### Self-Invocation Bypass

This one catches people off guard. `@Cacheable` relies on Spring's proxy mechanism. When a method calls another method within the same class, the call bypasses the proxy, and the cache annotation is silently ignored:

```java
@Service
public class ProductService {

    public Product getProductWithFallback(Long productId) {
        // This call does NOT go through the proxy, cache is skipped!
        return getProduct(productId);
    }

    @Cacheable("products")
    public Product getProduct(Long productId) {
        return productRepository.findById(productId).orElseThrow();
    }
}
```

The fix is to either inject the service into itself (which feels wrong but works), extract the cached method into a separate bean, or use `AopContext.currentProxy()`.

#### Cache Stampede (Thundering Herd)

When a popular cache entry expires, hundreds of concurrent requests can hit the database simultaneously to recompute the same value. This is called a **cache stampede**. Caffeine handles this natively with its `refreshAfterWrite` option, which refreshes entries asynchronously before they expire. For other providers, you may need to implement locking or use `@Cacheable(sync = true)`, which ensures only one thread computes the value while others wait.

#### Stale Data

Caching data that changes frequently leads to serving outdated information. If your product prices update every few seconds, a 10-minute TTL means users see stale prices for up to 10 minutes. Match your TTL to your data's actual change frequency, and use `@CachePut` or `@CacheEvict` on write paths to keep the cache consistent.

#### Memory Pressure

An unbounded cache can grow until it causes an `OutOfMemoryError`. The default `ConcurrentMapCacheManager` never evicts entries on its own. Always configure a maximum size in production, either through Caffeine's `maximumSize()` or Redis's memory policies.

### When Not to Cache

Caching is not always the right answer. Avoid it when:

- **Data changes on every request.** Real-time stock prices, live scores, or sensor readings are stale the moment you cache them.
- **Write-heavy data.** If every write triggers a cache eviction or update, the invalidation cost can exceed the savings from cached reads.
- **Security-sensitive data.** Cached authentication tokens, personal data, or financial records persist in memory and increase the attack surface.
- **Low read frequency.** If a particular piece of data is rarely read, caching it wastes memory without meaningful performance gain.

### Closing

Spring Cache provides a clean, annotation-driven way to add caching to your application with minimal code changes. Start with `@Cacheable` on your most frequently read methods, use `@CachePut` and `@CacheEvict` to keep the cache consistent with writes, and move to Caffeine or Redis when you need TTL, size limits, and eviction policies. The annotations are simple, but the decisions around what to cache, for how long, and how to invalidate are where the real engineering happens.

> **Related posts**: [Caching](/posts/caching/)
