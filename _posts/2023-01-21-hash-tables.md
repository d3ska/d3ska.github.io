---
title: "Hash Tables"
categories:
  - Data Structures
tags:
  - Data Structures
  - Complexity Analysis
  - Hash Tables
  - HashMap
---

A data structure that stores data in key-value pairs and provides fast insertion, deletion and searching.

Under the hood it uses a dynamic array of linked lists to efficiently store key/value pairs. This approach to handling collisions is called **separate chaining** -- each array slot points to a linked list of entries that hashed to the same index. An alternative strategy is **open addressing**, where all entries are stored directly in the array itself. When a collision happens, the algorithm probes for the next available slot using techniques like linear probing (checking the next slot sequentially) or quadratic probing (checking slots at quadratically increasing offsets).

When searching for some value or inserting a key/value pair, a hash function first maps our key to an integer value which will be our index in the underlying dynamic array.

Then the value associated with the key is added to the linked list stored at the index in the dynamic array, and the reference to the key is stored with the value as well, so in case if a collision occurs and more than one element will be stored in the same linked list, it would know which value is searched.

However, hash tables rely on very powerful and highly optimized hash functions to minimize the number of collisions that occur when two or more keys hash to the same index.

Visualization of how HashTable may look under the hood:

[<br>
0: (key1, value1) => null<br>
1: (key2, value2) => (key3, value3) => (key4, value4)<br>
2: (key5, value5) => null<br>
3: (key6, value6) => (key7, value7)<br>
4: null <br>
5: (key8, value8) => null<br>
]

In the hash table above, the keys key2, key3, key4 collided by all being mapped to the linked list at index 1, and key6 and key7 collided by both being mapped to index 3.

**Load factor and resizing:** The load factor is the ratio of stored entries to the total number of slots in the array. As the load factor grows, collisions become more frequent and performance degrades. To maintain efficient operations, hash tables monitor this ratio and resize when it crosses a threshold (commonly 0.75). Resizing typically means allocating a new array of double the size, then rehashing every existing entry into the new array. This single resize operation is O(n), but because it happens infrequently, the amortized cost of insertion remains O(1).

**Keys must be hashable:** For a hash table to work correctly, keys must produce consistent hash values. In Java, this means keys must properly implement `hashCode()` and `equals()`. In Python, only immutable types (strings, numbers, tuples) can serve as dictionary keys. Using a mutable object as a key can lead to lost entries, because if the object changes after insertion its hash value may change too, making it impossible to locate.

**Real-world implementations:** Most languages provide a built-in hash table. Java offers `HashMap` (and the thread-safe `ConcurrentHashMap`), Python has `dict`, C++ provides `std::unordered_map`, and JavaScript uses plain objects and `Map`.

Time complexity, as mentioned at the beginning they are fast and provide basic operations in the following complexity:

- Inserting a pair: O(1) average, O(n) worst case
- Removing a pair: O(1) average, O(n) worst case
- Searching a pair: O(1) average, O(n) worst case

The worst case, linear-time complexity occurs when a lot of collisions happen leading to long linked lists, but it happens extremely rarely as hash functions nowadays are really well optimized and constant-time complexity is all but guaranteed.

### Practical HashMap Usage in Java

Two patterns come up constantly when working with hash tables: counting things and caching results. Let's walk through both.

**Word frequency counting** is probably the most classic HashMap exercise. Given an array of strings, we want to know how many times each word appears. The `getOrDefault` method keeps the code clean — no need for manual null checks:

```java
import java.util.HashMap;
import java.util.Map;

public class WordFrequency {
    public static Map<String, Integer> countWords(String[] words) {
        Map<String, Integer> frequency = new HashMap<>();
        for (String word : words) {
            frequency.put(word, frequency.getOrDefault(word, 0) + 1);
        }
        return frequency;
    }

    public static void main(String[] args) {
        String[] words = {"apple", "banana", "apple", "cherry", "banana", "apple"};
        System.out.println(countWords(words));
        // {banana=2, cherry=1, apple=3}
    }
}
```

