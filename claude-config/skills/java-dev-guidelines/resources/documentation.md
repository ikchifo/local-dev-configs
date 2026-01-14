# Documentation and Javadoc in Java

## Core Principles

1. **Document public API** - All public classes and methods
2. **Summary fragment first** - Complete sentence in first line
3. **Be concise** - Avoid obvious statements
4. **Standard tag order** - @param, @return, @throws
5. **Code examples** - Show usage in complex cases

---

## Javadoc Basics

### Class Documentation

```java
/**
 * Represents a user in the system.
 *
 * <p>This class encapsulates user information including name, email,
 * and authentication credentials. Users can have different roles
 * which determine their access permissions.
 *
 * <p>Example usage:
 * <pre>{@code
 * User user = new User("John Doe", "john@example.com");
 * user.setRole(Role.ADMIN);
 * userService.save(user);
 * }</pre>
 *
 * @author Jane Smith
 * @since 1.0
 */
public class User {
  // Class implementation
}
```

---

### Method Documentation

```java
/**
 * Finds a user by their unique identifier.
 *
 * <p>This method queries the database for a user with the given ID.
 * If no user is found, an empty Optional is returned.
 *
 * @param id the user ID to search for, must not be null
 * @return an Optional containing the user if found, or empty otherwise
 * @throws IllegalArgumentException if id is null
 * @throws DatabaseException if database access fails
 */
public Optional<User> findById(Long id) {
  Objects.requireNonNull(id, "id must not be null");
  return repository.findById(id);
}
```

---

### Field Documentation

```java
/**
 * The maximum number of login attempts before account lockout.
 *
 * <p>Default value is 3. This can be configured via the
 * {@code security.max-login-attempts} property.
 */
private static final int MAX_LOGIN_ATTEMPTS = 3;

/**
 * Cache of active user sessions.
 *
 * <p>Keys are user IDs, values are session tokens.
 * Entries expire after 30 minutes of inactivity.
 */
private final Map<Long, String> sessionCache;
```

---

## Summary Fragment

**First sentence is special:**

```java
/**
 * Calculates the total price including tax.
 *
 * The tax rate is determined by the user's location
 * and current tax regulations.
 *
 * @param basePrice the price before tax
 * @param location the user's location
 * @return the total price with tax included
 */
public BigDecimal calculateTotal(BigDecimal basePrice, Location location) {
  // Implementation
}
```

**The first sentence** ("Calculates the total price including tax.") appears in:
- Method summaries
- Class overviews
- IDE tooltips

---

## Standard Block Tags

### @param

```java
/**
 * Creates a new user account.
 *
 * @param name the user's full name, must not be empty
 * @param email the user's email address, must be valid format
 * @param password the user's password, minimum 8 characters
 * @return the created user with generated ID
 */
public User createUser(String name, String email, String password) {
  // Implementation
}
```

---

### @return

```java
/**
 * Searches for users matching the given criteria.
 *
 * @param criteria the search criteria
 * @return a list of matching users, never null but may be empty
 */
public List<User> search(SearchCriteria criteria) {
  // Implementation
}

/**
 * Attempts to authenticate the user.
 *
 * @param credentials the login credentials
 * @return {@code true} if authentication succeeds, {@code false} otherwise
 */
public boolean authenticate(Credentials credentials) {
  // Implementation
}
```

---

### @throws

```java
/**
 * Deletes a user from the system.
 *
 * @param userId the ID of the user to delete
 * @throws UserNotFoundException if no user exists with the given ID
 * @throws IllegalStateException if the user has active sessions
 * @throws DatabaseException if database operation fails
 */
public void deleteUser(Long userId) {
  // Implementation
}
```

---

### @deprecated

