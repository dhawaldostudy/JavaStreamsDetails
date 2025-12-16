# 📘 The Ultimate Guide to Java Streams API
## From Basics to Expert Level 🚀

---

## 1. Introduction: What are Streams?

A **Stream** is a sequence of elements from a source that supports data processing operations. Introduced in Java 8, Streams provide a functional approach to processing collections of objects.

### Key Characteristics:

- **Declarative**: You specify *what* you want, not *how* to achieve it (no explicit loops)
- **Lazy Evaluation**: Intermediate operations are not executed until a terminal operation is invoked
- **Functional**: Streams don't modify the source; they produce new results
- **Possibly Unbounded**: Can work with infinite sequences
- **Consumable**: Can only be traversed once; attempting to reuse throws IllegalStateException

### The Anatomy of a Stream Pipeline

```
Source → Intermediate Operations → Terminal Operation
```

**Components:**
1. **Source**: Where data originates (Collection, Array, I/O, Generator)
2. **Intermediate Operations**: Transform the stream (lazy execution)
3. **Terminal Operation**: Triggers processing and produces a result (eager execution)

**Example:**
```java
List result = names.stream()           // Source
    .filter(name -> name.startsWith("A"))      // Intermediate
    .map(String::toUpperCase)                  // Intermediate
    .sorted()                                  // Intermediate
    .collect(Collectors.toList());             // Terminal
```

---

## 2. Creating Streams (The Source)

### 2.1 From Collections

```java
// List
List list = Arrays.asList("a", "b", "c");
Stream streamFromList = list.stream();

// Set
Set set = Set.of(1, 2, 3);
Stream streamFromSet = set.stream();

// Map (requires entry set)
Map map = Map.of("A", 1, "B", 2);
Stream<Map.Entry> streamFromMap = map.entrySet().stream();

// Parallel Stream
Stream parallelStream = list.parallelStream();
```

### 2.2 From Arrays

```java
// Array of objects
String[] array = {"Java", "Python", "C++"};
Stream streamFromArray = Arrays.stream(array);

// Primitive arrays
int[] numbers = {1, 2, 3, 4, 5};
IntStream intStream = Arrays.stream(numbers);

// Specify range
IntStream rangeStream = Arrays.stream(numbers, 1, 4); // [2, 3, 4]
```

### 2.3 From Values

```java
// Static values
Stream streamOfValues = Stream.of("A", "B", "C");

// Empty stream
Stream emptyStream = Stream.empty();

// Single element (nullable)
Stream streamOfNullable = Stream.ofNullable(getValue()); // Avoids NPE
```

### 2.4 Infinite Streams

```java
// Generate (Supplier-based)
Stream randomNumbers = Stream.generate(Math::random)
    .limit(5); // Must limit infinite streams!

// Iterate (seed + UnaryOperator)
Stream evenNumbers = Stream.iterate(0, n -> n + 2)
    .limit(10); // [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]

// Iterate with predicate (Java 9+)
Stream numbersBelow100 = Stream.iterate(0, n -> n < 100, n -> n + 5);
```

### 2.5 From Ranges (Primitive Streams)

```java
// IntStream range (exclusive end)
IntStream.range(1, 5);        // [1, 2, 3, 4]

// IntStream rangeClosed (inclusive end)
IntStream.rangeClosed(1, 5);  // [1, 2, 3, 4, 5]

// LongStream
LongStream.range(1L, 1000000L);

// Convert to Stream
Stream boxedStream = IntStream.range(1, 10).boxed();
```

### 2.6 From Files and I/O

```java
// Read lines from a file
try (Stream lines = Files.lines(Paths.get("data.txt"))) {
    lines.filter(line -> line.contains("error"))
         .forEach(System.out::println);
} catch (IOException e) {
    e.printStackTrace();
}

// List directory contents
try (Stream paths = Files.list(Paths.get("."))) {
    paths.filter(Files::isRegularFile)
         .forEach(System.out::println);
}
```

---

## 3. Intermediate Operations (Transforming Data)

Intermediate operations return a `Stream<T>` and are **lazy** (not executed until terminal operation).

### 3.1 Filtering Operations

#### filter(Predicate<T>)
Keeps elements that match the condition.

```java
List numbers = List.of(1, 2, 3, 4, 5, 6, 7, 8, 9, 10);

// Get even numbers
List evenNumbers = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList()); // [2, 4, 6, 8, 10]

// Multiple filters (chained)
List filtered = numbers.stream()
    .filter(n -> n > 3)
    .filter(n -> n < 8)
    .collect(Collectors.toList()); // [4, 5, 6, 7]
```

#### distinct()
Removes duplicates using `equals()` and `hashCode()`.

```java
List withDuplicates = List.of(1, 2, 2, 3, 3, 3, 4);
List unique = withDuplicates.stream()
    .distinct()
    .collect(Collectors.toList()); // [1, 2, 3, 4]

// Custom objects (override equals/hashCode)
List uniqueEmployees = employees.stream()
    .distinct() // Uses Employee's equals() method
    .collect(Collectors.toList());
```

#### limit(long n)
Truncates stream to first *n* elements (short-circuiting).

```java
List firstThree = IntStream.range(1, 100)
    .boxed()
    .limit(3)
    .collect(Collectors.toList()); // [1, 2, 3]

// Useful with infinite streams
Stream.generate(Math::random)
    .limit(5)
    .forEach(System.out::println);
```

#### skip(long n)
Discards the first *n* elements.

```java
List skipFirst = IntStream.range(1, 11)
    .skip(5)
    .boxed()
    .collect(Collectors.toList()); // [6, 7, 8, 9, 10]

// Pagination example
int page = 2;
int pageSize = 10;
List productsOnPage = products.stream()
    .skip((page - 1) * pageSize)
    .limit(pageSize)
    .collect(Collectors.toList());
```

#### takeWhile(Predicate) & dropWhile(Predicate) - Java 9+

