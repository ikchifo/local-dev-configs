# Spring Boot Patterns

## Core Principles

1. **Convention over configuration** - Use Spring Boot defaults
2. **Dependency injection** - Constructor injection preferred
3. **Layered architecture** - Controller → Service → Repository
4. **Configuration via properties** - Externalize configuration
5. **Auto-configuration** - Leverage Spring Boot starters

---

## Project Structure

**Standard Spring Boot layout:**

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
│   │       ├── application-dev.yml
│   │       ├── application-prod.yml
│   │       ├── static/
│   │       └── templates/
│   └── test/
│       └── java/
├── pom.xml (or build.gradle)
└── README.md
```

---

## Main Application Class

```java
package com.example.myapp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class MyApplication {
  public static void main(String[] args) {
    SpringApplication.run(MyApplication.class, args);
  }
}
```

---

## Controller Layer

**REST Controller:**

```java
@RestController
@RequestMapping("/api/users")
public class UserController {

  private final UserService userService;

  @Autowired
  public UserController(UserService userService) {
    this.userService = userService;
  }

  @GetMapping
  public ResponseEntity<List<UserDTO>> getAllUsers() {
    List<UserDTO> users = userService.findAll();
    return ResponseEntity.ok(users);
  }

  @GetMapping("/{id}")
  public ResponseEntity<UserDTO> getUserById(@PathVariable Long id) {
    return userService.findById(id)
        .map(ResponseEntity::ok)
        .orElse(ResponseEntity.notFound().build());
  }

  @PostMapping
  public ResponseEntity<UserDTO> createUser(@Valid @RequestBody CreateUserRequest request) {
    UserDTO created = userService.create(request);
    return ResponseEntity
        .created(URI.create("/api/users/" + created.getId()))
        .body(created);
  }

  @PutMapping("/{id}")
  public ResponseEntity<UserDTO> updateUser(
      @PathVariable Long id,
      @Valid @RequestBody UpdateUserRequest request) {
    return userService.update(id, request)
        .map(ResponseEntity::ok)
        .orElse(ResponseEntity.notFound().build());
  }

  @DeleteMapping("/{id}")
  public ResponseEntity<Void> deleteUser(@PathVariable Long id) {
    userService.delete(id);
    return ResponseEntity.noContent().build();
  }
}
```

---

## Service Layer

```java
@Service
public class UserService {

  private final UserRepository userRepository;
  private final EmailService emailService;

  @Autowired
  public UserService(UserRepository userRepository, EmailService emailService) {
    this.userRepository = userRepository;
    this.emailService = emailService;
  }

  @Transactional(readOnly = true)
  public List<UserDTO> findAll() {
    return userRepository.findAll().stream()
        .map(this::toDTO)
        .collect(Collectors.toList());
  }

  @Transactional(readOnly = true)
  public Optional<UserDTO> findById(Long id) {
    return userRepository.findById(id)
        .map(this::toDTO);
  }

  @Transactional
  public UserDTO create(CreateUserRequest request) {
    User user = new User();
    user.setName(request.getName());
    user.setEmail(request.getEmail());

    User saved = userRepository.save(user);
    emailService.sendWelcomeEmail(saved.getEmail());

    return toDTO(saved);
  }

  @Transactional
  public Optional<UserDTO> update(Long id, UpdateUserRequest request) {
    return userRepository.findById(id)
        .map(user -> {
          user.setName(request.getName());
          user.setEmail(request.getEmail());
          return toDTO(userRepository.save(user));
        });
  }

  @Transactional
  public void delete(Long id) {
    userRepository.deleteById(id);
  }

  private UserDTO toDTO(User user) {
    return new UserDTO(user.getId(), user.getName(), user.getEmail());
  }
}
```

---

## Repository Layer

**Spring Data JPA:**

```java
@Repository
public interface UserRepository extends JpaRepository<User, Long> {

  Optional<User> findByEmail(String email);

  List<User> findByStatus(Status status);

  @Query("SELECT u FROM User u WHERE u.createdAt > :date")
  List<User> findUsersCreatedAfter(@Param("date") LocalDateTime date);

  @Modifying
  @Query("UPDATE User u SET u.status = :status WHERE u.id = :id")
  int updateStatus(@Param("id") Long id, @Param("status") Status status);
}
```

---

## Entity Classes

```java
@Entity
@Table(name = "users")
public class User {

  @Id
  @GeneratedValue(strategy = GenerationType.IDENTITY)
  private Long id;

  @Column(nullable = false)
  private String name;

  @Column(nullable = false, unique = true)
  private String email;

  @Enumerated(EnumType.STRING)
  private Status status;

  @CreatedDate
  @Column(nullable = false, updatable = false)
  private LocalDateTime createdAt;

  @LastModifiedDate
  private LocalDateTime updatedAt;

  // Constructors
  public User() {}

