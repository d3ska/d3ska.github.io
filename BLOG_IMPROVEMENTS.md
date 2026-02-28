# Blog Improvement Plan

Comprehensive action items for all 65 posts, organized by priority.
Generated from a full editorial review on 2026-02-28.

---

## ~~Priority 1: Rewrite (Score 5/10)~~ COMPLETED

All three posts fully rewritten on 2026-02-28. See `scratch/*-editorial-review.md` for details.

### Singleton Pattern ~~(5/10)~~ -> 8.5/10
**File:** `_posts/2022-08-26-singleton-pattern.md`
- [x] Rewrite opening: remove "To be more particular" filler, start with a concrete shared-resource problem
- [x] Explain WHY `volatile` is needed in double-checked locking (happens-before, partial construction visibility)
- [x] Explain WHY Bill Pugh/holder idiom is thread-safe (JVM class initialization guarantee)
- [x] Add concrete DI alternative with Spring `@Scope("singleton")` code example
- [x] Show serialization attack in code, not just prose
- [x] Add comparison table: implementation vs thread-safety vs lazy-init vs serialization-safe
- [x] Restructure: move anti-pattern discussion earlier, make it the thesis of the post
- [x] Add "when singleton IS appropriate" guidance (stateless services, read-only config)
- [x] Add transition between implementations section and anti-pattern section

### Custom Assertions with AssertJ ~~(5/10)~~ -> 8.5/10
**File:** `_posts/2021-08-14-custom-assertions-with-assertj.md`
- [x] Replace generic `Person` example with a real domain object (Order, LoanApplication)
- [x] Add before/after comparison: messy test with standard assertions vs same test with custom assertions
- [x] Deliver on the DDD/ubiquitous language promise from the intro, or remove it
- [x] Add "when NOT to use" section (maintenance overhead, simple cases)
- [x] Replace `<br>` tags with proper markdown spacing
- [x] Add cross-references to Best Practices and Schools of Unit Testing posts
- [x] Rewrite opening with a concrete problem scenario
- [x] Show how custom assertions improve failure messages (show actual output)

### Network Protocols ~~(5/10)~~ -> 8/10
**File:** `_posts/2023-04-03-network-protocols.md`
- [x] Add socket programming code example (TCP connection, send/receive)
- [x] Add practical scenario: when to use UDP vs TCP (gaming, video, financial trading)
- [x] Fix tone inconsistency ("So basically it's a more powerful wrapper")
- [x] Replace textbook-definition opening with a problem-first approach
- [x] Deepen HTTP versions section with concrete differences (multiplexing, server push, QUIC)
- [x] Show actual packet structure or at minimum explain what travels over the wire
- [x] Remove or rewrite the "Prerequisites" section (feels like an afterthought)

---

## Priority 2: Major Revision (Score 6-6.5/10)

These posts have significant gaps but salvageable structure.

### Strategy Design Pattern (6/10)
**File:** `_posts/2021-08-11-strategy-design-pattern.md`
- [ ] **FIX BUG:** `PayByPayPal.verify()` is never called from `pay()`, leaving `signedIn` always false
- [ ] **FIX BUG:** Line 97 has backwards validation: `email.equals(DATA_BASE.get(password))` (email/password swapped)
- [ ] Fill in `Order.processOrder()` empty body
- [ ] Add end-to-end example (main method or test) showing strategy selection and execution
- [ ] Add discussion of when strategy is overkill (class proliferation trade-off)
- [ ] Replace or fix the missing diagram (`<ADD>diagram...` placeholder)
- [ ] Add depth on runtime strategy selection and DI integration

### Facade Design Pattern (6/10)
**File:** `_posts/2023-02-07-facade-design-pattern.md`
- [ ] Replace the Refactoring Guru example with an original one (or substantially enhance the analysis)
- [ ] Develop the god object warning: show how to recognize it and how to split a facade
- [ ] Define the missing `CodecFactory` class
- [ ] Add connection to Dependency Inversion Principle
- [ ] Keep the comparison table (Facade vs Adapter vs Mediator), it's the best part