```java
List numbers = List.of(1, 2, 3, 4, 5, 1, 2);

// takeWhile: stops at first false
List taken = numbers.stream()
    .takeWhile(n -> n < 4)
    .collect(Collectors.toList()); // [1, 2, 3]

// dropWhile: skips until first false
List dropped = numbers.stream()
    .dropWhile(n -> n < 4)
    .collect(Collectors.toList()); // [4, 5, 1, 2]
```

### 3.2 Mapping Operations (Transformation)

#### map(Function<T, R>)
One-to-One transformation. Converts each element.

```java
// String to uppercase
List names = List.of("alice", "bob", "charlie");
List upperNames = names.stream()
    .map(String::toUpperCase)
    .collect(Collectors.toList()); // [ALICE, BOB, CHARLIE]

// Extract property from objects
List employees = getEmployees();
List employeeNames = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.toList());

// Complex transformation
List lengths = names.stream()
    .map(String::length)
    .collect(Collectors.toList()); // [5, 3, 7]
```

#### flatMap(Function<T, Stream<R>>)
One-to-Many transformation. Flattens nested structures.

```java
// Flatten list of lists
List<List> nested = List.of(
    List.of("a", "b"),
    List.of("c", "d"),
    List.of("e")
);
List flattened = nested.stream()
    .flatMap(List::stream)
    .collect(Collectors.toList()); // [a, b, c, d, e]

// Split strings and flatten
List sentences = List.of("Hello World", "Java Streams");
List words = sentences.stream()
    .flatMap(sentence -> Arrays.stream(sentence.split(" ")))
    .collect(Collectors.toList()); // [Hello, World, Java, Streams]

// Real-world: Get all products from all orders
List orders = getOrders();
List allProducts = orders.stream()
    .flatMap(order -> order.getProducts().stream())
    .collect(Collectors.toList());
```

#### mapToInt / mapToLong / mapToDouble
Convert to primitive streams for performance.

```java
List strings = List.of("1", "2", "3");
int sum = strings.stream()
    .mapToInt(Integer::parseInt)
    .sum(); // 6

List employees = getEmployees();
double avgSalary = employees.stream()
    .mapToDouble(Employee::getSalary)
    .average()
    .orElse(0.0);
```

### 3.3 Sorting Operations

#### sorted()
Natural ordering (requires Comparable).

```java
List numbers = List.of(5, 2, 8, 1, 9);
List sorted = numbers.stream()
    .sorted()
    .collect(Collectors.toList()); // [1, 2, 5, 8, 9]

List words = List.of("zebra", "apple", "mango");
List sortedWords = words.stream()
    .sorted()
    .collect(Collectors.toList()); // [apple, mango, zebra]
```

#### sorted(Comparator<T>)
Custom sorting logic.

```java
// Reverse order
List reversed = numbers.stream()
    .sorted(Comparator.reverseOrder())
    .collect(Collectors.toList());

// Sort employees by salary
List sortedBySalary = employees.stream()
    .sorted(Comparator.comparing(Employee::getSalary))
    .collect(Collectors.toList());

// Multiple criteria
List sorted = employees.stream()
    .sorted(Comparator.comparing(Employee::getDepartment)
                      .thenComparing(Employee::getSalary).reversed())
    .collect(Collectors.toList());

// Sort strings by length
List byLength = words.stream()
    .sorted(Comparator.comparing(String::length))
    .collect(Collectors.toList());
```

### 3.4 Debugging Operations

#### peek(Consumer<T>)
Allows viewing elements as they flow through (for debugging only).

```java
List result = numbers.stream()
    .peek(n -> System.out.println("Original: " + n))
    .filter(n -> n > 5)
    .peek(n -> System.out.println("After filter: " + n))
    .map(n -> n * 2)
    .peek(n -> System.out.println("After map: " + n))
    .collect(Collectors.toList());

// WARNING: Don't modify state in peek()!
// Bad practice:
numbers.stream().peek(list::add); // DON'T DO THIS
```

---

## 4. Terminal Operations (Producing Results)

Terminal operations trigger the pipeline execution and close the stream.

### 4.1 Iteration

#### forEach(Consumer<T>)
Iterates over elements. Order not guaranteed for parallel streams.

```java
names.stream()
    .filter(name -> name.startsWith("A"))
    .forEach(System.out::println);

// Avoid modifying external state
List result = new ArrayList<>();
names.stream().forEach(result::add); // Not recommended

// Use forEachOrdered for parallel streams
names.parallelStream()
    .forEachOrdered(System.out::println); // Maintains order
```

### 4.2 Collection Operations

#### collect(Collector<T, A, R>)
The most versatile terminal operation.

```java
// To List
List list = stream.collect(Collectors.toList());
List unmodifiableList = stream.collect(Collectors.toUnmodifiableList());

// To Set
Set set = stream.collect(Collectors.toSet());

// To specific collection
ArrayList arrayList = stream.collect(Collectors.toCollection(ArrayList::new));
TreeSet treeSet = stream.collect(Collectors.toCollection(TreeSet::new));

// To Array
String[] array = stream.toArray(String[]::new);

// To Map
Map map = employees.stream()
    .collect(Collectors.toMap(
        Employee::getId,        // Key mapper
        Employee::getName       // Value mapper
    ));

// Handle duplicate keys
Map byName = employees.stream()
    .collect(Collectors.toMap(
        Employee::getName,
        Function.identity(),
        (existing, replacement) -> existing // Keep first
    ));

// To specific Map implementation
TreeMap treeMap = employees.stream()
    .collect(Collectors.toMap(
        Employee::getId,
        Employee::getName,
        (e1, e2) -> e1,
        TreeMap::new
    ));
```

### 4.3 Reduction Operations

#### reduce(BinaryOperator<T>)
Combines elements into a single value.

```java
// Sum
List numbers = List.of(1, 2, 3, 4, 5);
int sum = numbers.stream()
    .reduce(0, (a, b) -> a + b); // 15

// Using method reference
int sum2 = numbers.stream()
    .reduce(0, Integer::sum);

// Product
int product = numbers.stream()
    .reduce(1, (a, b) -> a * b); // 120

// Max value
Optional max = numbers.stream()
    .reduce(Integer::max);

// String concatenation
String combined = List.of("Hello", "World", "!").stream()
    .reduce("", (a, b) -> a + " " + b).trim();

// Complex object reduction
Optional highestPaid = employees.stream()
    .reduce((e1, e2) -> e1.getSalary() > e2.getSalary() ? e1 : e2);
```

