# Naming and Style Conventions in Java

## Core Principles

1. **Readability first** - Code is read more than written
2. **Consistency** - Follow conventions throughout codebase
3. **CamelCase everywhere** - No underscores except constants
4. **Self-documenting names** - Names should reveal intent
5. **100-character line limit** - Wrap longer lines appropriately

---

## Package Names

**All lowercase, no underscores:**

```java
// ✅ Good
package com.example.myproject;
package com.google.common.base;
package org.springframework.boot;

// ❌ Bad
package com.example.MyProject;    // No uppercase
package com.example.my_project;   // No underscores
package com.Example;               // Too short, not descriptive
```

**Use reverse domain name:**

```java
// Company domain: example.com
package com.example.projectname;
package com.example.projectname.feature;
package com.example.projectname.feature.subfeature;
```

---

## Class Names

**UpperCamelCase:**

```java
// ✅ Good - Nouns or noun phrases
public class User {}
public class UserService {}
public class CustomerRepository {}
public class PaymentProcessor {}
public class HTTPClient {}           // Acronyms stay uppercase
public class XMLParser {}

// ❌ Bad
public class user {}                 // Not capitalized
public class User_Service {}         // No underscores
public class UserSvc {}              // Don't abbreviate
public class IUser {}                // No prefix 'I'
public class UserImpl {}             // Avoid 'Impl' suffix
```

**Test classes:**

```java
// ✅ Good - Test suffix
public class UserServiceTest {}
public class CustomerRepositoryTest {}
```

---

## Interface Names

**UpperCamelCase - Nouns or adjectives:**

```java
// ✅ Good
public interface Runnable {}
public interface Comparable<T> {}
public interface UserRepository {}
public interface PaymentGateway {}

// ❌ Bad
public interface IUserRepository {}  // No 'I' prefix
public interface UserRepositoryInterface {}  // No 'Interface' suffix
```

---

## Method Names

**lowerCamelCase - Verbs or verb phrases:**

```java
// ✅ Good
public void save() {}
public User findById(Long id) {}
public boolean isValid() {}
public boolean hasPermission() {}
public void processPayment() {}
public List<User> getActiveUsers() {}

// ❌ Bad
public void Save() {}                // Not capitalized
public void find_by_id() {}          // No underscores
public void get() {}                 // Too vague
public boolean valid() {}            // Should be isValid()
public void do_processing() {}       // No underscores
```

**Naming patterns:**

```java
// Getters/Setters
public String getName() {}
public void setName(String name) {}

// Boolean getters
public boolean isActive() {}
public boolean hasChildren() {}
public boolean canDelete() {}

// Query methods
public User findById(Long id) {}
public List<User> findByEmail(String email) {}
public Optional<User> findFirstByName(String name) {}

// Action methods
public void create(User user) {}
public void update(User user) {}
public void delete(Long id) {}
public void process() {}
```

---

## Variable Names

**lowerCamelCase:**

```java
// ✅ Good
private String userName;
private int pageCount;
private List<Customer> activeCustomers;
private UserRepository userRepository;

// ❌ Bad
private String user_name;            // No underscores
private String strUserName;          // No type prefixes
private String mUserName;            // No member prefixes
private int n;                       // Too short
```

**Short names in small scopes:**

```java
// ✅ Good - Small scope
for (int i = 0; i < users.size(); i++) {
    User u = users.get(i);
    process(u);
}

// ✅ Good - Larger scope, descriptive
private UserRepository userRepository;
private PaymentProcessor paymentProcessor;
```

**Common abbreviations:**

```java
// ✅ Acceptable
int i, j, k;        // Loop indices
Exception e;        // Exception in catch
StringBuilder sb;   // StringBuilder
HttpServletRequest req;
HttpServletResponse resp;
```

---

## Constants

**UPPER_SNAKE_CASE:**

```java
// ✅ Good
public static final int MAX_RETRY_COUNT = 3;
public static final String DEFAULT_ENCODING = "UTF-8";
public static final long TIMEOUT_MILLIS = 5000L;

// ❌ Bad
public static final int maxRetryCount = 3;     // Not uppercase
public static final int MAX_RETRY = 3;         // Be specific
```

**Enum constants:**

```java
public enum Status {
    PENDING,
    ACTIVE,
    COMPLETED,
    FAILED
}
```

---

## Formatting Rules

### Braces

**Always use braces, K&R style:**

```java
// ✅ Good
if (condition) {
    doSomething();
}

if (condition) {
    doSomething();
} else {
    doSomethingElse();
}

// ❌ Bad
if (condition)
    doSomething();  // No braces

if (condition) doSomething();  // Not on separate line
```

