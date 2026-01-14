# Testing Patterns in Java

## Core Principles

1. **Test behavior, not implementation** - Focus on public API
2. **One assertion per test** - Clear failure messages
3. **Arrange-Act-Assert (AAA)** - Clear test structure
4. **Use descriptive test names** - Explain what's being tested
5. **Mock external dependencies** - Fast, isolated tests

---

## JUnit 5 Basics

### Test Structure

```java
import org.junit.jupiter.api.Test;
import static org.junit.jupiter.api.Assertions.*;

class UserServiceTest {

  @Test
  void shouldCreateUserWithValidData() {
    // Arrange
    UserService service = new UserService();
    String name = "John Doe";
    String email = "john@example.com";

    // Act
    User user = service.createUser(name, email);

    // Assert
    assertNotNull(user);
    assertEquals(name, user.getName());
    assertEquals(email, user.getEmail());
  }
}
```

---

## Test Naming

**Be descriptive:**

```java
// ✅ Good - Describes behavior
@Test
void shouldReturnUserWhenIdExists() {}

@Test
void shouldThrowExceptionWhenEmailIsInvalid() {}

@Test
void shouldReturnEmptyListWhenNoUsersFound() {}

// ❌ Bad - Vague
@Test
void testFindUser() {}

@Test
void test1() {}
```

**Given-When-Then pattern:**

```java
@Test
void givenValidId_whenFindById_thenReturnsUser() {
  // Given
  Long userId = 123L;
  User expectedUser = new User("John", "john@example.com");

  // When
  User actualUser = userService.findById(userId);

  // Then
  assertEquals(expectedUser.getName(), actualUser.getName());
}
```

---

## Assertions

### Basic Assertions

```java
import static org.junit.jupiter.api.Assertions.*;

@Test
void demonstrateAssertions() {
  // Equality
  assertEquals(expected, actual);
  assertEquals(expected, actual, "Custom failure message");

  // Boolean
  assertTrue(condition);
  assertFalse(condition);

  // Null checks
  assertNull(object);
  assertNotNull(object);

  // Same instance
  assertSame(expected, actual);
  assertNotSame(expected, actual);

  // Arrays
  assertArrayEquals(expectedArray, actualArray);

  // Exceptions
  assertThrows(IllegalArgumentException.class, () -> {
    service.invalidOperation();
  });

  // Timeout
  assertTimeout(Duration.ofSeconds(1), () -> {
    service.fastOperation();
  });
}
```

### AssertJ (Fluent Assertions)

```java
import static org.assertj.core.api.Assertions.*;

@Test
void demonstrateAssertJ() {
  // More readable
  assertThat(user.getName()).isEqualTo("John");
  assertThat(user.getAge()).isGreaterThan(18);
  assertThat(users).hasSize(3);
  assertThat(users).extracting("name")
      .containsExactly("Alice", "Bob", "Charlie");

  // Chaining
  assertThat(user)
      .isNotNull()
      .extracting("name", "email")
      .containsExactly("John", "john@example.com");
}
```

---

## Test Lifecycle

```java
import org.junit.jupiter.api.*;

class UserServiceTest {

  private static DatabaseConnection dbConnection;
  private UserService userService;

  @BeforeAll
  static void setupDatabase() {
    // Runs once before all tests
    dbConnection = new DatabaseConnection();
  }

  @BeforeEach
  void setup() {
    // Runs before each test
    userService = new UserService(dbConnection);
  }

  @Test
  void test1() {
    // Test code
  }

  @Test
  void test2() {
    // Test code
  }

  @AfterEach
  void cleanup() {
    // Runs after each test
    userService.clearCache();
  }

  @AfterAll
  static void closeDatabase() {
    // Runs once after all tests
    dbConnection.close();
  }
}
```

---

## Parameterized Tests

**Test multiple inputs:**

```java
import org.junit.jupiter.params.ParameterizedTest;
import org.junit.jupiter.params.provider.*;

class EmailValidatorTest {

  @ParameterizedTest
  @ValueSource(strings = {
      "john@example.com",
      "alice@company.co.uk",
      "bob123@test.org"
  })
  void shouldAcceptValidEmails(String email) {
    assertTrue(EmailValidator.isValid(email));
  }

  @ParameterizedTest
  @ValueSource(strings = {
      "invalid",
      "@example.com",
      "user@",
      ""
  })
  void shouldRejectInvalidEmails(String email) {
    assertFalse(EmailValidator.isValid(email));
  }
}
```

**CSV Source:**

```java
@ParameterizedTest
@CsvSource({
    "1, 2, 3",
    "5, 5, 10",
    "100, 200, 300"
})
void shouldAddNumbers(int a, int b, int expected) {
  assertEquals(expected, calculator.add(a, b));
}
```

**Method Source:**

```java
@ParameterizedTest
@MethodSource("userProvider")
void shouldValidateUsers(User user) {
  assertTrue(validator.isValid(user));
}

private static Stream<User> userProvider() {
  return Stream.of(
      new User("John", "john@example.com"),
      new User("Alice", "alice@example.com")
  );
}
```