#### Specialized Reductions

```java
// count()
long count = stream.count();

// min() / max()
Optional min = numbers.stream().min(Integer::compare);
Optional max = numbers.stream().max(Integer::compare);

// sum() / average() (primitive streams)
int sum = IntStream.of(1, 2, 3, 4, 5).sum();
OptionalDouble avg = IntStream.of(1, 2, 3, 4, 5).average();

// summaryStatistics()
IntSummaryStatistics stats = employees.stream()
    .mapToInt(Employee::getAge)
    .summaryStatistics();

System.out.println("Count: " + stats.getCount());
System.out.println("Sum: " + stats.getSum());
System.out.println("Min: " + stats.getMin());
System.out.println("Max: " + stats.getMax());
System.out.println("Average: " + stats.getAverage());
```

### 4.4 Matching Operations (Short-Circuiting)

These stop processing as soon as the result is determined.

```java
List numbers = List.of(1, 2, 3, 4, 5);

// anyMatch: At least one element matches
boolean hasEven = numbers.stream()
    .anyMatch(n -> n % 2 == 0); // true

// allMatch: All elements match
boolean allPositive = numbers.stream()
    .allMatch(n -> n > 0); // true

// noneMatch: No elements match
boolean noNegative = numbers.stream()
    .noneMatch(n -> n < 0); // true

// Real-world example
boolean hasHighSalary = employees.stream()
    .anyMatch(e -> e.getSalary() > 100000);
```

### 4.5 Finding Operations (Short-Circuiting)

```java
List names = List.of("Alice", "Bob", "Charlie");

// findFirst: Returns first element
Optional first = names.stream()
    .filter(name -> name.startsWith("C"))
    .findFirst(); // Optional[Charlie]

// findAny: Returns any element (non-deterministic in parallel)
Optional any = names.parallelStream()
    .filter(name -> name.length() > 3)
    .findAny();

// Usage with orElse
String name = names.stream()
    .filter(n -> n.startsWith("Z"))
    .findFirst()
    .orElse("Not Found");

// orElseThrow
String required = names.stream()
    .filter(n -> n.equals("Alice"))
    .findFirst()
    .orElseThrow(() -> new RuntimeException("Alice not found"));
```

---

## 5. The Collectors API (The Powerhouse 🔥)

### 5.1 Basic Collectors

#### joining()
Concatenates strings.

```java
List words = List.of("Java", "Streams", "API");

// Simple join
String result = words.stream()
    .collect(Collectors.joining()); // "JavaStreamsAPI"

// With delimiter
String withComma = words.stream()
    .collect(Collectors.joining(", ")); // "Java, Streams, API"

// With prefix and suffix
String formatted = words.stream()
    .collect(Collectors.joining(", ", "[", "]")); // "[Java, Streams, API]"
```

### 5.2 Grouping Operations

#### groupingBy()
Most common interview pattern - groups elements by a classifier.

```java
List employees = getEmployees();

// Simple grouping
Map> byDepartment = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));

// Multi-level grouping
Map>> byDeptAndCity = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.groupingBy(Employee::getCity)
    ));

// Custom key
Map> bySalaryRange = employees.stream()
    .collect(Collectors.groupingBy(e -> {
        if (e.getSalary() < 50000) return "Low";
        else if (e.getSalary() < 100000) return "Medium";
        else return "High";
    }));
```

### 5.3 Downstream Collectors

Perform additional operations within groups.

```java
// Counting
Map countByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.counting()
    ));

// Summing
Map totalSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.summingDouble(Employee::getSalary)
    ));

// Averaging
Map avgSalaryByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.averagingDouble(Employee::getSalary)
    ));

// Max/Min
Map> highestPaidByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.maxBy(Comparator.comparing(Employee::getSalary))
    ));

// Mapping (transform after grouping)
Map> employeeNamesByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.mapping(Employee::getName, Collectors.toList())
    ));

// Collecting to Set
Map> uniqueCitiesByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.mapping(Employee::getCity, Collectors.toSet())
    ));

// Multiple downstream collectors
Map statsByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.summarizingDouble(Employee::getSalary)
    ));
```

### 5.4 Partitioning

Splits data into exactly **two** groups based on a boolean predicate.

```java
// Basic partitioning
Map> partitioned = employees.stream()
    .collect(Collectors.partitioningBy(e -> e.getSalary() > 100000));

List highSalary = partitioned.get(true);
List lowSalary = partitioned.get(false);

// With downstream collector
Map countPartitioned = employees.stream()
    .collect(Collectors.partitioningBy(
        e -> e.getSalary() > 100000,
        Collectors.counting()
    ));

// Partition and find max
Map> topInEachPartition = employees.stream()
    .collect(Collectors.partitioningBy(
        e -> e.getAge() > 30,
        Collectors.maxBy(Comparator.comparing(Employee::getSalary))
    ));
```

### 5.5 Advanced Collectors

#### collectingAndThen()
Applies a finishing transformation after collecting.

```java
// Get unmodifiable list
List immutableList = names.stream()
    .collect(Collectors.collectingAndThen(
        Collectors.toList(),
        Collections::unmodifiableList
    ));

// Count and convert to int
int count = employees.stream()
    .collect(Collectors.collectingAndThen(
        Collectors.counting(),
        Long::intValue
    ));

// Find highest salary and unwrap Optional
Employee topEarner = employees.stream()
    .collect(Collectors.collectingAndThen(
        Collectors.maxBy(Comparator.comparing(Employee::getSalary)),
        Optional::get
    ));
```

#### toMap() with merging

```java
// Merge duplicate keys
Map merged = employees.stream()
    .collect(Collectors.toMap(
        Employee::getDepartment,
        Employee::getAge,
        (age1, age2) -> age1 + age2  // Sum ages for same department
    ));

// Last wins strategy
Map lastEmployeeByDept = employees.stream()
    .collect(Collectors.toMap(
        Employee::getDepartment,
        Function.identity(),
        (e1, e2) -> e2  // Keep last
    ));
```