**Empty blocks can be concise:**

```java
// ✅ Good
public void doNothing() {}

try {
    riskyOperation();
} catch (Exception ignored) {
    // Intentionally empty
}

// ❌ Bad - Should have comment
catch (Exception e) {}
```

---

### Indentation

**2 spaces (not tabs):**

```java
// ✅ Good
public class User {
  private String name;

  public User(String name) {
    this.name = name;
  }

  public void print() {
    if (name != null) {
      System.out.println(name);
    }
  }
}
```

---

### Line Length

**100 characters maximum:**

```java
// ✅ Good - Break long lines
public User findUserByEmailAndStatus(
    String email,
    Status status) {
  return userRepository
      .findByEmailAndStatus(email, status)
      .orElseThrow(() -> new UserNotFoundException(email));
}

// Method chaining - indent continuation
String result = someObject
    .method1()
    .method2()
    .method3();
```

---

### Whitespace

**Vertical whitespace:**

```java
// ✅ Good - Blank lines between members
public class User {
  private Long id;
  private String name;

  public User(String name) {
    this.name = name;
  }

  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }
}
```

**Horizontal whitespace:**

```java
// ✅ Good - Spaces around operators
int sum = a + b;
boolean isValid = x > 0 && y < 100;
String name = firstName + " " + lastName;

// After keywords
if (condition) {
for (int i = 0; i < 10; i++) {
while (running) {

// After commas
method(arg1, arg2, arg3);
List<String> list = Arrays.asList("a", "b", "c");
```

---

### Import Statements

**Order and grouping:**

```java
// ✅ Good - Grouped and ordered
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;

import javax.servlet.http.HttpServletRequest;

import com.google.common.collect.ImmutableList;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Service;

import com.example.myproject.model.User;
import com.example.myproject.repository.UserRepository;
```

**No wildcards:**

```java
// ✅ Good
import java.util.List;
import java.util.ArrayList;

// ❌ Bad
import java.util.*;
```

---

### Class Member Ordering

**Logical order:**

```java
public class User {
  // 1. Static constants
  public static final String DEFAULT_ROLE = "USER";

  // 2. Static fields
  private static int instanceCount = 0;

  // 3. Instance fields
  private Long id;
  private String name;
  private String email;

  // 4. Constructors
  public User() {
    instanceCount++;
  }

  public User(String name, String email) {
    this();
    this.name = name;
    this.email = email;
  }

  // 5. Methods (public first, then private)
  public String getName() {
    return name;
  }

  public void setName(String name) {
    this.name = name;
  }

  private void validate() {
    // Validation logic
  }

  // 6. Inner classes
  private static class Builder {
    // Builder pattern
  }
}
```

---

### Annotations

**Place on separate line:**

```java
// ✅ Good
@Override
public String toString() {
  return name;
}

@Service
@Transactional
public class UserService {
}

// Parameter annotations can be inline
public void save(@NotNull User user) {
}
```

---

## Special Cases

### Acronyms in Names

**Treat as words in CamelCase:**

```java
// ✅ Good
public class XmlParser {}
public class HttpClient {}
public void sendHttpRequest() {}
private String userId;

// Exceptions - keep common acronyms uppercase
public class XMLHttpRequest {}  // Historical/well-known
public class URLBuilder {}
```

---

### Prefixes and Suffixes

**Avoid Hungarian notation:**

```java
// ❌ Bad
private String strName;
private int intCount;
private boolean bIsValid;
private List<User> lstUsers;

// ✅ Good
private String name;
private int count;
private boolean isValid;
private List<User> users;
```

**No member prefixes:**

```java
// ❌ Bad
private String mName;
private int m_count;
private String _email;

// ✅ Good
private String name;
private int count;
private String email;
```

---

## Best Practices Summary

1. **Package names**: all lowercase, no underscores
2. **Classes**: UpperCamelCase, nouns
3. **Interfaces**: UpperCamelCase, no 'I' prefix
4. **Methods**: lowerCamelCase, verbs
5. **Variables**: lowerCamelCase, descriptive
6. **Constants**: UPPER_SNAKE_CASE
7. **Braces**: Always use, K&R style
8. **Indentation**: 2 spaces
9. **Line length**: 100 characters
10. **Imports**: No wildcards, grouped logically
11. **Member order**: Constants → fields → constructors → methods
12. **Whitespace**: Consistent vertical and horizontal spacing

---

## References

- [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)
- [Oracle Code Conventions](https://www.oracle.com/java/technologies/javase/codeconventions-contents.html)