**Memoization** is the other big one. A naive recursive Fibonacci implementation has O(2^n) time complexity because it recomputes the same subproblems over and over. By dropping a HashMap in as a cache, we store each result the first time we compute it and look it up on every subsequent call. This brings the complexity down to O(n) — each Fibonacci number is calculated exactly once:

```java
import java.util.HashMap;
import java.util.Map;

public class FibonacciMemo {
    private static Map<Integer, Long> cache = new HashMap<>();

    public static long fib(int n) {
        if (n <= 1) return n;
        if (cache.containsKey(n)) return cache.get(n);
        long result = fib(n - 1) + fib(n - 2);
        cache.put(n, result);
        return result;
    }

    public static void main(String[] args) {
        System.out.println(fib(50)); // 12586269025, computed instantly
    }
}
```

Without the cache, `fib(50)` would take an astronomical number of recursive calls. With it, we make roughly 50. That is the power of trading a little memory for a massive time savings. HashMap is the default choice for both of these patterns because its O(1) average-time `get` and `put` add virtually no overhead, letting the data structure disappear behind the logic.

### What Makes a Good Hash Function

Not all hash functions are created equal. A good one needs two key properties. First, it must be **deterministic** — the same input must always produce the same output. If `"hello"` hashes to 42 on one call and 99 on the next, the table can never find the value again. Second, it should provide **uniform distribution**, spreading keys as evenly as possible across the available buckets so that no single bucket accumulates a disproportionate share of entries.

A bad hash function is easy to write. Imagine hashing a string by looking only at its first character. The words "apple", "avocado", and "algorithm" would all land in the same bucket, creating a long chain while other buckets sit empty. That is exactly the kind of clustering we want to avoid.

Java's `String.hashCode()` is a good example of a well-designed hash function. It uses a **polynomial rolling hash** with multiplier 31:

`s[0]*31^(n-1) + s[1]*31^(n-2) + ... + s[n-1]`

Why 31? It is an odd prime, which helps spread bits around, and the JVM can optimize the multiplication `31 * x` into `(x << 5) - x` — a bitwise shift and a subtract — which is faster than a general-purpose multiply instruction.

Quality matters because a poor hash function can turn every operation from its expected O(1) average into O(n) worst case. When entries cluster into a handful of buckets, searching through those long chains is no different from scanning a linked list. The hash function is the single biggest lever you have over a hash table's real-world performance.

### Ordered Alternatives: TreeMap and LinkedHashMap

`HashMap` is fast, but it makes no guarantees about the order of its entries. When ordering matters, Java gives you two alternatives.

**LinkedHashMap** maintains **insertion order**. Internally it layers a doubly-linked list on top of the regular hash table, so iteration walks entries in the order they were added. Get and put stay O(1). It also supports an **access-order mode** — pass `true` as the third constructor argument — which moves entries to the end of the list on every access, making it a natural building block for **LRU caches**.

```java
Map<String, Integer> map = new LinkedHashMap<>();
map.put("banana", 2);
map.put("apple", 5);
map.put("cherry", 3);

for (Map.Entry<String, Integer> entry : map.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}
// banana = 2
// apple = 5
// cherry = 3  (insertion order preserved)
```

**TreeMap** keeps keys in **sorted order** — either their natural ordering or a custom `Comparator` you provide. Under the hood it is backed by a **Red-Black tree**, so `get` and `put` run in O(log n). Where TreeMap really shines is **range queries**: `subMap`, `headMap`, and `tailMap` let you slice out a range of keys efficiently.

```java
TreeMap<String, Integer> tree = new TreeMap<>();
tree.put("cherry", 3);
tree.put("apple", 5);
tree.put("banana", 2);

for (Map.Entry<String, Integer> entry : tree.entrySet()) {
    System.out.println(entry.getKey() + " = " + entry.getValue());
}
// apple = 5
// banana = 2
// cherry = 3  (sorted order)
```

When to choose: reach for `HashMap` when order does not matter — it is the fastest option. Use `LinkedHashMap` when you need to preserve insertion order or build an LRU cache. Pick `TreeMap` when you need keys in sorted order or plan to run range queries.

For related topics, see [Trees](https://matthewonsoftware.com/blog/trees/) and [Arrays](https://matthewonsoftware.com/blog/arrays/).