---

## 6. Real-World Interview Scenarios

### Scenario 1: Nth Highest Salary

Find the 2nd highest salary in the company.

```java
// Method 1: Using sorted() and skip()
double secondHighest = employees.stream()
    .mapToDouble(Employee::getSalary)
    .distinct()
    .boxed()
    .sorted(Comparator.reverseOrder())
    .skip(1)
    .findFirst()
    .orElseThrow();

// Method 2: Using sorted set
double secondHighest2 = employees.stream()
    .map(Employee::getSalary)
    .collect(Collectors.toCollection(TreeSet::new))
    .descendingSet()
    .stream()
    .skip(1)
    .findFirst()
    .orElseThrow();

// Generic: Nth highest
int n = 3;
double nthHighest = employees.stream()
    .map(Employee::getSalary)
    .distinct()
    .sorted(Comparator.reverseOrder())
    .skip(n - 1)
    .findFirst()
    .orElseThrow();
```

### Scenario 2: Department-wise Highest Paid Employee

```java
Map> topEarnerByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.maxBy(Comparator.comparing(Employee::getSalary))
    ));

// Without Optional wrapper
Map topEarnerByDeptUnwrapped = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.collectingAndThen(
            Collectors.maxBy(Comparator.comparing(Employee::getSalary)),
            Optional::get
        )
    ));
```

### Scenario 3: Character Frequency Count

```java
String input = "banana";

// Method 1: Using split
Map charFreq = Arrays.stream(input.split(""))
    .collect(Collectors.groupingBy(
        Function.identity(),
        Collectors.counting()
    ));
// {b=1, a=3, n=2}

// Method 2: Using chars()
Map charFreq2 = input.chars()
    .mapToObj(c -> (char) c)
    .collect(Collectors.groupingBy(
        Function.identity(),
        Collectors.counting()
    ));
```

### Scenario 4: First Non-Repeating Character

```java
String str = "aabbcde";

Optional firstNonRepeating = str.chars()
    .mapToObj(c -> (char) c)
    .collect(Collectors.groupingBy(
        Function.identity(),
        LinkedHashMap::new,  // Preserve order
        Collectors.counting()
    ))
    .entrySet()
    .stream()
    .filter(entry -> entry.getValue() == 1)
    .map(Map.Entry::getKey)
    .findFirst();
// Optional[c]
```

### Scenario 5: Anagram Grouping

Group words that are anagrams of each other.

```java
List words = List.of("listen", "silent", "hello", "world", "enlist");

Map> anagramGroups = words.stream()
    .collect(Collectors.groupingBy(word -> {
        char[] chars = word.toCharArray();
        Arrays.sort(chars);
        return new String(chars);
    }));
// {eilnst=[listen, silent, enlist], ehllo=[hello], dlorw=[world]}
```

### Scenario 6: Employee Age Statistics by Department

```java
Map ageStatsByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.summarizingDouble(Employee::getAge)
    ));

ageStatsByDept.forEach((dept, stats) -> {
    System.out.println(dept + ":");
    System.out.println("  Count: " + stats.getCount());
    System.out.println("  Average Age: " + stats.getAverage());
    System.out.println("  Min Age: " + stats.getMin());
    System.out.println("  Max Age: " + stats.getMax());
});
```

### Scenario 7: Total Transaction Amount by Type

```java
class Transaction {
    enum Type { CREDIT, DEBIT }
    Type type;
    double amount;
}

List transactions = getTransactions();

Map totalByType = transactions.stream()
    .collect(Collectors.groupingBy(
        Transaction::getType,
        Collectors.summingDouble(Transaction::getAmount)
    ));

// Net balance
double netBalance = transactions.stream()
    .mapToDouble(t -> t.getType() == Transaction.Type.CREDIT 
        ? t.getAmount() 
        : -t.getAmount())
    .sum();
```

### Scenario 8: Comma-Separated Employee Names by Department

```java
Map employeeNamesByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.mapping(
            Employee::getName,
            Collectors.joining(", ")
        )
    ));
// {IT=John, Alice, Bob, HR=Mary, Steve}
```

### Scenario 9: List of Lists to Single List (FlatMap)

```java
List orders = getOrders();

// Get all products from all orders
List allProducts = orders.stream()
    .flatMap(order -> order.getProducts().stream())
    .collect(Collectors.toList());

// Calculate total cost
double totalCost = orders.stream()
    .flatMap(order -> order.getProducts().stream())
    .mapToDouble(Product::getPrice)
    .sum();

// Unique product names
Set uniqueProductNames = orders.stream()
    .flatMap(order -> order.getProducts().stream())
    .map(Product::getName)
    .collect(Collectors.toSet());
```

### Scenario 10: Partition Primes

```java
public static boolean isPrime(int number) {
    return number > 1 && IntStream.rangeClosed(2, (int) Math.sqrt(number))
        .noneMatch(i -> number % i == 0);
}

List numbers = IntStream.rangeClosed(1, 100)
    .boxed()
    .collect(Collectors.toList());

Map> partitioned = numbers.stream()
    .collect(Collectors.partitioningBy(n -> isPrime(n)));

List primes = partitioned.get(true);
List nonPrimes = partitioned.get(false);
```

---
# Java Streams API - Performance Considerations & Advanced Topics

## 7. Performance Considerations

### 7.1 Parallel Streams

Parallel streams split data into multiple chunks and process them concurrently using the ForkJoinPool.

#### When to Use Parallel Streams

```java
List numbers = IntStream.rangeClosed(1, 1000000)
    .boxed()
    .collect(Collectors.toList());

// Sequential
long startSeq = System.currentTimeMillis();
long sumSeq = numbers.stream()
    .mapToLong(Integer::longValue)
    .sum();
long endSeq = System.currentTimeMillis();
System.out.println("Sequential: " + (endSeq - startSeq) + "ms");

// Parallel
long startPar = System.currentTimeMillis();
long sumPar = numbers.parallelStream()
    .mapToLong(Integer::longValue)
    .sum();
long endPar = System.currentTimeMillis();
System.out.println("Parallel: " + (endPar - startPar) + "ms");
```

