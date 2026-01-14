# Programming Practices in Java

## Core Principles

1. **Always use @Override** - Catch errors at compile time
2. **Never ignore exceptions** - Handle or rethrow
3. **Qualify static members with class** - Not instance
4. **Avoid finalizers** - Use try-with-resources instead
5. **Prefer composition over inheritance** - Flexibility

---

## Exception Handling

### Never Ignore Exceptions

**❌ Bad - Empty catch block:**

```java
try {
  riskyOperation();
} catch (Exception e) {
  // Silent failure
}
```

**✅ Good - Handle appropriately:**

```java
// Log and rethrow
try {
  riskyOperation();
} catch (IOException e) {
  logger.error("Failed to perform risky operation", e);
  throw new ServiceException("Operation failed", e);
}

// Handle with fallback
try {
  return fetchFromCache();
} catch (CacheException e) {
  logger.warn("Cache miss, fetching from database", e);
  return fetchFromDatabase();
}

// Document why ignoring
try {
  optional Operation();
} catch (Exception ignored) {
  // Optional operation, failure is acceptable
}
```

---

### Catch Specific Exceptions

**❌ Bad - Catch generic Exception:**

```java
try {
  parseJson(data);
} catch (Exception e) {
  // Too broad
}
```

**✅ Good - Catch specific types:**

```java
try {
  parseJson(data);
} catch (JsonParseException e) {
  throw new InvalidRequestException("Invalid JSON format", e);
} catch (IOException e) {
  throw new ServiceException("Failed to read JSON", e);
}
```

---

### Use try-with-resources

**❌ Bad - Manual resource management:**

```java
BufferedReader reader = null;
try {
  reader = new BufferedReader(new FileReader(file));
  return reader.readLine();
} finally {
  if (reader != null) {
    try {
      reader.close();
    } catch (IOException ignored) {
    }
  }
}
```

**✅ Good - Automatic resource management:**

```java
try (BufferedReader reader = new BufferedReader(new FileReader(file))) {
  return reader.readLine();
} catch (IOException e) {
  throw new FileReadException("Failed to read file", e);
}

// Multiple resources
try (InputStream in = new FileInputStream(source);
     OutputStream out = new FileOutputStream(dest)) {
  byte[] buffer = new byte[8192];
  int bytesRead;
  while ((bytesRead = in.read(buffer)) != -1) {
    out.write(buffer, 0, bytesRead);
  }
}
```

---

## @Override Annotation

**Always use @Override:**

```java
public class User {
  private String name;

  @Override
  public String toString() {
    return "User{name='" + name + "'}";
  }

  @Override
  public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    User user = (User) o;
    return Objects.equals(name, user.name);
  }

  @Override
  public int hashCode() {
    return Objects.hash(name);
  }
}

// Interface implementation
public class UserServiceImpl implements UserService {
  @Override
  public User findById(Long id) {
    // Implementation
  }
}
```

**Why:** Compiler catches typos and signature mismatches.

---

## Static Members

**Qualify with class name:**

```java
public class MathUtils {
  public static int add(int a, int b) {
    return a + b;
  }
}

// ❌ Bad - Qualifying with instance
MathUtils utils = new MathUtils();
int sum = utils.add(1, 2);

// ✅ Good - Qualifying with class
int sum = MathUtils.add(1, 2);
```

---

## Immutability

**Prefer immutable objects:**

```java
// ✅ Good - Immutable class
public final class User {
  private final String name;
  private final String email;

  public User(String name, String email) {
    this.name = name;
    this.email = email;
  }

  public String getName() {
    return name;
  }

  public String getEmail() {
    return email;
  }

  // Return new instance instead of modifying
  public User withName(String newName) {
    return new User(newName, this.email);
  }
}
```

**Defensive copying:**

```java
public class UserService {
  private final List<User> users;

  public UserService(List<User> users) {
    // Defensive copy
    this.users = new ArrayList<>(users);
  }

  public List<User> getUsers() {
    // Return unmodifiable view
    return Collections.unmodifiableList(users);
  }
}
```

---

## Nullability

**Use Optional for return values:**

```java
// ✅ Good
public Optional<User> findById(Long id) {
  User user = repository.findById(id);
  return Optional.ofNullable(user);
}

// Usage
findById(123L)
    .ifPresent(user -> System.out.println(user.getName()));

User user = findById(123L)
    .orElseThrow(() -> new UserNotFoundException(id));
```

**Check parameters:**

```java
public class UserService {
  public void save(User user) {
    Objects.requireNonNull(user, "user cannot be null");
    Objects.requireNonNull(user.getName(), "user name cannot be null");

    repository.save(user);
  }
}
```

**Use annotations:**

```java
import javax.annotation.Nullable;
import javax.annotation.Nonnull;

public class UserService {
  public void save(@Nonnull User user) {
    repository.save(user);
  }

  @Nullable
  public User findByEmail(String email) {
    return repository.findByEmail(email);
  }
}
```

---

## Equals and HashCode

**Always override both together:**

```java
public class User {
  private Long id;
  private String email;

  @Override
  public boolean equals(Object o) {
    if (this == o) return true;
    if (o == null || getClass() != o.getClass()) return false;
    User user = (User) o;
    return Objects.equals(id, user.id) &&
           Objects.equals(email, user.email);
  }

  @Override
  public int hashCode() {
    return Objects.hash(id, email);
  }
}
```

**Use Objects utility:**

```java
@Override
public boolean equals(Object o) {
  if (this == o) return true;
  if (!(o instanceof User)) return false;
  User user = (User) o;
  return Objects.equals(id, user.id);
}
```

---

## String Operations

**Use StringBuilder for loops:**

