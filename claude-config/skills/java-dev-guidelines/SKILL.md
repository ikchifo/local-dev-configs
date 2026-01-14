---
name: java-dev-guidelines
description: Use when writing Java code, Spring Boot applications, microservices, REST APIs, JPA/Hibernate, or testing with JUnit/Mockito. Covers Google Java Style Guide and enterprise patterns.
---

# Java Development Guidelines

**Purpose:** Comprehensive best practices and patterns for Java development based on Google's Java Style Guide and industry standards.

**When to use:**
- Writing Java code (Spring Boot, microservices, enterprise applications)
- Setting up Java projects
- Building REST APIs with Spring Boot
- Code reviews and refactoring
- Testing Java applications

---

## Quick Start

This skill follows the **500-line modular pattern** with progressive disclosure:
- **Main file (this)**: High-level overview and quick reference
- **Resource files**: Deep dives into specific topics

### Core Principles

1. **Readability first** - Code is read more than written
2. **Consistency** - Follow conventions throughout
3. **Explicit over implicit** - Clear intent
4. **Immutability when possible** - Thread-safe, predictable
5. **Test everything** - Automated tests for reliability

---

## Project Structure

**Standard Maven/Gradle Spring Boot layout:**

```
my-application/
├── src/
│   ├── main/
│   │   ├── java/
│   │   │   └── com/example/myapp/
│   │   │       ├── MyApplication.java
│   │   │       ├── controller/
│   │   │       ├── service/
│   │   │       ├── repository/
│   │   │       ├── model/
│   │   │       ├── dto/
│   │   │       ├── config/
│   │   │       └── exception/
│   │   └── resources/
│   │       ├── application.yml
│   │       └── db/migration/
│   └── test/
│       └── java/
├── pom.xml (or build.gradle)
└── README.md
```

---

## Quick Examples

### Class Structure

```java
public class User {
  // Constants
  public static final String DEFAULT_ROLE = "USER";

  // Fields
  private Long id;
  private String name;
  private String email;

  // Constructors
  public User() {}

  public User(String name, String email) {
    this.name = name;
    this.email = email;
  }

  // Methods (public first, then private)
  public String getName() {
    return name;
  }

  @Override
  public String toString() {
    return "User{name='" + name + "'}";
  }
}
```

### REST Controller

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

  private final UserService userService;

  @Autowired
  public UserController(UserService userService) {
    this.userService = userService;
  }

  @GetMapping("/{id}")
  public ResponseEntity<UserDTO> getUser(@PathVariable Long id) {
    return userService.findById(id)
        .map(ResponseEntity::ok)
        .orElse(ResponseEntity.notFound().build());
  }

  @PostMapping
  public ResponseEntity<UserDTO> createUser(@Valid @RequestBody CreateUserRequest request) {
    UserDTO created = userService.create(request);
    return ResponseEntity.created(URI.create("/api/users/" + created.getId())).body(created);
  }
}
```

### Service Layer

```java
@Service
public class UserService {

  private final UserRepository userRepository;

  @Autowired
  public UserService(UserRepository userRepository) {
    this.userRepository = userRepository;
  }

  @Transactional(readOnly = true)
  public Optional<UserDTO> findById(Long id) {
    return userRepository.findById(id)
        .map(this::toDTO);
  }

  @Transactional
  public UserDTO create(CreateUserRequest request) {
    User user = new User(request.getName(), request.getEmail());
    User saved = userRepository.save(user);
    return toDTO(saved);
  }