#### ✅ Good Use Cases for Parallel Streams

```java
// 1. Large datasets (10,000+ elements)
List largeList = IntStream.rangeClosed(1, 100000)
    .boxed()
    .collect(Collectors.toList());

// 2. CPU-intensive operations
List result = largeList.parallelStream()
    .filter(n -> isPrime(n))  // Expensive computation
    .collect(Collectors.toList());

// 3. Independent operations (no shared state)
double average = employees.parallelStream()
    .mapToDouble(Employee::getSalary)
    .average()
    .orElse(0.0);

// 4. Stateless operations
List upperCase = names.parallelStream()
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

#### ❌ Bad Use Cases for Parallel Streams

```java
// 1. Small datasets (overhead > benefit)
List smallList = List.of(1, 2, 3, 4, 5);
// Don't parallelize - overhead not worth it
smallList.parallelStream().forEach(System.out::println);

// 2. I/O operations (threads will wait for I/O)
// BAD: Parallel doesn't help with I/O-bound operations
files.parallelStream()
    .forEach(file -> {
        try {
            String content = Files.readString(file); // I/O blocking
        } catch (IOException e) {}
    });

// 3. Operations with shared mutable state
List sharedList = new ArrayList<>(); // NOT thread-safe!
// WRONG - Race condition!
numbers.parallelStream()
    .forEach(sharedList::add); // Concurrent modification!

// 4. Order-dependent operations
// Result may be unpredictable
List ordered = numbers.parallelStream()
    .limit(10)  // Which 10 elements?
    .collect(Collectors.toList());

// 5. Operations that require sequential processing
// BAD: findFirst() defeats parallel purpose
Optional first = numbers.parallelStream()
    .filter(n -> n > 100)
    .findFirst(); // Forces sequential constraint
```

#### Parallel Stream Best Practices

```java
// Use forEachOrdered() to maintain order
numbers.parallelStream()
    .filter(n -> n % 2 == 0)
    .forEachOrdered(System.out::println); // Maintains encounter order

// Use concurrent collectors
Map> byDept = employees.parallelStream()
    .collect(Collectors.groupingByConcurrent(Employee::getDepartment));

// Avoid side effects
// WRONG
int[] sum = {0};
numbers.parallelStream()
    .forEach(n -> sum[0] += n); // Race condition!

// CORRECT
int sum = numbers.parallelStream()
    .mapToInt(Integer::intValue)
    .sum(); // Thread-safe reduction

// Control parallelism level (if needed)
ForkJoinPool customPool = new ForkJoinPool(4); // 4 threads
try {
    customPool.submit(() -> 
        numbers.parallelStream()
            .map(n -> n * 2)
            .collect(Collectors.toList())
    ).get();
} catch (Exception e) {
    e.printStackTrace();
} finally {
    customPool.shutdown();
}
```

### 7.2 Lazy Evaluation Benefits

Streams don't execute intermediate operations until a terminal operation is called.

```java
// No computation happens here
Stream stream = numbers.stream()
    .filter(n -> {
        System.out.println("Filtering: " + n);
        return n > 5;
    })
    .map(n -> {
        System.out.println("Mapping: " + n);
        return n * 2;
    });

// Computation starts only here
List result = stream.collect(Collectors.toList());

// Short-circuiting with lazy evaluation
Optional first = IntStream.range(1, 1000000)
    .filter(n -> n > 500)
    .boxed()
    .findFirst(); // Stops after finding first match, doesn't process all 1M
```

### 7.3 Primitive Streams for Performance

Use primitive streams to avoid boxing/unboxing overhead.

```java
// ❌ BAD: Boxing overhead
List numbers = List.of(1, 2, 3, 4, 5);
int sum = numbers.stream()
    .mapToInt(Integer::intValue) // Unboxing
    .sum(); // More efficient, but initial stream boxes

// ✅ GOOD: Use primitive stream from start
int sum = IntStream.rangeClosed(1, 1000000)
    .sum(); // No boxing/unboxing

// ❌ BAD: Unnecessary boxing
double average = IntStream.range(1, 1000)
    .boxed() // Converts to Stream
    .mapToDouble(Integer::doubleValue)
    .average()
    .orElse(0.0);

// ✅ GOOD: Stay primitive
double average = IntStream.range(1, 1000)
    .average()
    .orElse(0.0);

// Primitive stream operations are optimized
long sum = LongStream.rangeClosed(1, 1000000)
    .parallel()
    .sum(); // Highly optimized

// Convert between primitive streams
IntStream.range(1, 10)
    .asLongStream()   // IntStream → LongStream
    .asDoubleStream() // LongStream → DoubleStream
    .sum();
```

### 7.4 Avoid Unnecessary Operations

```java
// ❌ BAD: Multiple passes over data
List employees = getEmployees();
List names = employees.stream()
    .map(Employee::getName)
    .collect(Collectors.toList());

List filtered = names.stream()
    .filter(name -> name.startsWith("A"))
    .collect(Collectors.toList());

// ✅ GOOD: Single pass
List filtered = employees.stream()
    .map(Employee::getName)
    .filter(name -> name.startsWith("A"))
    .collect(Collectors.toList());

// ❌ BAD: Unnecessary intermediate collection
long count = numbers.stream()
    .filter(n -> n > 10)
    .collect(Collectors.toList())
    .size(); // Materializes list just to count

// ✅ GOOD: Use count() directly
long count = numbers.stream()
    .filter(n -> n > 10)
    .count();

// ❌ BAD: Calling stream() multiple times
employees.stream().filter(e -> e.getAge() > 30).count();
employees.stream().filter(e -> e.getAge() > 30).collect(Collectors.toList());

// ✅ GOOD: Reuse filtered result if needed multiple times
List filtered = employees.stream()
    .filter(e -> e.getAge() > 30)
    .collect(Collectors.toList());
long count = filtered.size();
```

### 7.5 Stream vs Traditional Loops - Performance Comparison

```java
List numbers = IntStream.rangeClosed(1, 1000000)
    .boxed()
    .collect(Collectors.toList());