```java
/**
 * Calculates user score using legacy algorithm.
 *
 * @param user the user to score
 * @return the calculated score
 * @deprecated Use {@link #calculateScore(User, ScoreConfig)} instead.
 *             This method will be removed in version 3.0.
 */
@Deprecated
public int calculateScore(User user) {
  // Implementation
}

/**
 * Calculates user score using configurable algorithm.
 *
 * @param user the user to score
 * @param config the scoring configuration
 * @return the calculated score
 * @since 2.0
 */
public int calculateScore(User user, ScoreConfig config) {
  // Implementation
}
```

---

### @see and @link

```java
/**
 * Processes a payment transaction.
 *
 * <p>This method handles payment processing using the configured
 * payment gateway. For refunds, see {@link #processRefund(Long)}.
 *
 * @param transaction the payment transaction to process
 * @return the processed transaction with updated status
 * @see #processRefund(Long)
 * @see PaymentGateway#charge(BigDecimal)
 */
public Transaction processPayment(Transaction transaction) {
  // Implementation
}
```

---

### @code and @literal

```java
/**
 * Parses a configuration string.
 *
 * <p>Expected format: {@code key=value;key2=value2}
 *
 * <p>Example:
 * <pre>{@code
 * String config = "timeout=30;retries=3";
 * Map<String, String> parsed = parseConfig(config);
 * }</pre>
 *
 * @param configString the configuration string to parse
 * @return a map of configuration key-value pairs
 */
public Map<String, String> parseConfig(String configString) {
  // Implementation
}
```

---

## Package Documentation

**package-info.java:**

```java
/**
 * Provides classes for user management and authentication.
 *
 * <p>This package contains the core domain models and services
 * for handling user accounts, authentication, and authorization.
 *
 * <p>Key classes:
 * <ul>
 *   <li>{@link com.example.user.User} - User domain model
 *   <li>{@link com.example.user.UserService} - User management service
 *   <li>{@link com.example.user.AuthService} - Authentication service
 * </ul>
 *
 * @since 1.0
 */
package com.example.user;
```

---

## Implementation Comments

**Use when code is complex:**

```java
public void processOrder(Order order) {
  // Validate order items are in stock
  for (OrderItem item : order.getItems()) {
    if (!inventory.isInStock(item.getProductId(), item.getQuantity())) {
      throw new OutOfStockException(item.getProductId());
    }
  }

  // Calculate total with applicable discounts
  BigDecimal total = order.getItems().stream()
      .map(this::calculateItemPrice)
      .reduce(BigDecimal.ZERO, BigDecimal::add);

  // Apply coupon if present
  if (order.hasCoupon()) {
    total = applyCoupon(total, order.getCoupon());
  }

  order.setTotal(total);
}
```

---

## TODO Comments

**Mark incomplete work:**

```java
public class FeatureService {

  // TODO(john): Implement caching to improve performance (issue #123)
  public List<Feature> findAll() {
    return repository.findAll();
  }

  // TODO: Add validation for feature flags before 2.0 release
  public void enableFeature(String featureName) {
    features.put(featureName, true);
  }
}
```

---

## What NOT to Document

**Avoid obvious comments:**

```java
// ❌ Bad - States the obvious
/**
 * Gets the name.
 *
 * @return the name
 */
public String getName() {
  return name;
}

/**
 * Sets the name.
 *
 * @param name the name to set
 */
public void setName(String name) {
  this.name = name;
}

// ✅ Good - Only document if there's something non-obvious
/**
 * Returns the user's display name.
 *
 * <p>This is formatted as "FirstName LastName" and cached
 * for performance.
 *
 * @return the formatted display name
 */
public String getDisplayName() {
  return displayNameCache.computeIfAbsent(id, this::formatName);
}
```

---

## Code Examples in Javadoc

**Show usage for complex APIs:**

```java
/**
 * Builds a query for searching users.
 *
 * <p>This builder provides a fluent API for constructing complex
 * user queries with multiple criteria.
 *
 * <p>Example usage:
 * <pre>{@code
 * List<User> users = UserQuery.builder()
 *     .withRole("ADMIN")
 *     .withStatus(Status.ACTIVE)
 *     .createdAfter(LocalDate.of(2023, 1, 1))
 *     .orderByName()
 *     .build()
 *     .execute();
 * }</pre>
 *
 * @since 2.0
 */
public class UserQuery {
  // Implementation
}
```