```java
// ❌ Bad - String concatenation in loop
String result = "";
for (String item : items) {
  result += item + ",";
}

// ✅ Good - StringBuilder
StringBuilder sb = new StringBuilder();
for (String item : items) {
  sb.append(item).append(",");
}
String result = sb.toString();

// ✅ Better - String.join
String result = String.join(",", items);
```

**Use formatted strings:**

```java
// ✅ Good
String message = String.format(
    "User %s (ID: %d) logged in at %s",
    user.getName(),
    user.getId(),
    timestamp
);

// Java 15+ text blocks
String json = """
    {
      "name": "%s",
      "email": "%s"
    }
    """.formatted(name, email);
```

---

## Collections

**Use appropriate collection types:**

```java
// Lists - Ordered, allows duplicates
List<String> names = new ArrayList<>();
List<String> immutableNames = List.of("Alice", "Bob");

// Sets - Unique elements
Set<String> uniqueIds = new HashSet<>();
Set<String> immutableIds = Set.of("1", "2", "3");

// Maps - Key-value pairs
Map<Long, User> userMap = new HashMap<>();
Map<String, String> config = Map.of("key1", "value1", "key2", "value2");
```

**Use diamond operator:**

```java
// ✅ Good
List<String> names = new ArrayList<>();
Map<Long, User> users = new HashMap<>();

// ❌ Bad
List<String> names = new ArrayList<String>();
```

**Prefer factory methods (Java 9+):**

```java
// ✅ Good - Immutable collections
List<String> names = List.of("Alice", "Bob", "Charlie");
Set<Integer> numbers = Set.of(1, 2, 3);
Map<String, Integer> map = Map.of("a", 1, "b", 2);
```

---

## Stream API

**Use streams for collection operations:**

```java
// Filter and collect
List<User> activeUsers = users.stream()
    .filter(User::isActive)
    .collect(Collectors.toList());

// Map and reduce
int totalAge = users.stream()
    .mapToInt(User::getAge)
    .sum();

// Group by
Map<String, List<User>> byRole = users.stream()
    .collect(Collectors.groupingBy(User::getRole));

// Find first
Optional<User> admin = users.stream()
    .filter(u -> u.getRole().equals("ADMIN"))
    .findFirst();
```

**Prefer method references:**

```java
// ✅ Good
users.stream()
    .map(User::getName)
    .forEach(System.out::println);

// ❌ Less readable
users.stream()
    .map(u -> u.getName())
    .forEach(n -> System.out.println(n));
```

---

## Enums

**Use enums for fixed sets of constants:**

```java
public enum Status {
  PENDING("Pending", 1),
  ACTIVE("Active", 2),
  COMPLETED("Completed", 3);

  private final String displayName;
  private final int code;

  Status(String displayName, int code) {
    this.displayName = displayName;
    this.code = code;
  }

  public String getDisplayName() {
    return displayName;
  }

  public int getCode() {
    return code;
  }

  public static Status fromCode(int code) {
    for (Status status : values()) {
      if (status.code == code) {
        return status;
      }
    }
    throw new IllegalArgumentException("Invalid code: " + code);
  }
}
```

**Use EnumSet and EnumMap:**

```java
Set<Status> activeStatuses = EnumSet.of(Status.PENDING, Status.ACTIVE);
Map<Status, List<User>> usersByStatus = new EnumMap<>(Status.class);
```

---

## Generics

**Use bounded type parameters:**

```java
// Extend for upper bound
public <T extends Comparable<T>> T max(List<T> list) {
  return list.stream()
      .max(Comparable::compareTo)
      .orElseThrow();
}

// Wildcards for flexibility
public void processList(List<? extends Number> numbers) {
  // Can read as Number
}

public void addToList(List<? super Integer> list) {
  list.add(42);  // Can write Integer
}
```

---

## Dependency Injection

**Use constructor injection:**

```java
@Service
public class UserService {
  private final UserRepository userRepository;
  private final EmailService emailService;

  // ✅ Good - Constructor injection
  @Autowired
  public UserService(
      UserRepository userRepository,
      EmailService emailService) {
    this.userRepository = userRepository;
    this.emailService = emailService;
  }

  // ❌ Bad - Field injection
  // @Autowired
  // private UserRepository userRepository;
}
```

---

## Common Pitfalls

### Don't Use == for Strings

```java
// ❌ Bad
if (name == "John") {
}

// ✅ Good
if ("John".equals(name)) {  // Null-safe
}

if (Objects.equals(name, "John")) {  // Also null-safe
}
```

### Don't Use float/double for Money

```java
// ❌ Bad
double price = 19.99;
double total = price * 3;  // Precision issues

// ✅ Good
BigDecimal price = new BigDecimal("19.99");
BigDecimal total = price.multiply(BigDecimal.valueOf(3));
```

### Close Resources

```java
// ❌ Bad
Connection conn = dataSource.getConnection();
// If exception occurs, conn never closed

// ✅ Good
try (Connection conn = dataSource.getConnection()) {
  // Use connection
}  // Automatically closed
```

---

## Best Practices Summary

1. **@Override**: Always use for overridden methods
2. **Exceptions**: Never ignore, catch specific types
3. **Resources**: Use try-with-resources
4. **Static members**: Qualify with class name
5. **Immutability**: Prefer final fields and classes
6. **Nullability**: Use Optional, check parameters
7. **equals/hashCode**: Override both together
8. **Collections**: Use factory methods, diamond operator
9. **Streams**: For collection operations
10. **Enums**: For fixed sets of constants
11. **Generics**: Use bounded types appropriately
12. **DI**: Constructor injection over field injection

---

## References

- [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)
- [Effective Java (Joshua Bloch)](https://www.oreilly.com/library/view/effective-java/9780134686097/)
