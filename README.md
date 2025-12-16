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