  public User(String name, String email) {
    this.name = name;
    this.email = email;
    this.status = Status.ACTIVE;
  }

  // Getters and Setters
  public Long getId() { return id; }
  public void setId(Long id) { this.id = id; }

  public String getName() { return name; }
  public void setName(String name) { this.name = name; }

  public String getEmail() { return email; }
  public void setEmail(String email) { this.email = email; }

  public Status getStatus() { return status; }
  public void setStatus(Status status) { this.status = status; }
}
```

---

## DTOs (Data Transfer Objects)

```java
public record UserDTO(
    Long id,
    String name,
    String email
) {}

public record CreateUserRequest(
    @NotBlank(message = "Name is required")
    String name,

    @NotBlank(message = "Email is required")
    @Email(message = "Email must be valid")
    String email
) {}

public record UpdateUserRequest(
    @NotBlank(message = "Name is required")
    String name,

    @NotBlank(message = "Email is required")
    @Email(message = "Email must be valid")
    String email
) {}
```

---

## Configuration

**application.yml:**

```yaml
spring:
  application:
    name: my-application

  datasource:
    url: jdbc:postgresql://localhost:5432/mydb
    username: ${DB_USERNAME:postgres}
    password: ${DB_PASSWORD:password}
    driver-class-name: org.postgresql.Driver

  jpa:
    hibernate:
      ddl-auto: validate
    show-sql: false
    properties:
      hibernate:
        format_sql: true
        dialect: org.hibernate.dialect.PostgreSQLDialect

  liquibase:
    change-log: classpath:db/changelog/db.changelog-master.xml

server:
  port: 8080
  compression:
    enabled: true

logging:
  level:
    root: INFO
    com.example.myapp: DEBUG
```

**Profile-specific configuration:**

```yaml
# application-dev.yml
spring:
  jpa:
    show-sql: true
    hibernate:
      ddl-auto: create-drop

# application-prod.yml
spring:
  jpa:
    show-sql: false
    hibernate:
      ddl-auto: validate

logging:
  level:
    root: WARN
    com.example.myapp: INFO
```

---

## Configuration Classes

```java
@Configuration
public class AppConfig {

  @Bean
  public RestTemplate restTemplate() {
    return new RestTemplate();
  }

  @Bean
  public ObjectMapper objectMapper() {
    ObjectMapper mapper = new ObjectMapper();
    mapper.registerModule(new JavaTimeModule());
    mapper.disable(SerializationFeature.WRITE_DATES_AS_TIMESTAMPS);
    return mapper;
  }
}
```

---

## Exception Handling

**Global exception handler:**

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

  @ExceptionHandler(ResourceNotFoundException.class)
  public ResponseEntity<ErrorResponse> handleResourceNotFound(
      ResourceNotFoundException ex) {
    ErrorResponse error = new ErrorResponse(
        HttpStatus.NOT_FOUND.value(),
        ex.getMessage(),
        LocalDateTime.now()
    );
    return ResponseEntity.status(HttpStatus.NOT_FOUND).body(error);
  }

  @ExceptionHandler(ValidationException.class)
  public ResponseEntity<ErrorResponse> handleValidation(
      ValidationException ex) {
    ErrorResponse error = new ErrorResponse(
        HttpStatus.BAD_REQUEST.value(),
        ex.getMessage(),
        LocalDateTime.now()
    );
    return ResponseEntity.badRequest().body(error);
  }

  @ExceptionHandler(MethodArgumentNotValidException.class)
  public ResponseEntity<Map<String, String>> handleValidationErrors(
      MethodArgumentNotValidException ex) {
    Map<String, String> errors = new HashMap<>();
    ex.getBindingResult().getFieldErrors().forEach(error ->
        errors.put(error.getField(), error.getDefaultMessage())
    );
    return ResponseEntity.badRequest().body(errors);
  }

  @ExceptionHandler(Exception.class)
  public ResponseEntity<ErrorResponse> handleGeneral(Exception ex) {
    ErrorResponse error = new ErrorResponse(
        HttpStatus.INTERNAL_SERVER_ERROR.value(),
        "An unexpected error occurred",
        LocalDateTime.now()
    );
    return ResponseEntity.status(HttpStatus.INTERNAL_SERVER_ERROR).body(error);
  }
}

record ErrorResponse(int status, String message, LocalDateTime timestamp) {}
```

---

## Validation

```java
@RestController
@RequestMapping("/api/users")
@Validated
public class UserController {

  @PostMapping
  public ResponseEntity<UserDTO> createUser(
      @Valid @RequestBody CreateUserRequest request) {
    // Spring automatically validates and throws MethodArgumentNotValidException
    UserDTO created = userService.create(request);
    return ResponseEntity.ok(created);
  }
}

// Request DTO with validation
public record CreateUserRequest(
    @NotBlank(message = "Name is required")
    @Size(min = 2, max = 100, message = "Name must be between 2 and 100 characters")
    String name,

    @NotBlank(message = "Email is required")
    @Email(message = "Email must be valid")
    String email,

    @Min(value = 18, message = "Age must be at least 18")
    @Max(value = 120, message = "Age must be at most 120")
    Integer age
) {}
```