  private UserDTO toDTO(User user) {
    return new UserDTO(user.getId(), user.getName(), user.getEmail());
  }
}
```

---

## Resource Files (Deep Dives)

Use the `Read` tool to access detailed guides on specific topics:

### 1. Naming and Style
**File:** `resources/naming-and-style.md`

**Covers:**
- Package naming (lowercase, no underscores)
- Class naming (UpperCamelCase)
- Method naming (lowerCamelCase, verbs)
- Variable naming (descriptive, lowerCamelCase)
- Constants (UPPER_SNAKE_CASE)
- Formatting rules (braces, indentation, line length)
- Import organization
- Member ordering

**Use when:** Naming things, formatting code, code reviews

---

### 2. Programming Practices
**File:** `resources/programming-practices.md`

**Covers:**
- Exception handling (never ignore, try-with-resources)
- @Override annotation (always use)
- Static members (qualify with class)
- Immutability (prefer final)
- Nullability (Optional, null checks)
- equals() and hashCode()
- String operations
- Collections usage
- Stream API
- Enums and generics

**Use when:** Writing Java code, handling errors, working with collections

---

### 3. Testing Patterns
**File:** `resources/testing-patterns.md`

**Covers:**
- JUnit 5 basics
- Test naming conventions
- Assertions (JUnit and AssertJ)
- Parameterized tests
- Mocking with Mockito
- Spring Boot testing (@WebMvcTest, @DataJpaTest)
- Test doubles (stubs, mocks, fakes)
- Test data builders

**Use when:** Writing tests, setting up test infrastructure

---

### 4. Documentation
**File:** `resources/documentation.md`

**Covers:**
- Javadoc basics
- Class and method documentation
- Standard block tags (@param, @return, @throws)
- Summary fragments
- Code examples in Javadoc
- Package documentation
- Implementation comments (when and how)

**Use when:** Documenting code, writing API docs

---

### 5. Common Patterns
**File:** `resources/common-patterns.md`

**Covers:**
- Builder pattern
- Factory pattern
- Singleton pattern
- Strategy pattern
- Observer pattern
- Dependency injection
- Template method
- Adapter pattern
- Repository pattern
- Fluent interfaces
- Null object pattern
- Command pattern

**Use when:** Designing classes, implementing design patterns

---

### 6. Spring Boot Patterns
**File:** `resources/spring-boot-patterns.md`

**Covers:**
- Spring Boot project structure
- Controller layer (REST APIs)
- Service layer (business logic)
- Repository layer (Spring Data JPA)
- Entity classes and DTOs
- Configuration (application.yml, profiles)
- Exception handling (@RestControllerAdvice)
- Validation
- Async processing
- Caching
- Security
- Testing Spring Boot apps

**Use when:** Building Spring Boot applications, REST APIs

---

## Common Commands

```bash
# Maven
mvn clean install          # Build project
mvn test                   # Run tests
mvn spring-boot:run        # Run Spring Boot app
mvn package                # Package as JAR/WAR