### Spring Bean Scopes (6/10)
**File:** `_posts/2021-08-15-spring-bean-scopes.md`
- [ ] Add problem-first intro: "If you inject the same bean into two controllers, do they share state?"
- [ ] Explain WHY `proxyMode` is necessary (not just "is necessary")
- [ ] Replace `@Resource` with `@Autowired` in SecurityInterceptor example
- [ ] Add decision table at the end: scope vs use case vs thread-safety vs typical scenario
- [ ] Move custom ThreadScope section earlier (it's the most valuable part)
- [ ] Unify voice: currently switches between tutorial and reference manual style

### Availability (6/10)
**File:** `_posts/2023-04-07-availability.md`
- [ ] Replace weak opening ("can be thought of in a couple of ways")
- [ ] Analyze a real outage in depth (AWS S3 2017, Facebook 2021, or GCP 2019): what failed, duration, impact, root cause
- [ ] Add cost analysis: what does 99.9% vs 99.99% cost in infrastructure?
- [ ] Show real SLA targets from AWS/GCP/Azure
- [ ] Add actual architecture example: multi-region failover diagram with tools (Kubernetes, Auto Scaling)
- [ ] Mention actual monitoring tools (PagerDuty, Datadog, CloudWatch)

### Replication and Sharding (6/10)
**File:** `_posts/2023-05-02-replication-and-sharding.md`
- [ ] Add comparison table for sharding strategies (range vs hash vs directory vs geographic)
- [ ] Discuss resharding: how do you add shard 5 to a 4-shard system?
- [ ] Add shard key selection examples and anti-patterns (user_id vs user_created_date)
- [ ] Show code for shard key calculation
- [ ] Address hot spot problem with concrete solution
- [ ] Add real-world example: "Twitter shards by user_id because..."
- [ ] Link CAP theorem post properly (currently says "refer to an article" without linking)

### Big O Notation (6/10)
**File:** `_posts/2022-12-08-big-o-notation.md`
- [ ] Rewrite opening: current definition is incomprehensible to beginners
- [ ] Add conceptual framework BEFORE listing complexities
- [ ] Fix informal pseudocode: use actual Java or clearly labeled pseudocode
- [ ] Remove exclamation marks and double negatives ("No! We can not!")
- [ ] Either expand Big Theta/Omega section with practical guidance or remove it
- [ ] Integrate amortized complexity properly (currently feels tacked on)
- [ ] Add a concrete motivating example showing why Big O matters

### Data Structures Compendium (6/10)
**File:** `_posts/2023-03-18-data-structures-compendium.md`
- [ ] Add suggested learning path/order (arrays -> linked lists -> hash tables -> trees -> graphs)
- [ ] Make descriptions consistent in style and specificity
- [ ] Move heaps from "additional" to main section (they're fundamental)
- [ ] Add structure relationship diagram
- [ ] Add missing structures: sets, segment trees
- [ ] Consider whether this post adds value beyond the auto-generated category page; if not, either enrich it or remove it

### Association, Composition, Aggregation (6/10)
**File:** `_posts/2021-12-18-association-composition-aggregation.md`
- [ ] **FIX:** School/Room composition example is technically incorrect (a room can exist without a school). Replace with Engine/Piston or Human/Heart
- [ ] Reconsider Mother/Child as association (it's arguably one of the strongest relationships)
- [ ] Add practical guidance: how to choose which relationship during design
- [ ] Add more depth after the "Note on Code Similarities" section (the best part, but the post stops too early)
- [ ] Show code that actually differs between the three types (lifecycle management, cascading delete)

### Cohesion (6/10)
**File:** `_posts/2024-04-22-cohesion.md`
- [ ] Explain each of the 7 cohesion types with code examples (currently just listed)
- [ ] Rank cohesion types from best to worst with clear labels
- [ ] Add step-by-step LCOM calculation example (currently too abstract)
- [ ] Move LCOM score interpretation directly after LCOM explanation
- [ ] Add actionable advice: how to improve cohesion during design
- [ ] Discuss relationship between cohesion and coupling (they're complementary)
- [ ] Fix inconsistent `<br>` tag formatting
- [ ] Cross-reference the Coupling post

### Key Types of Testing (6/10)
**File:** `_posts/2023-05-12-key-types-of-testing-in-software-development.md`
- [ ] Add testing pyramid or testing trophy diagram
- [ ] Add recommended test ratios (e.g., 70% unit, 20% integration, 10% E2E)
- [ ] Add anti-patterns section (inverting the pyramid, testing everything with E2E)
- [ ] Expand contract testing section (important for microservices)
- [ ] Add decision guidance: "when to write which type of test"
- [ ] Replace weak `shouldDisplayErrorMessage()` E2E example with something realistic
- [ ] Add synthesis: how these types work as a cohesive strategy

### Best Practices for Tests (6/10)
**File:** `_posts/2024-06-12-best-practices-for-writing-effective-and-reliable-tests.md`
- [ ] **FIX:** Replace the refactored test example (lines 62-84) that still violates its own Given-When-Then principle. Pick a better example or split the HTTP test into two separate tests
- [ ] Either expand FIRST principle (currently 5 lines) or remove it
- [ ] Demonstrate the patterns mentioned in bullet list (Assertion Class, Fixture Setup, Custom Assertions)
- [ ] Cross-reference the AssertJ custom assertions post
- [ ] Create stronger thematic cohesion (currently jumps between unrelated topics)
- [ ] Replace `<br>` tags with markdown spacing
- [ ] Strengthen opening: connect the "half hour builds" problem to the practices that follow

### Spring Cache (6.5/10)
**File:** `_posts/2021-08-28-spring-cache.md`
- [ ] Replace question-barrage opening with a problem-first approach
- [ ] Change headings from questions to statements
- [ ] Add "Common Pitfalls" section: cache stampede, thundering herd, key collisions
- [ ] Add "When NOT to cache" section (rapidly changing data, low read frequency)
- [ ] Add cache key design strategies
- [ ] Replace weak conclusion ("cache is important... when we want it to be fast")
- [ ] Use a single running example throughout instead of disconnected fragments

### Graphs (6.5/10)
**File:** `_posts/2023-03-02-graphs.md`
- [ ] **CRITICAL:** Add Java code for adjacency list (`Map<Integer, List<Integer>>`) and adjacency matrix (`int[][]`)
- [ ] Add BFS and/or DFS code implementation (at minimum one)
- [ ] Replace bullet-point glossary with prose and examples
- [ ] Show weighted graph representation (edge weights in data structure)
- [ ] Add cycle detection with visited set
- [ ] Replace abstract opening with concrete example (social network, road map)
- [ ] Show what "degree" means in the comparison table

---

## Priority 3: Enhancement (Score 7-7.5/10)

These posts are solid but have specific gaps worth filling.

### Trees (7/10)
**File:** `_posts/2023-03-14-trees.md`
- [ ] Add TreeNode implementation code
- [ ] Add BST insert/delete/search implementations
- [ ] Show at least recursive inorder traversal (don't only link to other post)
- [ ] Explain WHY balanced trees are O(log n) (height = log n)
- [ ] Add heap section (heaps are tree-based)
- [ ] Fix opening capitalization error
- [ ] Consider showing one rotation example for self-balancing trees

### Builder Design Pattern (7/10)
**File:** `_posts/2021-09-18-builder-design-pattern.md`
- [ ] Fill in `validateCarObject()` with actual validation logic
- [ ] Show what happens when validation fails (exception or Optional)
- [ ] Discuss builder interaction with class hierarchies (common pain point)
- [ ] Consider replacing car example with a more original domain
- [ ] Fix awkward phrasing on line 16-18

### Logarithm (7/10)
**File:** `_posts/2022-12-18-logarithm.md`
- [ ] Replace weak opening with forced joke
- [ ] Establish notation convention in first paragraph
- [ ] Replace `&nbsp;` spacing hacks with proper code blocks
- [ ] Add base conversion formula to explain why base doesn't matter in Big O
- [ ] Better transition to sorting algorithm table
- [ ] Either integrate real-world examples (decibels, Richter) into narrative or cut them

### SOLID Principles (7/10)
**File:** `_posts/2021-08-10-solid-the-first-5-principles-of-object-oriented-design.md`
- [ ] Add discussion of when SOLID can be over-applied/over-engineered
- [ ] Deepen LSP: explain WHY FlyingBird solution is better from substitutability perspective
- [ ] Consider replacing canonical examples (Bird/Ostrich, Book/BookPrinter) with original ones
- [ ] Connect code snippets: show how pieces work together
- [ ] Fix missing article on line 22

### Aggregate Pattern (7/10)
**File:** `_posts/2022-08-19-aggregate-pattern.md`
- [ ] **Complete the post** (currently ends abruptly mid-section around line 104)
- [ ] Add working code example of Order/OrderLineItem aggregate
- [ ] Explain eventual consistency through domain events (currently just mentioned)
- [ ] Add "when NOT to use" section (when a simple entity is sufficient)
- [ ] Discuss common mistake: making aggregates too large

### Schools of Unit Testing (7/10)
**File:** `_posts/2023-06-27-schools-of-unit-tests.md`
- [ ] Replace textbook-like opening with engaging hook
- [ ] Add concrete code example demonstrating Pilimon's heuristic
- [ ] Add decision tree or flowchart for choosing which school
- [ ] Take a clearer stance in conclusion (Classic/Sociable is generally preferred in modern testing)
- [ ] Add personal experience and real-world anecdotes
- [ ] Cross-reference @Mock vs @MockBean post

### Leader Election (7/10)
**File:** `_posts/2023-05-08-leader-election.md`
- [ ] Add ZooKeeper or etcd code example for leader election
- [ ] Explain Paxos and Raft at high level (currently just name-dropped)
- [ ] Add failure detection and lease/timeout mechanisms
- [ ] Replace "sophisticated" filler with specific description of consensus algorithms

### Depth First Search (7/10)
**File:** `_posts/2022-09-17-depth-first-search-in-java.md`
- [ ] Add graph DFS code with visited set for cycle handling
- [ ] Clarify title/opening: currently promises graph DFS but only delivers tree DFS
- [ ] Explain stack vs call stack connection for iterative versions
- [ ] Add concrete example for DFS vs BFS comparison
- [ ] Discuss DFS limitations (infinite loops without visited, bad for shortest paths)
- [ ] Improve informal opening ("It's very popular")

### Hashing (7/10)
**File:** `_posts/2023-04-24-hashing.md`
- [ ] **FIX:** Remove recommendation of MD5 and SHA-1 (cryptographically broken). Recommend xxHash, MurmurHash3, or SHA-256
- [ ] Add actual consistent hashing implementation (TreeMap-based ring)
- [ ] Explain virtual nodes concept in depth
- [ ] Replace oversimplified manual hash calculation with realistic example
- [ ] Explain how rendezvous hashing weights are computed

### Rate Limiting (7/10)
**File:** `_posts/2023-05-17-rate-limiting.md`
- [ ] Add token bucket or sliding window implementation code (e.g., Redis-based)
- [ ] Add comparison guidance: when to choose each of the 5 algorithms
- [ ] Reduce formal/wordy opening
- [ ] Connect DoS/DDoS section to the rest of the post (currently disconnected)
- [ ] Remove low-value analogies section at end
- [ ] Add distributed rate limiting challenges discussion

### Caching (System Design) (7/10)
**File:** `_posts/2023-04-14-caching.md`
- [ ] Add cache-aside pattern code example (Redis or Memcached)
- [ ] Clarify write-around strategy explanation
- [ ] Add thundering herd / cache stampede discussion
- [ ] Add cache warming strategies
- [ ] Replace passive opening ("is utilized to speed up")
- [ ] Better integrate CDN section with rest of post

### equals and hashCode (7/10)
**File:** `_posts/2021-09-05-java-equals-and-hashCode-contract.md`
- [ ] Expand `instanceof` vs `getClass()` into its own section with concrete examples
- [ ] Add HashMap example showing consequences of broken hashCode contract (duplicate keys)
- [ ] Remove `<br>` tag on line 49
- [ ] Strengthen conclusion (currently just repeats earlier content)
- [ ] Fix missing article on line 15

### Synchronized vs Volatile (7/10)
**File:** `_posts/2023-04-26-java-synchronized-vs-volatile.md`
- [ ] Add motivating opening (currently jumps straight into execution control definition)
- [ ] Show the race condition that volatile DOESN'T solve (`count++` read-modify-write)
- [ ] Add double-checked locking as a volatile use case
- [ ] Add visual diagram: thread caches vs main memory
- [ ] Simplify dense sentence on line 114 about happens-before
- [ ] Add performance comparison between synchronized vs volatile vs atomic

### Complexity Analysis (7.5/10)
**File:** `_posts/2022-11-14-complexity-analysis.md`
- [ ] Remove or update outdated "10^8 operations" rule of thumb
- [ ] Add compelling real-world motivation (why O(n^2) vs O(n log n) matters)
- [ ] Rewrite weak opening
- [ ] Clarify which space complexity definition you'll use
- [ ] Use consistent code formatting (pick Java, not pseudocode mix)

### Strings (7.5/10)
**File:** `_posts/2023-02-12-strings.md`
- [ ] Fix heading inconsistency (line 11: `####` should be `###`)
- [ ] Explain that `String.length()` counts UTF-16 code units, not characters (emoji bug)
- [ ] Add pathological regex example (`(a+)+b` against "aaaaaaaaaaaaa!")
- [ ] Clarify that `==` working for string literals is an accident of the string pool, not by design
- [ ] Mention Java 7 string pool migration from PermGen to heap
- [ ] Add StringBuilder vs StringBuffer distinction

### Comparable and Comparator (7.5/10)
**File:** `_posts/2021-09-04-comparable-and-comparator-interfaces.md`
- [ ] **FIX:** Remove copy-paste error on line 67 (`carSingleton()` from bean scopes post)
- [ ] Add TreeSet example showing what happens when compareTo is inconsistent with equals
- [ ] Add motivating example before jumping into definitions

### Load Balancers (7.5/10)
**File:** `_posts/2023-04-19-load-balancers.md`
- [ ] Remove "Pure Random" strategy (no real load balancer uses this) or label it as theoretical
- [ ] Add Nginx or HAProxy config snippet
- [ ] Add sticky sessions / session persistence discussion
- [ ] Add connection draining during deployments
- [ ] Quantify "limit to how much a single server can be enhanced"

### Polling and Streaming (7.5/10)
**File:** `_posts/2023-05-11-polling-and-streaming.md`
- [ ] Add WebSocket handshake code example
- [ ] Add SSE EventSource API example
- [ ] Add backpressure handling discussion
- [ ] Make "streaming via sockets" less abstract with concrete example
- [ ] Restructure to acknowledge long polling earlier in the progression

### Request-Response vs Pub-Sub (7.5/10)
**File:** `_posts/2023-09-23-request-response-vs-publish-subscribe.md`
- [ ] Add Kafka producer/consumer code example
- [ ] Explain what a partition is (currently assumes knowledge)
- [ ] Expand backpressure from parenthetical mention to its own section
- [ ] Clarify that request-response isn't always stateless (gRPC example)

---

## Priority 4: Minor Fixes (Score 8-8.5/10)

These posts are strong but have specific issues worth fixing.

### Why Are Tests Essential (8/10)
**File:** `_posts/2021-08-09-why-are-tests-essential.md`
- [ ] Fix image path typo: `writh` -> `write` and date mismatch (2021-08-09 vs 2021-09-09)
- [ ] Strengthen conclusion with a unique closing thought

### Observer Design Pattern (8/10)
**File:** `_posts/2021-08-16-observer-design-pattern.md`
- [ ] Add memory leak discussion (observers not unsubscribed, weak references)
- [ ] Add brief thread safety note (concurrent subscribe/notify)
- [ ] Show unsubscribe in action with code

### Latency and Throughput (8/10)
**File:** `_posts/2023-04-04-latency-and-throughput.md`
- [ ] Fix awkward opening header formatting (line 11)
- [ ] Add a real-world case study showing latency degradation under load
- [ ] Better integrate cross-links at end

### SQL vs NoSQL (8/10)
**File:** `_posts/2023-04-27-sql-vs-no-sql-databases.md`
- [ ] Add migration challenges: what happens when you chose wrong
- [ ] Add cost comparison of managed services
- [ ] Discuss operational complexity differences
- [ ] Explain implications of MongoDB transactions vs Postgres transactions

### System Design Compendium (8/10)
**File:** `_posts/2023-04-25-system-design-compendium.md`
- [ ] Improve generic intro
- [ ] Standardize description style across entries
- [ ] Add recommended reading order and prerequisites

### Static Factory Method (8/10)
**File:** `_posts/2022-08-22-static-factory-method.md`
- [ ] Add subtype selection code example (return different subtypes based on parameters)
- [ ] Add testing-focused factory methods (`createDefault()`, `createForTest()`)
- [ ] Clarify confusing sentence on line 46 about returned class not existing

### Java Memory Management (8/10)
**File:** `_posts/2024-02-25-java-memory-management.md`
- [ ] Add GC algorithm selection guidance ("For most web apps, use G1. Reach for ZGC only if...")
- [ ] Add diagnostic tools: VisualVM, heap dumps, jcmd
- [ ] Add common memory leak patterns in Java
- [ ] Mention reference types (soft, weak, phantom)
- [ ] Link to Pass by Value post to explain where references point

### Memory (Computer) (8/10)
**File:** `_posts/2022-11-28-memory.md`
- [ ] Fix inconsistent heading levels (line 57: `####` should be `###`)
- [ ] Add virtual memory / OS paging section
- [ ] Add JVM memory model mapping to physical RAM
- [ ] Fix table access times notation consistency
- [ ] Clarify cache line example phrasing ("next 15 integers" -> "16 elements total")

### Arrays (8/10)
**File:** `_posts/2022-12-29-arrays.md`
- [ ] Add proper introduction (currently jumps straight into "two types")
- [ ] Be specific about growth factors: Java ArrayList = 1.5x, others = 2x
- [ ] Explain worst-case resizing cost before introducing amortized analysis
- [ ] Mention variable sliding window pattern
- [ ] Follow own code comment style guide (`// Given`, `// When`, `// Then`)

### Hash Tables (8/10)
**File:** `_posts/2023-01-21-hash-tables.md`
- [ ] Lead opening with O(1) average-case guarantee
- [ ] Emphasize hashCode/equals consistency requirement (common bug source)
- [ ] Explain load factor 0.75 trade-off
- [ ] Fix unbounded static cache in Fibonacci memoization
- [ ] Connect rolling hash to Rabin-Karp algorithm
- [ ] Improve ASCII table visualization

### Stacks and Queues (8/10)
**File:** `_posts/2023-02-03-stacks-and-queues.md`
- [ ] Start with tangible example (undo/redo) instead of abstract ones
- [ ] Explain WHY ArrayDeque is faster (Stack extends synchronized Vector)
- [ ] Add concrete priority queue use case (hospital triage, job scheduling)
- [ ] Expand deque section with stack+queue usage example
- [ ] Mention circular buffers (ArrayDeque uses one internally)

### Microservices vs Monolith (8/10)
**File:** `_posts/2022-08-31-microservices-vs-monolith.md`
- [ ] Add modular monolith code example (Java package structure)
- [ ] Discuss "distributed monolith" anti-pattern
- [ ] Expand Strangler Fig pattern with implementation details
- [ ] Add database splitting guidance for modular monolith

### Why Business Doesn't Allow Refactoring (8/10)
**File:** `_posts/2022-01-30-why-business-sometimes-doesnt-allow-for-refactoring.md`
- [ ] Fix title grammar: "Why business is" -> "Why the business is" or "Why businesses are"
- [ ] Add concrete before/after case study with DORA metrics
- [ ] Address what to do when business still says no after presenting metrics
- [ ] Mention prevention through TDD, pair programming, code review

### Concurrency and Parallelism (8/10)
**File:** `_posts/2022-10-12-concurrency-and-parallelism.md`
- [ ] Add code examples for race condition, deadlock, and starvation (currently just mentioned)
- [ ] Replace `new Thread()` with ExecutorService/thread pool example
- [ ] Mention modern concurrency: virtual threads (Project Loom), CompletableFuture
- [ ] Clarify which workloads benefit from hyper-threading

### Core Concepts Behind OOP (8/10)
**File:** `_posts/2022-10-12-core-concepts-behind-oop.md`
- [ ] Fix BankAccount: validate non-negative initial balance (breaks own invariant)
- [ ] Complete StripePaymentProcessor: show how constructor dependencies are used
- [ ] Define abstract Propulsion class used in Car constructor
- [ ] Consider adding composition as complementary concept

### Composition over Inheritance (8/10)
**File:** `_posts/2023-01-09-prefer-composition-over-inheritance.md`
- [ ] Remove duplicate `#` heading on line 12 (conflicts with front matter title)
- [ ] Improve Vehicle/Bicycle inheritance example (Bicycle inheriting start() is a strawman)
- [ ] Add concrete example of when inheritance IS appropriate (AbstractList, template method)
- [ ] Show concrete encapsulation violation example

### Defining Software Architecture (8/10)
**File:** `_posts/2024-08-23-defining-software-architecture.md`
- [ ] Add ArchUnit fitness function code example
- [ ] Explain differences between similar characteristics (reliability vs availability, scalability vs elasticity)
- [ ] Add trade-off discussion: optimizing one characteristic sacrifices another
- [ ] Show concrete architecture decision example with code or diagram

### BigDecimal and BigInteger (8.5/10)
**File:** `_posts/2021-11-02-bigdecimal-and-biginteger.md`
- [ ] Clarify valueOf(double) vs new BigDecimal(String) phrasing (line 46)
- [ ] Add HashSet surprise note: `3.0` and `3.00` are different in equals but same in compareTo
- [ ] Expand BigInteger section slightly (primality testing is interesting but unexplored)

### Linked Lists (8.5/10)
**File:** `_posts/2023-01-07-linked-lists.md`
- [ ] **FIX:** Correct doubly linked list definition: it's about next AND previous pointers per node, not head/tail references (line 57)
- [ ] Quantify pointer memory overhead (4x on 64-bit systems vs arrays)
- [ ] Show what infinite loop mistake looks like in circular list traversal
- [ ] Add sentinel node discussion
- [ ] Ensure consistent null safety across all methods

### Specialized Storage Paradigms (8.5/10)
**File:** `_posts/2023-04-28-specialized-storage-paradigms.md`
- [ ] Quantify graph DB performance trade-offs (currently vague)
- [ ] Minor: already one of the strongest posts, use as template for others

---

## Priority 5: Already Strong (Score 9/10)

These posts need only cosmetic fixes. Use them as templates for rewriting weaker posts.

### CAP Theorem (9/10)
**File:** `_posts/2024-11-19-cap-theorem.md`
- [ ] Fix informal phrasing: "quite old" (line 16), "I guess" (line 109) -> state directly
- [ ] Add concrete database examples (Cassandra = AP, PostgreSQL with sync replication = CP)

### Peer-to-Peer Networks (9/10)
**File:** `_posts/2023-05-09-peer-to-peer-networks.md`
- [ ] Minor: expand IPFS content addressing detail
- [ ] Otherwise excellent, use as system design post template

### Proxies (9/10)
**File:** `_posts/2023-04-17-proxies.md`
- [ ] Add X-Forwarded-For header mention
- [ ] Consider adding Nginx reverse proxy config snippet
- [ ] Otherwise excellent

### Specification Design Pattern (9/10)
**File:** `_posts/2022-10-03-specification-design-pattern.md`
- [ ] Fill in `FieldSpecification` placeholder comments with actual implementation
- [ ] Consider adding one more real-world domain example
- [ ] Otherwise the best design pattern post in the collection

### @Mock vs @MockBean (9/10)
**File:** `_posts/2022-07-16-difference-between-mock-and-mock-bean.md`
- [ ] Replace `<br>` tags with markdown spacing
- [ ] Add concrete performance numbers (~50ms unit vs ~2-5s with @MockBean)
- [ ] Cross-reference Schools of Unit Testing post
- [ ] Fix orphaned footnote placement

### Economy of Testing (9/10)
**File:** `_posts/2024-05-10-economy-of-testing.md`
- [ ] Clarify cost-of-change figures: cite current consensus (5-15x, not just "up to 100x")
- [ ] Either expand Toyota/Knight Capital examples or cut in favor of more CrowdStrike depth
- [ ] Add one sentence about industry-specific risk profiles

### Is Java Pass by Value? (9/10)
**File:** `_posts/2023-02-20-is-java-pass-by-value-or-by-reference.md`
- [ ] Fix image alt text (currently just "img", should be descriptive for accessibility)
- [ ] Link to Java Memory Management post
- [ ] Otherwise template-quality teaching

### General Path of Refactoring (9/10)
**File:** `_posts/2022-07-18-general-path-of-refactoring.md`
- [ ] Refine ROI calculation: mention that savings compound across team (10 devs x 2 visits/week)
- [ ] Show `TestDataProvider` class or explain what it is
- [ ] Add missing `Money.percentage()` method implementation
- [ ] Consider mentioning Boy Scout Rule

### What is Coupling (9/10)
**File:** `_posts/2022-10-05-what-is-coupling.md`
- [ ] Replace inline HTML styles in table with cleaner markup
- [ ] Add data coupling discussion (shared global data structures)
- [ ] Consider mentioning connascence (richer vocabulary for coupling)

### IoC and Dependency Injection (9/10)
**File:** `_posts/2022-10-24-inversion-of-control-and-the-dependency-injection.md`
- [ ] Fix naming inconsistency: `SpotifySongsProvider` vs `SpotifyProvider`
- [ ] Add circular dependency discussion
- [ ] Consider adding Spring @Configuration class example tying everything together

### De Morgan's Laws (9/10)
**File:** `_posts/2021-11-01-demorgan's-laws.md`
- [ ] Add one nested condition example: `(a && b) || (c && d)`
- [ ] Mention short-circuit evaluation implications
- [ ] Consider mentioning IDE auto-application support

---

## Cross-Cutting Actions (Apply Everywhere)

### Grammar
- [ ] Audit all posts for missing articles (a, an, the) - known recurring issue
- [ ] Fix all `<br>` HTML tags -> proper markdown spacing
- [ ] Check for duplicate headings (front matter title duplicated as `#` in body)

### Structure
- [ ] Ensure all posts start with a problem, not a definition
- [ ] Add "When to use / When NOT to use" sections to all design pattern and technique posts
- [ ] Standardize "Related posts" format: use `> **Related posts**:` blockquotes consistently

### Code
- [ ] Verify all code examples compile and are correct
- [ ] Use `// Given`, `// When`, `// Then` comments in all test examples
- [ ] Replace outdated practices: `new Thread()` -> ExecutorService, `@Resource` -> `@Autowired`

### Content
- [ ] Add personal anecdotes/war stories to the weakest posts (they make the strongest posts shine)
- [ ] Replace canonical textbook examples where possible (Bird/Ostrich, Book/BookPrinter, Car builder)
- [ ] Cross-reference related posts consistently (testing posts should link each other, system design posts should link each other)

---

## Suggested Order of Work

1. Fix the 3 posts with actual bugs: **Strategy** (broken code), **Linked Lists** (wrong definition), **Hashing** (recommends broken algorithms)
2. Rewrite the 3 weakest posts: **Singleton**, **AssertJ**, **Network Protocols**
3. Add code to code-less posts: **Graphs**, **Replication/Sharding**, **Rate Limiting**, **Caching**
4. Complete the incomplete post: **Aggregate Pattern**
5. Enhance the 6/10 posts with structural improvements
6. Apply cross-cutting grammar and formatting fixes
7. Polish the 8-9/10 posts with minor fixes