// Traditional for loop
long start = System.nanoTime();
int sum = 0;
for (int n : numbers) {
    if (n % 2 == 0) {
        sum += n * 2;
    }
}
long end = System.nanoTime();
System.out.println("For loop: " + (end - start) / 1_000_000 + "ms");

// Stream (sequential)
start = System.nanoTime();
sum = numbers.stream()
    .filter(n -> n % 2 == 0)
    .mapToInt(n -> n * 2)
    .sum();
end = System.nanoTime();
System.out.println("Stream: " + (end - start) / 1_000_000 + "ms");

// Stream (parallel)
start = System.nanoTime();
sum = numbers.parallelStream()
    .filter(n -> n % 2 == 0)
    .mapToInt(n -> n * 2)
    .sum();
end = System.nanoTime();
System.out.println("Parallel: " + (end - start) / 1_000_000 + "ms");

// Guidelines:
// - For simple operations on small lists: Traditional loops often faster
// - For complex operations: Streams more readable, similar performance
// - For large datasets with CPU-intensive ops: Parallel streams win
```

---

## 8. Common Pitfalls and How to Avoid Them

### 8.1 Reusing Streams

```java
// ❌ WRONG: Stream already consumed
Stream stream = names.stream();
long count = stream.count();
List list = stream.collect(Collectors.toList()); // IllegalStateException!

// ✅ CORRECT: Create new stream
long count = names.stream().count();
List list = names.stream().collect(Collectors.toList());

// ✅ CORRECT: Use Supplier for reusable stream source
Supplier<Stream> streamSupplier = () -> names.stream();
long count = streamSupplier.get().count();
List list = streamSupplier.get().collect(Collectors.toList());
```

### 8.2 Modifying Source During Stream Operations

```java
List numbers = new ArrayList<>(List.of(1, 2, 3, 4, 5));

// ❌ WRONG: ConcurrentModificationException
numbers.stream()
    .forEach(n -> {
        if (n % 2 == 0) {
            numbers.remove(n); // Modifying source!
        }
    });

// ✅ CORRECT: Collect results first, then modify
List toRemove = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());
numbers.removeAll(toRemove);

// ✅ CORRECT: Use removeIf
numbers.removeIf(n -> n % 2 == 0);
```

### 8.3 Null Handling in Streams

```java
List names = Arrays.asList("Alice", null, "Bob", null, "Charlie");

// ❌ WRONG: NullPointerException
names.stream()
    .map(String::toUpperCase) // NPE on null!
    .forEach(System.out::println);

// ✅ CORRECT: Filter nulls
names.stream()
    .filter(Objects::nonNull)
    .map(String::toUpperCase)
    .forEach(System.out::println);

// ✅ CORRECT: Use Optional
names.stream()
    .map(name -> Optional.ofNullable(name)
                         .map(String::toUpperCase)
                         .orElse("UNKNOWN"))
    .forEach(System.out::println);

// Java 9+: Stream.ofNullable()
Stream.ofNullable(getValue()) // Avoids NPE
    .forEach(System.out::println);
```

### 8.4 Collectors.toMap() Duplicate Key Issues

```java
List employees = getEmployees();

// ❌ WRONG: IllegalStateException if duplicate keys
Map map = employees.stream()
    .collect(Collectors.toMap(
        Employee::getDepartment,
        Function.identity()
    )); // Fails if 2+ employees in same department

// ✅ CORRECT: Provide merge function
Map map = employees.stream()
    .collect(Collectors.toMap(
        Employee::getDepartment,
        Function.identity(),
        (existing, replacement) -> existing // Keep first
    ));

// ✅ CORRECT: Group instead
Map> map = employees.stream()
    .collect(Collectors.groupingBy(Employee::getDepartment));
```

### 8.5 Side Effects in Stream Operations

```java
List result = new ArrayList<>();

// ❌ BAD PRACTICE: Side effect in forEach
numbers.stream()
    .filter(n -> n % 2 == 0)
    .forEach(result::add); // External state modification

// ✅ GOOD: Use collect
List result = numbers.stream()
    .filter(n -> n % 2 == 0)
    .collect(Collectors.toList());

// ❌ BAD: Side effect in map
List processed = new ArrayList<>();
names.stream()
    .map(name -> {
        processed.add(name); // Side effect!
        return name.toUpperCase();
    })
    .collect(Collectors.toList());

// ✅ GOOD: Use peek only for debugging
names.stream()
    .peek(name -> System.out.println("Processing: " + name))
    .map(String::toUpperCase)
    .collect(Collectors.toList());
```

### 8.6 Optional Misuse

```java
// ❌ BAD: Using get() without checking
Optional optional = names.stream().findFirst();
String result = optional.get(); // NoSuchElementException if empty!

// ✅ GOOD: Use orElse/orElseGet/orElseThrow
String result = optional.orElse("Default");
String result = optional.orElseGet(() -> computeDefault());
String result = optional.orElseThrow(() -> new RuntimeException("Not found"));

// ✅ GOOD: Use ifPresent
optional.ifPresent(value -> System.out.println(value));

// ❌ BAD: Unnecessary Optional wrapping
Optional opt = Optional.of(names.stream()
    .filter(n -> n.startsWith("A"))
    .findFirst()
    .orElse("Default")); // Redundant nesting

// ✅ GOOD: findFirst already returns Optional
Optional opt = names.stream()
    .filter(n -> n.startsWith("A"))
    .findFirst();
```

### 8.7 Forgetting to Call Terminal Operation

```java
// ❌ WRONG: Nothing happens!
numbers.stream()
    .filter(n -> n > 10)
    .map(n -> n * 2); // No terminal operation - stream not executed

// ✅ CORRECT: Add terminal operation
List result = numbers.stream()
    .filter(n -> n > 10)
    .map(n -> n * 2)
    .collect(Collectors.toList()); // Triggers execution
```

---

## 9. Advanced Stream Patterns

### 9.1 Custom Collectors

Create your own collectors for specialized operations.

```java
// Custom collector to join strings with custom logic
Collector customJoiner = Collector.of(
    () -> new StringJoiner(", ", "[", "]"),     // Supplier
    StringJoiner::add,                           // Accumulator
    StringJoiner::merge,                         // Combiner
    StringJoiner::toString                       // Finisher
);