---

## Mocking with Mockito

### Basic Mocking

```java
import static org.mockito.Mockito.*;

class UserServiceTest {

  private UserRepository userRepository;
  private EmailService emailService;
  private UserService userService;

  @BeforeEach
  void setup() {
    userRepository = mock(UserRepository.class);
    emailService = mock(EmailService.class);
    userService = new UserService(userRepository, emailService);
  }

  @Test
  void shouldSaveUserAndSendEmail() {
    // Arrange
    User user = new User("John", "john@example.com");
    when(userRepository.save(user)).thenReturn(user);

    // Act
    userService.registerUser(user);

    // Assert
    verify(userRepository).save(user);
    verify(emailService).sendWelcomeEmail(user.getEmail());
  }
}
```

### Using @Mock and @InjectMocks

```java
import org.mockito.Mock;
import org.mockito.InjectMocks;
import org.mockito.junit.jupiter.MockitoExtension;
import org.junit.jupiter.api.extension.ExtendWith;

@ExtendWith(MockitoExtension.class)
class UserServiceTest {

  @Mock
  private UserRepository userRepository;

  @Mock
  private EmailService emailService;

  @InjectMocks
  private UserService userService;

  @Test
  void shouldFindUserById() {
    // Arrange
    Long userId = 123L;
    User expectedUser = new User("John", "john@example.com");
    when(userRepository.findById(userId))
        .thenReturn(Optional.of(expectedUser));

    // Act
    Optional<User> result = userService.findById(userId);

    // Assert
    assertTrue(result.isPresent());
    assertEquals(expectedUser.getName(), result.get().getName());
  }
}
```

### Stubbing

```java
@Test
void demonstrateStubbing() {
  // Return value
  when(repository.findById(1L))
      .thenReturn(Optional.of(user));

  // Throw exception
  when(repository.findById(999L))
      .thenThrow(new NotFoundException());

  // Return different values on successive calls
  when(service.getValue())
      .thenReturn(1)
      .thenReturn(2)
      .thenReturn(3);

  // Answer with lambda
  when(service.calculate(anyInt()))
      .thenAnswer(invocation -> {
        int arg = invocation.getArgument(0);
        return arg * 2;
      });
}
```

### Verification

```java
@Test
void demonstrateVerification() {
  // Verify method called
  verify(repository).save(user);

  // Verify with arguments
  verify(emailService).sendEmail(eq("john@example.com"), anyString());

  // Verify number of invocations
  verify(repository, times(3)).findAll();
  verify(repository, never()).delete(any());
  verify(repository, atLeastOnce()).save(any());

  // Verify order
  InOrder inOrder = inOrder(repository, emailService);
  inOrder.verify(repository).save(user);
  inOrder.verify(emailService).sendEmail(anyString(), anyString());

  // Capture arguments
  ArgumentCaptor<User> userCaptor = ArgumentCaptor.forClass(User.class);
  verify(repository).save(userCaptor.capture());
  assertEquals("John", userCaptor.getValue().getName());
}
```

---

## Spring Boot Testing

### Unit Tests

```java
import org.junit.jupiter.api.Test;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.springframework.boot.test.mock.mockito.MockBean;

@SpringBootTest
class UserServiceIntegrationTest {

  @Autowired
  private UserService userService;

  @MockBean
  private UserRepository userRepository;

  @Test
  void shouldCreateUser() {
    User user = new User("John", "john@example.com");
    when(userRepository.save(any(User.class))).thenReturn(user);

    User created = userService.createUser("John", "john@example.com");

    assertNotNull(created);
    verify(userRepository).save(any(User.class));
  }
}
```

### Web Layer Tests

```java
import org.springframework.boot.test.autoconfigure.web.servlet.WebMvcTest;
import org.springframework.test.web.servlet.MockMvc;
import static org.springframework.test.web.servlet.request.MockMvcRequestBuilders.*;
import static org.springframework.test.web.servlet.result.MockMvcResultMatchers.*;

@WebMvcTest(UserController.class)
class UserControllerTest {

  @Autowired
  private MockMvc mockMvc;

  @MockBean
  private UserService userService;

  @Test
  void shouldReturnUserWhenFound() throws Exception {
    User user = new User("John", "john@example.com");
    when(userService.findById(1L)).thenReturn(Optional.of(user));

    mockMvc.perform(get("/api/users/1"))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.name").value("John"))
        .andExpect(jsonPath("$.email").value("john@example.com"));
  }

  @Test
  void shouldReturn404WhenUserNotFound() throws Exception {
    when(userService.findById(999L)).thenReturn(Optional.empty());

    mockMvc.perform(get("/api/users/999"))
        .andExpect(status().isNotFound());
  }

  @Test
  void shouldCreateUser() throws Exception {
    String userJson = """
        {
          "name": "John",
          "email": "john@example.com"
        }
        """;

    mockMvc.perform(post("/api/users")
            .contentType(MediaType.APPLICATION_JSON)
            .content(userJson))
        .andExpect(status().isCreated());
  }
}
```

