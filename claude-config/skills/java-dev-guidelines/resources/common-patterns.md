# Common Patterns in Java

## Core Principles

1. **Favor composition over inheritance** - More flexible
2. **Program to interfaces** - Not implementations
3. **DRY (Don't Repeat Yourself)** - Extract common code
4. **SOLID principles** - Clean, maintainable code
5. **Immutability** - Thread-safe, predictable

---

## Builder Pattern

**For objects with many parameters:**

```java
public class User {
  private final String name;
  private final String email;
  private final int age;
  private final String phone;
  private final String address;

  private User(Builder builder) {
    this.name = builder.name;
    this.email = builder.email;
    this.age = builder.age;
    this.phone = builder.phone;
    this.address = builder.address;
  }

  public static class Builder {
    private String name;
    private String email;
    private int age;
    private String phone;
    private String address;

    public Builder name(String name) {
      this.name = name;
      return this;
    }

    public Builder email(String email) {
      this.email = email;
      return this;
    }

    public Builder age(int age) {
      this.age = age;
      return this;
    }

    public Builder phone(String phone) {
      this.phone = phone;
      return this;
    }

    public Builder address(String address) {
      this.address = address;
      return this;
    }

    public User build() {
      // Validation
      Objects.requireNonNull(name, "name is required");
      Objects.requireNonNull(email, "email is required");
      return new User(this);
    }
  }

  // Getters
  public String getName() { return name; }
  public String getEmail() { return email; }
  // ...
}

// Usage
User user = new User.Builder()
    .name("John Doe")
    .email("john@example.com")
    .age(30)
    .phone("555-1234")
    .build();
```

---

## Factory Pattern

**Encapsulate object creation:**

```java
public interface Payment {
  void process(BigDecimal amount);
}

public class CreditCardPayment implements Payment {
  @Override
  public void process(BigDecimal amount) {
    // Process credit card payment
  }
}

public class PayPalPayment implements Payment {
  @Override
  public void process(BigDecimal amount) {
    // Process PayPal payment
  }
}

public class PaymentFactory {
  public static Payment createPayment(PaymentType type) {
    return switch (type) {
      case CREDIT_CARD -> new CreditCardPayment();
      case PAYPAL -> new PayPalPayment();
      case BANK_TRANSFER -> new BankTransferPayment();
      default -> throw new IllegalArgumentException("Unknown payment type: " + type);
    };
  }
}

// Usage
Payment payment = PaymentFactory.createPayment(PaymentType.CREDIT_CARD);
payment.process(new BigDecimal("99.99"));
```

---

## Singleton Pattern

**One instance per JVM:**

```java
public class DatabaseConnection {
  private static volatile DatabaseConnection instance;

  private DatabaseConnection() {
    // Private constructor
  }

  public static DatabaseConnection getInstance() {
    if (instance == null) {
      synchronized (DatabaseConnection.class) {
        if (instance == null) {
          instance = new DatabaseConnection();
        }
      }
    }
    return instance;
  }
}

// Better: Use enum (thread-safe by default)
public enum DatabaseConnection {
  INSTANCE;

  private Connection connection;

  DatabaseConnection() {
    // Initialize connection
  }

  public Connection getConnection() {
    return connection;
  }
}

// Usage
DatabaseConnection.INSTANCE.getConnection();
```

---

## Strategy Pattern

**Encapsulate algorithms:**

```java
public interface PricingStrategy {
  BigDecimal calculatePrice(BigDecimal basePrice);
}

public class RegularPricing implements PricingStrategy {
  @Override
  public BigDecimal calculatePrice(BigDecimal basePrice) {
    return basePrice;
  }
}

public class DiscountPricing implements PricingStrategy {
  private final BigDecimal discountPercent;

  public DiscountPricing(BigDecimal discountPercent) {
    this.discountPercent = discountPercent;
  }

  @Override
  public BigDecimal calculatePrice(BigDecimal basePrice) {
    BigDecimal discount = basePrice.multiply(discountPercent)
        .divide(BigDecimal.valueOf(100));
    return basePrice.subtract(discount);
  }
}

public class ShoppingCart {
  private PricingStrategy pricingStrategy;
  private List<Item> items;

  public void setPricingStrategy(PricingStrategy strategy) {
    this.pricingStrategy = strategy;
  }

  public BigDecimal getTotal() {
    BigDecimal sum = items.stream()
        .map(Item::getPrice)
        .reduce(BigDecimal.ZERO, BigDecimal::add);
    return pricingStrategy.calculatePrice(sum);
  }
}

// Usage
ShoppingCart cart = new ShoppingCart();
cart.setPricingStrategy(new DiscountPricing(new BigDecimal("10")));
BigDecimal total = cart.getTotal();
```

---

## Observer Pattern

**Notify dependents of state changes:**

```java
public interface Observer {
  void update(String message);
}

public class Subject {
  private List<Observer> observers = new ArrayList<>();

  public void attach(Observer observer) {
    observers.add(observer);
  }

  public void detach(Observer observer) {
    observers.remove(observer);
  }

  protected void notifyObservers(String message) {
    for (Observer observer : observers) {
      observer.update(message);
    }
  }
}

public class User extends Subject {
  private String status;

  public void setStatus(String status) {
    this.status = status;
    notifyObservers("User status changed to: " + status);
  }
}

public class EmailNotifier implements Observer {
  @Override
  public void update(String message) {
    System.out.println("Email notification: " + message);
  }
}

// Usage
User user = new User();
user.attach(new EmailNotifier());
user.attach(new SMSNotifier());
user.setStatus("ACTIVE");  // Notifies all observers
```

---

## Dependency Injection

**Inversion of Control:**

```java
// ❌ Bad - Tight coupling
public class UserService {
  private UserRepository repository = new DatabaseUserRepository();

  public User findById(Long id) {
    return repository.findById(id);
  }
}

// ✅ Good - Dependency injection
public class UserService {
  private final UserRepository repository;

  public UserService(UserRepository repository) {
    this.repository = repository;
  }

  public User findById(Long id) {
    return repository.findById(id);
  }
}

// Spring example
@Service
public class UserService {
  private final UserRepository repository;
  private final EmailService emailService;

  @Autowired
  public UserService(UserRepository repository, EmailService emailService) {
    this.repository = repository;
    this.emailService = emailService;
  }
}
```

---

## Template Method Pattern

**Define algorithm skeleton:**

```java
public abstract class DataProcessor {

  public final void process() {
    loadData();
    validateData();
    transformData();
    saveData();
  }

  protected abstract void loadData();

  protected void validateData() {
    // Default validation
    System.out.println("Validating data");
  }

  protected abstract void transformData();

  protected abstract void saveData();
}

public class CsvDataProcessor extends DataProcessor {
  @Override
  protected void loadData() {
    System.out.println("Loading CSV data");
  }

  @Override
  protected void transformData() {
    System.out.println("Transforming CSV data");
  }

  @Override
  protected void saveData() {
    System.out.println("Saving to database");
  }
}
```

---

## Adapter Pattern

**Make incompatible interfaces work together:**

```java
// Legacy interface
public class LegacyPaymentSystem {
  public void makePayment(String cardNumber, double amount) {
    // Legacy implementation
  }
}

// New interface
public interface PaymentProcessor {
  void processPayment(PaymentRequest request);
}

// Adapter
public class LegacyPaymentAdapter implements PaymentProcessor {
  private final LegacyPaymentSystem legacySystem;

  public LegacyPaymentAdapter(LegacyPaymentSystem legacySystem) {
    this.legacySystem = legacySystem;
  }

  @Override
  public void processPayment(PaymentRequest request) {
    legacySystem.makePayment(
        request.getCardNumber(),
        request.getAmount().doubleValue()
    );
  }
}
```

---

## Repository Pattern

**Abstract data access:**

```java
public interface UserRepository {
  User save(User user);
  Optional<User> findById(Long id);
  List<User> findAll();
  void delete(Long id);
}

@Repository
public class JpaUserRepository implements UserRepository {
  @PersistenceContext
  private EntityManager entityManager;

  @Override
  public User save(User user) {
    if (user.getId() == null) {
      entityManager.persist(user);
      return user;
    } else {
      return entityManager.merge(user);
    }
  }

  @Override
  public Optional<User> findById(Long id) {
    return Optional.ofNullable(entityManager.find(User.class, id));
  }

  @Override
  public List<User> findAll() {
    return entityManager
        .createQuery("SELECT u FROM User u", User.class)
        .getResultList();
  }

  @Override
  public void delete(Long id) {
    findById(id).ifPresent(entityManager::remove);
  }
}
```

---

## Fluent Interface

**Method chaining:**

```java
public class QueryBuilder {
  private StringBuilder query = new StringBuilder();
  private List<Object> parameters = new ArrayList<>();

  public QueryBuilder select(String... columns) {
    query.append("SELECT ").append(String.join(", ", columns));
    return this;
  }

  public QueryBuilder from(String table) {
    query.append(" FROM ").append(table);
    return this;
  }

  public QueryBuilder where(String condition) {
    query.append(" WHERE ").append(condition);
    return this;
  }

  public QueryBuilder orderBy(String column) {
    query.append(" ORDER BY ").append(column);
    return this;
  }

  public String build() {
    return query.toString();
  }
}

// Usage
String sql = new QueryBuilder()
    .select("id", "name", "email")
    .from("users")
    .where("status = 'ACTIVE'")
    .orderBy("name")
    .build();
```

---

## Null Object Pattern

**Avoid null checks:**

```java
public interface Logger {
  void log(String message);
}

public class ConsoleLogger implements Logger {
  @Override
  public void log(String message) {
    System.out.println(message);
  }
}

public class NullLogger implements Logger {
  @Override
  public void log(String message) {
    // Do nothing
  }
}

public class UserService {
  private final Logger logger;

  public UserService(Logger logger) {
    this.logger = logger != null ? logger : new NullLogger();
  }

  public void createUser(User user) {
    logger.log("Creating user: " + user.getName());
    // Create user
  }
}
```

---

## Command Pattern

**Encapsulate requests:**

```java
public interface Command {
  void execute();
  void undo();
}

public class CreateUserCommand implements Command {
  private final UserRepository repository;
  private final User user;
  private Long createdUserId;

  public CreateUserCommand(UserRepository repository, User user) {
    this.repository = repository;
    this.user = user;
  }

  @Override
  public void execute() {
    User created = repository.save(user);
    createdUserId = created.getId();
  }

  @Override
  public void undo() {
    if (createdUserId != null) {
      repository.delete(createdUserId);
    }
  }
}

public class CommandInvoker {
  private Stack<Command> history = new Stack<>();

  public void execute(Command command) {
    command.execute();
    history.push(command);
  }

  public void undo() {
    if (!history.isEmpty()) {
      Command command = history.pop();
      command.undo();
    }
  }
}
```

---

## Best Practices Summary

1. **Builder**: For objects with many optional parameters
2. **Factory**: Encapsulate complex object creation
3. **Singleton**: Use enum for thread safety
4. **Strategy**: Encapsulate interchangeable algorithms
5. **Observer**: Notify dependents of state changes
6. **DI**: Inject dependencies, don't create them
7. **Template Method**: Define algorithm skeleton
8. **Adapter**: Make incompatible interfaces work
9. **Repository**: Abstract data access
10. **Fluent Interface**: Method chaining for readability
11. **Null Object**: Avoid null checks
12. **Command**: Encapsulate requests as objects

---

## References

- [Design Patterns: Elements of Reusable Object-Oriented Software](https://en.wikipedia.org/wiki/Design_Patterns)
- [Head First Design Patterns](https://www.oreilly.com/library/view/head-first-design/0596007124/)
- [Effective Java (Joshua Bloch)](https://www.oreilly.com/library/view/effective-java/9780134686097/)