String result = names.stream()
    .collect(customJoiner);

// Custom collector for immutable list
public static  Collector> toImmutableList() {
    return Collectors.collectingAndThen(
        Collectors.toList(),
        Collections::unmodifiableList
    );
}

List immutable = names.stream()
    .collect(toImmutableList());

// Custom collector for statistical analysis
public static class Stats {
    private double sum;
    private long count;
    private double min = Double.MAX_VALUE;
    private double max = Double.MIN_VALUE;
    
    public void accept(double value) {
        sum += value;
        count++;
        min = Math.min(min, value);
        max = Math.max(max, value);
    }
    
    public Stats combine(Stats other) {
        sum += other.sum;
        count += other.count;
        min = Math.min(min, other.min);
        max = Math.max(max, other.max);
        return this;
    }
    
    public double getAverage() {
        return count > 0 ? sum / count : 0;
    }
}
```

### 9.2 Stream Concatenation

```java
// Concatenate multiple streams
Stream stream1 = Stream.of("A", "B");
Stream stream2 = Stream.of("C", "D");
Stream stream3 = Stream.of("E", "F");

Stream combined = Stream.concat(
    Stream.concat(stream1, stream2),
    stream3
);

// Better for multiple streams
Stream combined = Stream.of(stream1, stream2, stream3)
    .flatMap(Function.identity());

// Concatenate collections
List list1 = List.of("A", "B");
List list2 = List.of("C", "D");

List merged = Stream.concat(list1.stream(), list2.stream())
    .collect(Collectors.toList());

// Using flatMap for multiple collections
List merged = Stream.of(list1, list2, list3)
    .flatMap(Collection::stream)
    .collect(Collectors.toList());
```

### 9.3 Conditional Stream Operations

```java
// Dynamic filtering based on condition
public Stream filterEmployees(
    Stream stream, 
    boolean filterBySalary,
    boolean filterByAge) {
    
    if (filterBySalary) {
        stream = stream.filter(e -> e.getSalary() > 50000);
    }
    if (filterByAge) {
        stream = stream.filter(e -> e.getAge() > 25);
    }
    return stream;
}

// Better approach using functional composition
Predicate salaryFilter = e -> e.getSalary() > 50000;
Predicate ageFilter = e -> e.getAge() > 25;

Predicate combinedFilter = employee -> true;
if (filterBySalary) combinedFilter = combinedFilter.and(salaryFilter);
if (filterByAge) combinedFilter = combinedFilter.and(ageFilter);

List filtered = employees.stream()
    .filter(combinedFilter)
    .collect(Collectors.toList());
```

### 9.4 Stream Teeing (Java 12+)

Process a stream in two different ways simultaneously.

```java
// Calculate average and max in one pass
record Result(double average, int max) {}

Result result = numbers.stream()
    .collect(Collectors.teeing(
        Collectors.averagingDouble(Integer::doubleValue),
        Collectors.maxBy(Integer::compare),
        (avg, max) -> new Result(avg, max.orElse(0))
    ));

// Calculate statistics by department in parallel
Map statsByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.teeing(
            Collectors.averagingDouble(Employee::getSalary),
            Collectors.counting(),
            (avgSalary, count) -> new Result(avgSalary, count.intValue())
        )
    ));
```

### 9.5 Infinite Streams with Control

```java
// Generate Fibonacci sequence
Stream.iterate(new int[]{0, 1}, arr -> new int[]{arr[1], arr[0] + arr[1]})
    .map(arr -> arr[0])
    .limit(10)
    .forEach(System.out::println);

// Generate random numbers until condition
Random random = new Random();
Stream.generate(random::nextInt)
    .filter(n -> n > 0 && n < 100)
    .limit(5)
    .forEach(System.out::println);

// Infinite stream with takeWhile (Java 9+)
Stream.iterate(1, n -> n + 1)
    .takeWhile(n -> n <= 100)
    .filter(n -> n % 2 == 0)
    .forEach(System.out::println);

// Generate dates
LocalDate startDate = LocalDate.now();
Stream.iterate(startDate, date -> date.plusDays(1))
    .limit(30)
    .forEach(System.out::println);
```

---

## 10. Practice Challenges with Solutions

### Challenge 1: Square & Filter
**Problem**: Square all numbers and return only those greater than 100.

```java
List numbers = List.of(5, 8, 12, 15, 20);

List result = numbers.stream()
    .map(n -> n * n)
    .filter(n -> n > 100)
    .collect(Collectors.toList());
// Result: [144, 225, 400]
```

### Challenge 2: Longest String
**Problem**: Find the longest string in a list.

```java
List words = List.of("Java", "Python", "JavaScript", "C++");

String longest = words.stream()
    .max(Comparator.comparing(String::length))
    .orElse("");
// Result: "JavaScript"

// Alternative
String longest = words.stream()
    .reduce((w1, w2) -> w1.length() > w2.length() ? w1 : w2)
    .orElse("");
```

### Challenge 3: Oldest Employee by Department
**Problem**: Find the oldest employee in each department.

```java
Map> oldestByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.maxBy(Comparator.comparing(Employee::getAge))
    ));

// Without Optional wrapper
Map oldestByDept = employees.stream()
    .collect(Collectors.groupingBy(
        Employee::getDepartment,
        Collectors.collectingAndThen(
            Collectors.maxBy(Comparator.comparing(Employee::getAge)),
            Optional::get
        )
    ));
```

### Challenge 4: Comma-Separated String
**Problem**: Convert List<String> to "A, B, C".

```java
List items = List.of("A", "B", "C");

String result = items.stream()
    .collect(Collectors.joining(", "));
// Result: "A, B, C"

// With brackets
String result = items.stream()
    .collect(Collectors.joining(", ", "[", "]"));
// Result: "[A, B, C]"
```

### Challenge 5: Summary Statistics
**Problem**: Get count, sum, min, max, and average in one operation.

```java
List salaries = List.of(50000, 60000, 75000, 90000, 120000);