---

## Annotation Documentation

```java
/**
 * Marks a method as requiring authentication.
 *
 * <p>Methods annotated with {@code @RequiresAuth} will be
 * intercepted by the security framework. If the current user
 * is not authenticated, an {@link UnauthorizedException} is thrown.
 *
 * <p>Example:
 * <pre>{@code
 * @RequiresAuth
 * public void deleteAccount(Long userId) {
 *   // Only authenticated users can delete accounts
 * }
 * }</pre>
 *
 * @see AuthenticationInterceptor
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface RequiresAuth {
}
```

---

## Inline Comments

**Explain the why, not the what:**

```java
// ❌ Bad - Explains what (obvious)
// Increment counter
counter++;

// Loop through users
for (User user : users) {
  // Send email
  emailService.send(user.getEmail());
}

// ✅ Good - Explains why
// Skip email validation for internal test accounts
if (user.isTestAccount()) {
  return;
}

// Retry with exponential backoff to handle transient failures
for (int attempt = 0; attempt < MAX_RETRIES; attempt++) {
  try {
    return callExternalApi();
  } catch (TransientException e) {
    Thread.sleep(Math.pow(2, attempt) * 1000);
  }
}
```

---

## Documentation for Inherited Methods

```java
public interface UserRepository {
  /**
   * Finds a user by ID.
   *
   * @param id the user ID
   * @return optional containing user if found
   */
  Optional<User> findById(Long id);
}

public class DatabaseUserRepository implements UserRepository {
  /**
   * {@inheritDoc}
   *
   * <p>This implementation queries the PostgreSQL database
   * using a prepared statement for performance.
   */
  @Override
  public Optional<User> findById(Long id) {
    // Implementation
  }
}
```

---

## Best Practices

### Do's

```java
// ✅ Complete sentence in summary
/**
 * Validates user input for security threats.
 */

// ✅ Document edge cases
/**
 * @param amount the amount to charge, must be positive
 * @throws IllegalArgumentException if amount is zero or negative
 */

// ✅ Show examples for complex APIs
/**
 * <pre>{@code
 * Builder builder = new Builder()
 *     .withName("John")
 *     .build();
 * }</pre>
 */

// ✅ Use @code for inline references
/**
 * Call {@code initialize()} before use.
 */
```

### Don'ts

```java
// ❌ Don't document getters/setters unless non-obvious
/**
 * Gets the ID.
 * @return the ID
 */
public Long getId() { return id; }

// ❌ Don't repeat method name
/**
 * Find user by ID
 */
public User findUserById(Long id) {}

// ❌ Don't use @author for individual methods
/**
 * @author John  // Only use on class level
 */
public void method() {}

// ❌ Don't leave empty Javadoc
/**
 *
 */
public void method() {}  // Either document properly or remove
```

---

## Javadoc Tag Order

**Standard order:**

```java
/**
 * Method description.
 *
 * @param paramName description
 * @return description
 * @throws ExceptionType description
 * @see RelatedClass
 * @since 1.0
 * @deprecated description
 */
```

---

## Best Practices Summary

1. **Public API**: Document all public classes and methods
2. **Summary first**: Complete sentence in first line
3. **Be concise**: Avoid obvious statements
4. **Tag order**: @param, @return, @throws, @see, @since
5. **Code examples**: Use {@code} and <pre>{@code}</pre>
6. **Explain why**: Not just what the code does
7. **Document edge cases**: Null handling, exceptions
8. **Update docs**: When code changes
9. **Use {@link}**: For cross-references
10. **Skip obvious**: Don't document simple getters/setters

---

## References

- [How to Write Doc Comments for Javadoc](https://www.oracle.com/technical-resources/articles/java/javadoc-tool.html)
- [Google Java Style Guide - Javadoc](https://google.github.io/styleguide/javaguide.html#s7-javadoc)