### Repository Tests

```java
import org.springframework.boot.test.autoconfigure.orm.jpa.DataJpaTest;
import org.springframework.boot.test.autoconfigure.orm.jpa.TestEntityManager;

@DataJpaTest
class UserRepositoryTest {

  @Autowired
  private TestEntityManager entityManager;

  @Autowired
  private UserRepository userRepository;

  @Test
  void shouldFindUserByEmail() {
    // Arrange
    User user = new User("John", "john@example.com");
    entityManager.persist(user);
    entityManager.flush();

    // Act
    Optional<User> found = userRepository.findByEmail("john@example.com");

    // Assert
    assertTrue(found.isPresent());
    assertEquals("John", found.get().getName());
  }
}
```

---

## Test Organization

### Nested Tests

```java
@Nested
@DisplayName("When user is logged in")
class WhenLoggedIn {

  private User user;

  @BeforeEach
  void login() {
    user = authService.login("john@example.com", "password");
  }

  @Test
  void canAccessProfile() {
    assertTrue(user.canAccess("/profile"));
  }

  @Test
  void canUpdateSettings() {
    assertTrue(user.canAccess("/settings"));
  }

  @Nested
  @DisplayName("And user is admin")
  class AndIsAdmin {

    @BeforeEach
    void makeAdmin() {
      user.setRole("ADMIN");
    }

    @Test
    void canAccessAdminPanel() {
      assertTrue(user.canAccess("/admin"));
    }
  }
}
```

---

## Test Doubles

### Stub - Returns canned responses

```java
class StubUserRepository implements UserRepository {
  @Override
  public User findById(Long id) {
    return new User("Stub User", "stub@example.com");
  }
}
```

### Mock - Verifies interactions

```java
UserRepository mock = mock(UserRepository.class);
when(mock.findById(1L)).thenReturn(user);
verify(mock).findById(1L);
```

### Fake - Working implementation

```java
class FakeUserRepository implements UserRepository {
  private Map<Long, User> users = new HashMap<>();

  @Override
  public User save(User user) {
    users.put(user.getId(), user);
    return user;
  }

  @Override
  public Optional<User> findById(Long id) {
    return Optional.ofNullable(users.get(id));
  }
}
```

---

## Test Data Builders

```java
class UserBuilder {
  private String name = "John Doe";
  private String email = "john@example.com";
  private int age = 30;

  public UserBuilder withName(String name) {
    this.name = name;
    return this;
  }

  public UserBuilder withEmail(String email) {
    this.email = email;
    return this;
  }

  public UserBuilder withAge(int age) {
    this.age = age;
    return this;
  }

  public User build() {
    return new User(name, email, age);
  }
}

// Usage in tests
@Test
void testUser() {
  User user = new UserBuilder()
      .withName("Alice")
      .withAge(25)
      .build();

  assertNotNull(user);
}
```

---

## Best Practices

### Do's

```java
// ✅ Test one thing per test
@Test
void shouldCalculateTotal() {
  assertEquals(100, calculator.add(60, 40));
}

// ✅ Use descriptive names
@Test
void shouldThrowExceptionWhenDividingByZero() {
  assertThrows(ArithmeticException.class, () -> calculator.divide(10, 0));
}

// ✅ Arrange-Act-Assert
@Test
void shouldUpdateUser() {
  // Arrange
  User user = new User("John", "john@example.com");

  // Act
  user.setName("Jane");

  // Assert
  assertEquals("Jane", user.getName());
}
```

### Don'ts

```java
// ❌ Don't test multiple things
@Test
void testEverything() {
  // Creating user
  User user = service.create("John", "john@example.com");
  assertNotNull(user);

  // Updating user
  user.setName("Jane");
  assertEquals("Jane", user.getName());

  // Deleting user
  service.delete(user.getId());
  // ... too many responsibilities
}

// ❌ Don't use production data
@Test
void testWithProductionData() {
  User user = productionDatabase.findById(12345L);  // Bad!
}

// ❌ Don't ignore test failures
@Test
@Disabled("This test is flaky")  // Fix it instead!
void flakyTest() {
}
```

---

## Best Practices Summary

1. **AAA pattern**: Arrange-Act-Assert
2. **One assertion** per test (when possible)
3. **Descriptive names**: Explain what's being tested
4. **Mock externals**: Fast, isolated tests
5. **Test behavior**: Not implementation details
6. **Use builders**: For complex test data
7. **Verify interactions**: With Mockito
8. **Parameterized tests**: For multiple inputs
9. **Spring slices**: @WebMvcTest, @DataJpaTest
10. **Keep tests fast**: No network, no database (when possible)

---

## References

- [JUnit 5 User Guide](https://junit.org/junit5/docs/current/user-guide/)
- [Mockito Documentation](https://javadoc.io/doc/org.mockito/mockito-core/latest/org/mockito/Mockito.html)
- [Spring Boot Testing](https://spring.io/guides/gs/testing-web/)
- [AssertJ Documentation](https://assertj.github.io/doc/)