IntSummaryStatistics stats = salaries.stream()
    .mapToInt(Integer::intValue)
    .summaryStatistics();

System.out.println("Count: " + stats.getCount());
System.out.println("Sum: " + stats.getSum());
System.out.println("Min: " + stats.getMin());
System.out.println("Max: " + stats.getMax());
System.out.println("Average: " + stats.getAverage());
```

### Challenge 6: Partition Primes
**Problem**: Partition numbers into primes and non-primes.

```java
public static boolean isPrime(int n) {
    if (n <= 1) return false;
    return IntStream.rangeClosed(2, (int) Math.sqrt(n))
        .noneMatch(i -> n % i == 0);
}

List numbers = IntStream.rangeClosed(1, 50)
    .boxed()
    .collect(Collectors.toList());

Map> partitioned = numbers.stream()
    .collect(Collectors.partitioningBy(n -> isPrime(n)));

List primes = partitioned.get(true);
List nonPrimes = partitioned.get(false);
```

### Challenge 7: FlatMap - Total Cost of All Products
**Problem**: Calculate total cost from List<Order> containing List<Product>.

```java
class Order {
    private List products;
    public List getProducts() { return products; }
}

class Product {
    private double price;
    public double getPrice() { return price; }
}

List orders = getOrders();

double totalCost = orders.stream()
    .flatMap(order -> order.getProducts().stream())
    .mapToDouble(Product::getPrice)
    .sum();
```

### Challenge 8: First Non-Repeating Character
**Problem**: Find first character that appears only once.

```java
String input = "aabbcdeff";

Optional firstNonRepeating = input.chars()
    .mapToObj(c -> (char) c)
    .collect(Collectors.groupingBy(
        Function.identity(),
        LinkedHashMap::new,  // Maintains order
        Collectors.counting()
    ))
    .entrySet()
    .stream()
    .filter(entry -> entry.getValue() == 1)
    .map(Map.Entry::getKey)
    .findFirst();

System.out.println(firstNonRepeating.orElse('_')); // 'c'
```

### Challenge 9: Fibonacci Sequence
**Problem**: Generate first 20 Fibonacci numbers.

```java
List fibonacci = Stream.iterate(
    new long[]{0, 1},
    arr -> new long[]{arr[1], arr[0] + arr[1]}
)
.limit(20)
.map(arr -> arr[0])
.collect(Collectors.toList());

System.out.println(fibonacci);
// [0, 1, 1, 2, 3, 5, 8, 13, 21, 34, 55, 89, 144, 233, 377, 610, 987, 1597, 2584, 4181]
```

### Challenge 10: Transaction Net Balance
**Problem**: Calculate net balance from DEBIT/CREDIT transactions.

```java
enum TransactionType { CREDIT, DEBIT }

class Transaction {
    private TransactionType type;
    private double amount;
    
    public TransactionType getType() { return type; }
    public double getAmount() { return amount; }
}

List transactions = getTransactions();

double netBalance = transactions.stream()
    .mapToDouble(t -> t.getType() == TransactionType.CREDIT 
        ? t.getAmount() 
        : -t.getAmount())
    .sum();

// Alternatively, group by type first
Map totalsByType = transactions.stream()
    .collect(Collectors.groupingBy(
        Transaction::getType,
        Collectors.summingDouble(Transaction::getAmount)
    ));

double balance = totalsByType.getOrDefault(TransactionType.CREDIT, 0.0) 
               - totalsByType.getOrDefault(TransactionType.DEBIT, 0.0);
```

---

## 11. Quick Reference Cheat Sheet

### Stream Creation
```java
list.stream()                           // From Collection
Arrays.stream(array)                    // From Array
Stream.of(1, 2, 3)                     // From Values
Stream.generate(Math::random).limit(n)  // Infinite
IntStream.range(1, 100)                // Range
Files.lines(path)                       // From File
```

### Intermediate Operations
```java
filter(Predicate)       // Filter elements
map(Function)           // Transform 1-to-1
flatMap(Function)       // Flatten & transform
distinct()              // Remove duplicates
sorted()                // Sort naturally
sorted(Comparator)      // Sort with comparator
limit(n)                // Take first n
skip(n)                 // Skip first n
peek(Consumer)          // Debug only
```

### Terminal Operations
```java
collect(Collector)      // Collect to collection
forEach(Consumer)       // Iterate
reduce(BinaryOperator)  // Reduce to single value
count()                 // Count elements
min/max(Comparator)     // Find min/max
anyMatch(Predicate)     // Check if any match
allMatch(Predicate)     // Check if all match
noneMatch(Predicate)    // Check if none match
findFirst()             // Find first element
findAny()               // Find any element
toArray()               // Convert to array
```

### Common Collectors
```java
Collectors.toList()
Collectors.toSet()
Collectors.toMap(keyMapper, valueMapper)
Collectors.joining(delimiter)
Collectors.groupingBy(classifier)
Collectors.partitioningBy(predicate)
Collectors.counting()
Collectors.summingInt/Long/Double()
Collectors.averagingInt/Long/Double()
Collectors.maxBy/minBy(comparator)
Collectors.mapping(mapper, downstream)
```

### Performance Tips
- ✅ Use primitive streams (IntStream, LongStream, DoubleStream)
- ✅ Parallelize only for large datasets (10,000+) with CPU-intensive ops
- ✅ Avoid side effects in stream operations
- ✅ Use method references over lambdas when possible
- ✅ Chain operations in single pipeline
- ❌ Don't reuse streams
- ❌ Don't modify source during stream operations
- ❌ Don't parallelize I/O-bound operations

---

## Conclusion

Java Streams API provides a powerful, functional way to process collections. Key takeaways:

1. **Readability**: Streams make code more declarative and easier to understand
2. **Performance**: Use parallel streams wisely for large, CPU-intensive operations
3. **Immutability**: Streams don't modify sources, promoting functional programming
4. **Flexibility**: Rich set of operations for complex data transformations
5. **Practice**: Master common patterns like groupingBy, partitioning, and flatMap

Keep practicing with real-world scenarios, and streams will become second nature!