---

## Async Processing

```java
@Configuration
@EnableAsync
public class AsyncConfig {

  @Bean
  public Executor taskExecutor() {
    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();
    executor.setCorePoolSize(2);
    executor.setMaxPoolSize(5);
    executor.setQueueCapacity(100);
    executor.setThreadNamePrefix("async-");
    executor.initialize();
    return executor;
  }
}

@Service
public class EmailService {

  @Async
  public CompletableFuture<Void> sendEmailAsync(String to, String subject, String body) {
    // Send email
    sendEmail(to, subject, body);
    return CompletableFuture.completedFuture(null);
  }

  private void sendEmail(String to, String subject, String body) {
    // Implementation
  }
}
```

---

## Scheduling

```java
@Configuration
@EnableScheduling
public class SchedulingConfig {
}

@Component
public class ScheduledTasks {

  private final Logger logger = LoggerFactory.getLogger(ScheduledTasks.class);

  @Scheduled(fixedRate = 60000) // Every minute
  public void reportCurrentTime() {
    logger.info("Current time: {}", LocalDateTime.now());
  }

  @Scheduled(cron = "0 0 2 * * ?") // Every day at 2 AM
  public void performNightlyCleanup() {
    logger.info("Performing nightly cleanup");
    // Cleanup logic
  }
}
```

---

## Caching

```java
@Configuration
@EnableCaching
public class CacheConfig {

  @Bean
  public CacheManager cacheManager() {
    return new ConcurrentMapCacheManager("users", "products");
  }
}

@Service
public class UserService {

  @Cacheable(value = "users", key = "#id")
  public UserDTO findById(Long id) {
    // This result will be cached
    return userRepository.findById(id)
        .map(this::toDTO)
        .orElseThrow(() -> new ResourceNotFoundException("User not found"));
  }

  @CacheEvict(value = "users", key = "#id")
  public void delete(Long id) {
    userRepository.deleteById(id);
  }

  @CachePut(value = "users", key = "#result.id")
  public UserDTO update(Long id, UpdateUserRequest request) {
    // Update and refresh cache
    User user = userRepository.findById(id)
        .orElseThrow(() -> new ResourceNotFoundException("User not found"));
    user.setName(request.getName());
    return toDTO(userRepository.save(user));
  }
}
```

---

## Security (Spring Security)

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {

  @Bean
  public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
    http
        .csrf(csrf -> csrf.disable())
        .authorizeHttpRequests(auth -> auth
            .requestMatchers("/api/public/**").permitAll()
            .requestMatchers("/api/admin/**").hasRole("ADMIN")
            .anyRequest().authenticated()
        )
        .httpBasic(Customizer.withDefaults());

    return http.build();
  }

  @Bean
  public PasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
  }
}
```

---

## Testing

**Controller test:**

```java
@WebMvcTest(UserController.class)
class UserControllerTest {

  @Autowired
  private MockMvc mockMvc;

  @MockBean
  private UserService userService;

  @Test
  void shouldReturnUser() throws Exception {
    UserDTO user = new UserDTO(1L, "John", "john@example.com");
    when(userService.findById(1L)).thenReturn(Optional.of(user));

    mockMvc.perform(get("/api/users/1"))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.name").value("John"));
  }
}
```

**Service test:**

```java
@SpringBootTest
class UserServiceTest {

  @Autowired
  private UserService userService;

  @MockBean
  private UserRepository userRepository;

  @Test
  void shouldCreateUser() {
    User user = new User("John", "john@example.com");
    when(userRepository.save(any(User.class))).thenReturn(user);

    CreateUserRequest request = new CreateUserRequest("John", "john@example.com");
    UserDTO created = userService.create(request);

    assertNotNull(created);
    assertEquals("John", created.name());
  }
}
```

---

## Best Practices Summary

1. **Layered architecture**: Controller → Service → Repository
2. **Constructor injection**: For dependencies
3. **DTOs**: Separate internal models from API
4. **Validation**: Use @Valid and bean validation
5. **Exception handling**: @RestControllerAdvice
6. **Transactions**: @Transactional on service methods
7. **Configuration**: Externalize via application.yml
8. **Profiles**: Different configs for dev/prod
9. **Testing**: Use Spring Boot test slices
10. **Security**: Configure with Spring Security

---

## References

- [Spring Boot Documentation](https://spring.io/projects/spring-boot)
- [Spring Data JPA](https://spring.io/projects/spring-data-jpa)
- [Spring Security](https://spring.io/projects/spring-security)
- [Baeldung Spring Tutorials](https://www.baeldung.com/spring-tutorial)