# Gradle
./gradlew build           # Build project
./gradlew test            # Run tests
./gradlew bootRun         # Run Spring Boot app
./gradlew clean build     # Clean and build
```

---

## When to Use What

| Situation | Resource to Read |
|-----------|------------------|
| Naming classes/methods | naming-and-style.md |
| Handling exceptions | programming-practices.md |
| Writing JUnit tests | testing-patterns.md |
| Documenting public API | documentation.md |
| Implementing builder | common-patterns.md |
| Creating REST API | spring-boot-patterns.md |
| Using Spring Data JPA | spring-boot-patterns.md |
| Working with streams | programming-practices.md |
| Organizing imports | naming-and-style.md |
| Exception handling in Spring | spring-boot-patterns.md |

---

## Quick Decision Trees

### Naming
```
Naming what?
├─ Package? → lowercase, no underscores (com.example.myapp)
├─ Class? → UpperCamelCase, noun (UserService)
├─ Interface? → UpperCamelCase, no 'I' prefix (PaymentProcessor)
├─ Method? → lowerCamelCase, verb (findById)
├─ Variable? → lowerCamelCase, descriptive (userRepository)
└─ Constant? → UPPER_SNAKE_CASE (MAX_RETRY_COUNT)
```

### Exception Handling
```
Exception occurred?
├─ Can I handle it? → Catch specific exception, handle gracefully
├─ Should caller handle? → Declare in throws, let it propagate
├─ Need cleanup? → Use try-with-resources
└─ Unexpected error? → Log and rethrow as appropriate type
```

### Testing
```
What to test?
├─ Controller? → @WebMvcTest, MockMvc
├─ Service? → @SpringBootTest, mock repositories
├─ Repository? → @DataJpaTest, TestEntityManager
├─ Multiple inputs? → @ParameterizedTest
└─ Unit test? → JUnit + Mockito
```

---

## Anti-Patterns to Avoid

**❌ Don't:**
- Ignore caught exceptions
- Use == for String comparison
- Forget @Override annotation
- Create mutable static fields
- Use finalizers
- Wildcard imports (import java.util.*)
- Prefix interfaces with 'I'
- Suffix implementations with 'Impl'
- Use Hungarian notation
- Field injection (use constructor injection)
- Catch generic Exception

**✅ Do:**
- Handle or rethrow exceptions
- Use .equals() for String comparison
- Always use @Override
- Make fields final when possible
- Use try-with-resources
- Specific imports
- Name interfaces naturally
- Hide implementation details
- Use descriptive names
- Constructor injection
- Catch specific exception types

---

## Integration with Development Workflow

### Before Writing Code
1. **Read:** `naming-and-style.md` - Understand conventions
2. **Read:** `spring-boot-patterns.md` - If using Spring Boot

### While Writing Code
1. **Follow:** Naming conventions from `naming-and-style.md`
2. **Follow:** Exception handling from `programming-practices.md`
3. **Use:** Patterns from `common-patterns.md` when appropriate

### After Writing Code
1. **Write tests** using patterns from `testing-patterns.md`
2. **Document** public API using `documentation.md`
3. **Review:** Against anti-patterns listed above

### For Spring Boot Projects
1. **Read:** `spring-boot-patterns.md` for architecture
2. **Follow:** Layered approach (Controller → Service → Repository)
3. **Use:** Spring Boot starters and auto-configuration

---

## Progressive Learning Path

**Beginner:**
1. Start with naming-and-style.md
2. Learn programming-practices.md
3. Practice testing-patterns.md

**Intermediate:**
4. Study common-patterns.md
5. Learn documentation.md
6. Build projects with Spring Boot

**Advanced:**
7. Master spring-boot-patterns.md
8. Implement design patterns
9. Contribute to open source

---

## Code Quality Checklist

**Before committing:**
- [ ] Code follows naming conventions
- [ ] All public methods documented
- [ ] Tests written and passing
- [ ] No compiler warnings
- [ ] Exceptions handled properly
- [ ] @Override annotations present
- [ ] equals() and hashCode() implemented together
- [ ] try-with-resources for closeable resources
- [ ] No wildcard imports
- [ ] Constructor injection used

---

## References

**Style Guides:**
- [Google Java Style Guide](https://google.github.io/styleguide/javaguide.html)
- [Oracle Code Conventions](https://www.oracle.com/java/technologies/javase/codeconventions-contents.html)

**Books:**
- [Effective Java (Joshua Bloch)](https://www.oreilly.com/library/view/effective-java/9780134686097/)
- [Clean Code (Robert C. Martin)](https://www.oreilly.com/library/view/clean-code-a/9780136083238/)

**Spring:**
- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Spring Data JPA](https://spring.io/projects/spring-data-jpa)
- [Baeldung Tutorials](https://www.baeldung.com/)

**Testing:**
- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Mockito Documentation](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)

---

## How to Use This Skill

1. **Start here** for quick reference and overview
2. **Use Read tool** to access specific resource files
3. **Follow examples** for your use case
4. **Reference decision trees** for quick guidance
5. **Check anti-patterns** during code review

**Example workflow:**
```
User: "How should I handle validation in Spring Boot?"
Assistant: Let me read spring-boot-patterns.md for validation patterns...
[Shows @Valid annotation usage and global exception handling]
```

---

**Remember:** Java emphasizes readability, type safety, and explicit code. When in doubt, choose the clearer, more explicit approach.
