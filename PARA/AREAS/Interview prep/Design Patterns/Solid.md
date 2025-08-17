---
area: Design Patterns
tags:
  - design-patterns
  - solid-principles
type: 
created: 2025-08-17 12:13
---

# Overview


## Introduction to SOLID

SOLID is an acronym for five design principles that make software designs more understandable, flexible, and maintainable. These principles were introduced by Robert C. Martin (Uncle Bob) and form the foundation of good object-oriented design.

**S** - Single Responsibility Principle  
**O** - Open/Closed Principle  
**L** - Liskov Substitution Principle  
**I** - Interface Segregation Principle  
**D** - Dependency Inversion Principle

## Single Responsibility Principle #SRP

### Definition

> "A class should have only one reason to change."

A class should have only one job or responsibility. If a class has multiple responsibilities, changes to one responsibility may affect the other, making the system fragile.

### ❌ Violation Example

```java
// BAD: Multiple responsibilities in one class
public class Employee {
    private String name;
    private String position;
    private double salary;
    
    // Employee data management
    public void setName(String name) { this.name = name; }
    public String getName() { return name; }
    
    // Salary calculation logic
    public double calculatePay() {
        if (position.equals("Manager")) {
            return salary * 1.5;
        }
        return salary;
    }
    
    // Database operations
    public void save() {
        // Database saving logic
        System.out.println("Saving employee to database...");
    }
    
    // Report generation
    public void generateReport() {
        System.out.println("Employee Report:");
        System.out.println("Name: " + name);
        System.out.println("Position: " + position);
        System.out.println("Salary: " + calculatePay());
    }
}
```

**Problems:**

- Changes in pay calculation affect the Employee class
- Database schema changes affect the Employee class
- Report format changes affect the Employee class
- Difficult to test individual responsibilities

### ✅ SRP Compliant Solution

```java
// Employee data only
public class Employee {
    private String name;
    private String position;
    private double salary;
    
    public Employee(String name, String position, double salary) {
        this.name = name;
        this.position = position;
        this.salary = salary;
    }
    
    // Getters and setters only
    public String getName() { return name; }
    public String getPosition() { return position; }
    public double getSalary() { return salary; }
}

// Salary calculation responsibility
public class PayrollCalculator {
    public double calculatePay(Employee employee) {
        if (employee.getPosition().equals("Manager")) {
            return employee.getSalary() * 1.5;
        }
        return employee.getSalary();
    }
}

// Database operations responsibility
public class EmployeeRepository {
    public void save(Employee employee) {
        System.out.println("Saving employee to database: " + employee.getName());
    }
    
    public Employee findById(Long id) {
        // Database retrieval logic
        return new Employee("John Doe", "Developer", 50000);
    }
}

// Report generation responsibility
public class EmployeeReportGenerator {
    private PayrollCalculator payrollCalculator;
    
    public EmployeeReportGenerator(PayrollCalculator payrollCalculator) {
        this.payrollCalculator = payrollCalculator;
    }
    
    public void generateReport(Employee employee) {
        System.out.println("Employee Report:");
        System.out.println("Name: " + employee.getName());
        System.out.println("Position: " + employee.getPosition());
        System.out.println("Salary: " + payrollCalculator.calculatePay(employee));
    }
}
```

**Benefits:**

- Each class has a single reason to change
- Easier to test each responsibility independently
- Better code organization and maintainability

## Open/Closed Principle #OCP

### Definition

> "Software entities should be open for extension but closed for modification."

You should be able to add new functionality without changing existing code. This is typically achieved through inheritance, interfaces, and composition.

### ❌ Violation Example

```java
// BAD: Must modify existing code to add new shapes
public class AreaCalculator {
    public double calculateArea(Object shape) {
        if (shape instanceof Rectangle) {
            Rectangle rectangle = (Rectangle) shape;
            return rectangle.getWidth() * rectangle.getHeight();
        } else if (shape instanceof Circle) {
            Circle circle = (Circle) shape;
            return Math.PI * circle.getRadius() * circle.getRadius();
        }
        // Need to modify this method every time we add a new shape!
        return 0;
    }
}

class Rectangle {
    private double width, height;
    public Rectangle(double width, double height) {
        this.width = width; this.height = height;
    }
    public double getWidth() { return width; }
    public double getHeight() { return height; }
}

class Circle {
    private double radius;
    public Circle(double radius) { this.radius = radius; }
    public double getRadius() { return radius; }
}
```

### ✅ OCP Compliant Solution

```java
// Abstract base class defining the contract
public abstract class Shape {
    public abstract double calculateArea();
}

// Concrete implementations
public class Rectangle extends Shape {
    private double width, height;
    
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    
    @Override
    public double calculateArea() {
        return width * height;
    }
}

public class Circle extends Shape {
    private double radius;
    
    public Circle(double radius) {
        this.radius = radius;
    }
    
    @Override
    public double calculateArea() {
        return Math.PI * radius * radius;
    }
}

// New shape can be added without modifying existing code
public class Triangle extends Shape {
    private double base, height;
    
    public Triangle(double base, double height) {
        this.base = base;
        this.height = height;
    }
    
    @Override
    public double calculateArea() {
        return 0.5 * base * height;
    }
}

// Calculator doesn't need modification for new shapes
public class AreaCalculator {
    public double calculateTotalArea(List<Shape> shapes) {
        return shapes.stream()
                    .mapToDouble(Shape::calculateArea)
                    .sum();
    }
}
```

### Advanced OCP Example: Strategy Pattern

```java
// Payment processing example
public interface PaymentProcessor {
    void processPayment(double amount);
}

public class CreditCardProcessor implements PaymentProcessor {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing $" + amount + " via Credit Card");
    }
}

public class PayPalProcessor implements PaymentProcessor {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing $" + amount + " via PayPal");
    }
}

// Can add new payment methods without modifying existing code
public class CryptocurrencyProcessor implements PaymentProcessor {
    @Override
    public void processPayment(double amount) {
        System.out.println("Processing $" + amount + " via Cryptocurrency");
    }
}

public class PaymentService {
    public void processPayment(PaymentProcessor processor, double amount) {
        processor.processPayment(amount);
    }
}
```

## Liskov Substitution Principle #LSP

### Definition

> "Objects of a superclass should be replaceable with objects of its subclasses without breaking the application."

Subtypes must be substitutable for their base types. This means derived classes must not strengthen preconditions or weaken postconditions.

### ❌ Violation Example

```java
// BAD: Square violates LSP when substituted for Rectangle
public class Rectangle {
    protected double width, height;
    
    public void setWidth(double width) { this.width = width; }
    public void setHeight(double height) { this.height = height; }
    public double getWidth() { return width; }
    public double getHeight() { return height; }
    public double getArea() { return width * height; }
}

public class Square extends Rectangle {
    @Override
    public void setWidth(double width) {
        this.width = width;
        this.height = width;  // Violates expectations!
    }
    
    @Override
    public void setHeight(double height) {
        this.width = height;  // Violates expectations!
        this.height = height;
    }
}

// This test will fail for Square
public class GeometryTest {
    public void testRectangle(Rectangle rectangle) {
        rectangle.setWidth(5);
        rectangle.setHeight(4);
        
        // Expected: 20, but Square will give 16!
        assert rectangle.getArea() == 20; // Fails for Square
    }
}
```

### ✅ LSP Compliant Solution

```java
// Better design using composition and proper abstraction
public abstract class Shape {
    public abstract double getArea();
    public abstract double getPerimeter();
}

public class Rectangle extends Shape {
    private final double width, height;
    
    public Rectangle(double width, double height) {
        this.width = width;
        this.height = height;
    }
    
    public double getWidth() { return width; }
    public double getHeight() { return height; }
    
    @Override
    public double getArea() { return width * height; }
    
    @Override
    public double getPerimeter() { return 2 * (width + height); }
}

public class Square extends Shape {
    private final double side;
    
    public Square(double side) {
        this.side = side;
    }
    
    public double getSide() { return side; }
    
    @Override
    public double getArea() { return side * side; }
    
    @Override
    public double getPerimeter() { return 4 * side; }
}

// Both can be used interchangeably as Shape
public class ShapeCalculator {
    public double calculateTotalArea(List<Shape> shapes) {
        return shapes.stream()
                    .mapToDouble(Shape::getArea)
                    .sum();
    }
}
```

### Real-World LSP Example: Bird Hierarchy

```java
// ❌ BAD: Not all birds can fly
public class Bird {
    public void fly() {
        System.out.println("Flying...");
    }
}

public class Penguin extends Bird {
    @Override
    public void fly() {
        throw new UnsupportedOperationException("Penguins can't fly!");
    }
}

// ✅ GOOD: Proper segregation
public abstract class Bird {
    public abstract void eat();
    public abstract void sleep();
}

public interface Flyable {
    void fly();
}

public interface Swimmable {
    void swim();
}

public class Eagle extends Bird implements Flyable {
    @Override
    public void eat() { System.out.println("Eagle eating..."); }
    
    @Override
    public void sleep() { System.out.println("Eagle sleeping..."); }
    
    @Override
    public void fly() { System.out.println("Eagle flying high..."); }
}

public class Penguin extends Bird implements Swimmable {
    @Override
    public void eat() { System.out.println("Penguin eating fish..."); }
    
    @Override
    public void sleep() { System.out.println("Penguin sleeping..."); }
    
    @Override
    public void swim() { System.out.println("Penguin swimming..."); }
}
```

## Interface Segregation Principle #ISP

### Definition

> "Clients should not be forced to depend on interfaces they do not use."

Create smaller, more focused interfaces rather than large, monolithic ones. This prevents classes from implementing methods they don't need.

### ❌ Violation Example

```java
// BAD: Fat interface that forces unnecessary implementations
public interface Worker {
    void work();
    void eat();
    void sleep();
    void attendMeeting();
    void writeCode();
    void designUI();
    void manageTeam();
}

// Human worker implements all methods
public class HumanWorker implements Worker {
    @Override
    public void work() { System.out.println("Human working..."); }
    @Override
    public void eat() { System.out.println("Human eating..."); }
    @Override
    public void sleep() { System.out.println("Human sleeping..."); }
    @Override
    public void attendMeeting() { System.out.println("Attending meeting..."); }
    @Override
    public void writeCode() { System.out.println("Writing code..."); }
    @Override
    public void designUI() { System.out.println("Designing UI..."); }
    @Override
    public void manageTeam() { System.out.println("Managing team..."); }
}

// Robot forced to implement biological methods!
public class RobotWorker implements Worker {
    @Override
    public void work() { System.out.println("Robot working..."); }
    @Override
    public void eat() { throw new UnsupportedOperationException(); }
    @Override
    public void sleep() { throw new UnsupportedOperationException(); }
    @Override
    public void attendMeeting() { throw new UnsupportedOperationException(); }
    @Override
    public void writeCode() { System.out.println("Robot coding..."); }
    @Override
    public void designUI() { throw new UnsupportedOperationException(); }
    @Override
    public void manageTeam() { throw new UnsupportedOperationException(); }
}
```

### ✅ ISP Compliant Solution

```java
// Segregated interfaces
public interface Workable {
    void work();
}

public interface Biological {
    void eat();
    void sleep();
}

public interface Communicable {
    void attendMeeting();
}

public interface Programmer {
    void writeCode();
}

public interface Designer {
    void designUI();
}

public interface Manager {
    void manageTeam();
}

// Implementations only implement what they need
public class Developer implements Workable, Biological, Communicable, Programmer {
    @Override
    public void work() { System.out.println("Developer working..."); }
    @Override
    public void eat() { System.out.println("Developer eating..."); }
    @Override
    public void sleep() { System.out.println("Developer sleeping..."); }
    @Override
    public void attendMeeting() { System.out.println("Developer in meeting..."); }
    @Override
    public void writeCode() { System.out.println("Developer coding..."); }
}

public class TeamLead implements Workable, Biological, Communicable, Programmer, Manager {
    @Override
    public void work() { System.out.println("TeamLead working..."); }
    @Override
    public void eat() { System.out.println("TeamLead eating..."); }
    @Override
    public void sleep() { System.out.println("TeamLead sleeping..."); }
    @Override
    public void attendMeeting() { System.out.println("TeamLead in meeting..."); }
    @Override
    public void writeCode() { System.out.println("TeamLead coding..."); }
    @Override
    public void manageTeam() { System.out.println("TeamLead managing..."); }
}

public class Robot implements Workable, Programmer {
    @Override
    public void work() { System.out.println("Robot working 24/7..."); }
    @Override
    public void writeCode() { System.out.println("Robot generating code..."); }
}
```

### Real-World ISP Example: Document Processing

```java
// ❌ BAD: Monolithic interface
public interface Document {
    void open();
    void save();
    void print();
    void fax();
    void scan();
    void email();
}

// ✅ GOOD: Segregated interfaces
public interface Readable {
    void open();
}

public interface Writable {
    void save();
}

public interface Printable {
    void print();
}

public interface Faxable {
    void fax();
}

public interface Scannable {
    void scan();
}

public interface Emailable {
    void email();
}

// Simple text document
public class TextDocument implements Readable, Writable, Printable, Emailable {
    @Override
    public void open() { System.out.println("Opening text document..."); }
    @Override
    public void save() { System.out.println("Saving text document..."); }
    @Override
    public void print() { System.out.println("Printing text document..."); }
    @Override
    public void email() { System.out.println("Emailing text document..."); }
}

// Multi-function printer
public class MultiFunctionPrinter implements Printable, Faxable, Scannable {
    @Override
    public void print() { System.out.println("Printing..."); }
    @Override
    public void fax() { System.out.println("Faxing..."); }
    @Override
    public void scan() { System.out.println("Scanning..."); }
}
```

## Dependency Inversion Principle #DIP

### Definition

> "High-level modules should not depend on low-level modules. Both should depend on abstractions. Abstractions should not depend on details. Details should depend on abstractions."

Depend on interfaces or abstract classes rather than concrete classes. This makes your code more flexible and testable.

### ❌ Violation Example

```java
// BAD: High-level class depends on low-level concrete classes
public class EmailService {
    public void sendEmail(String message) {
        System.out.println("Sending email: " + message);
    }
}

public class SMSService {
    public void sendSMS(String message) {
        System.out.println("Sending SMS: " + message);
    }
}

// NotificationService is tightly coupled to concrete implementations
public class NotificationService {
    private EmailService emailService;  // Concrete dependency
    private SMSService smsService;      // Concrete dependency
    
    public NotificationService() {
        this.emailService = new EmailService();  // Hard-coded dependency
        this.smsService = new SMSService();      // Hard-coded dependency
    }
    
    public void sendNotification(String message, String type) {
        if (type.equals("email")) {
            emailService.sendEmail(message);
        } else if (type.equals("sms")) {
            smsService.sendSMS(message);
        }
        // Adding new notification types requires modifying this class!
    }
}
```

### ✅ DIP Compliant Solution

```java
// Abstraction that both high-level and low-level modules depend on
public interface NotificationSender {
    void send(String message);
}

// Low-level modules implement the abstraction
public class EmailService implements NotificationSender {
    @Override
    public void send(String message) {
        System.out.println("Sending email: " + message);
    }
}

public class SMSService implements NotificationSender {
    @Override
    public void send(String message) {
        System.out.println("Sending SMS: " + message);
    }
}

public class PushNotificationService implements NotificationSender {
    @Override
    public void send(String message) {
        System.out.println("Sending push notification: " + message);
    }
}

// High-level module depends on abstraction
public class NotificationService {
    private final List<NotificationSender> notificationSenders;
    
    // Dependency injection through constructor
    public NotificationService(List<NotificationSender> notificationSenders) {
        this.notificationSenders = notificationSenders;
    }
    
    public void sendNotification(String message) {
        notificationSenders.forEach(sender -> sender.send(message));
    }
}

// Usage with dependency injection
public class Application {
    public static void main(String[] args) {
        List<NotificationSender> senders = Arrays.asList(
            new EmailService(),
            new SMSService(),
            new PushNotificationService()
        );
        
        NotificationService service = new NotificationService(senders);
        service.sendNotification("Hello World!");
    }
}
```

### Advanced DIP Example: Repository Pattern

```java
// Abstraction for data access
public interface UserRepository {
    void save(User user);
    User findById(Long id);
    List<User> findAll();
}

// Low-level implementations
public class DatabaseUserRepository implements UserRepository {
    @Override
    public void save(User user) {
        System.out.println("Saving user to database: " + user.getName());
    }
    
    @Override
    public User findById(Long id) {
        return new User(id, "John Doe", "john@example.com");
    }
    
    @Override
    public List<User> findAll() {
        return Arrays.asList(
            new User(1L, "John Doe", "john@example.com"),
            new User(2L, "Jane Smith", "jane@example.com")
        );
    }
}

public class InMemoryUserRepository implements UserRepository {
    private final Map<Long, User> users = new HashMap<>();
    private Long nextId = 1L;
    
    @Override
    public void save(User user) {
        if (user.getId() == null) {
            user.setId(nextId++);
        }
        users.put(user.getId(), user);
        System.out.println("Saving user to memory: " + user.getName());
    }
    
    @Override
    public User findById(Long id) {
        return users.get(id);
    }
    
    @Override
    public List<User> findAll() {
        return new ArrayList<>(users.values());
    }
}

// High-level service depends on abstraction
public class UserService {
    private final UserRepository userRepository;
    
    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }
    
    public void createUser(String name, String email) {
        User user = new User(null, name, email);
        userRepository.save(user);
    }
    
    public User getUser(Long id) {
        return userRepository.findById(id);
    }
    
    public List<User> getAllUsers() {
        return userRepository.findAll();
    }
}

// User entity
public class User {
    private Long id;
    private String name;
    private String email;
    
    public User(Long id, String name, String email) {
        this.id = id;
        this.name = name;
        this.email = email;
    }
    
    // Getters and setters
    public Long getId() { return id; }
    public void setId(Long id) { this.id = id; }
    public String getName() { return name; }
    public String getEmail() { return email; }
}
```

## Putting It All Together

### E-commerce Example Following All SOLID Principles

```java
// SRP: Single responsibility for order data
public class Order {
    private String orderId;
    private List<OrderItem> items;
    private OrderStatus status;
    private double totalAmount;
    
    public Order(String orderId, List<OrderItem> items) {
        this.orderId = orderId;
        this.items = items;
        this.status = OrderStatus.PENDING;
        this.totalAmount = calculateTotal();
    }
    
    private double calculateTotal() {
        return items.stream().mapToDouble(OrderItem::getSubtotal).sum();
    }
    
    // Getters...
    public String getOrderId() { return orderId; }
    public List<OrderItem> getItems() { return items; }
    public OrderStatus getStatus() { return status; }
    public double getTotalAmount() { return totalAmount; }
    public void setStatus(OrderStatus status) { this.status = status; }
}

public class OrderItem {
    private String productId;
    private int quantity;
    private double price;
    
    public OrderItem(String productId, int quantity, double price) {
        this.productId = productId;
        this.quantity = quantity;
        this.price = price;
    }
    
    public double getSubtotal() { return quantity * price; }
    
    // Getters...
    public String getProductId() { return productId; }
    public int getQuantity() { return quantity; }
    public double getPrice() { return price; }
}

public enum OrderStatus {
    PENDING, CONFIRMED, SHIPPED, DELIVERED, CANCELLED
}

// OCP & DIP: Payment processing strategy
public interface PaymentProcessor {
    PaymentResult processPayment(double amount, PaymentDetails details);
}

public class CreditCardProcessor implements PaymentProcessor {
    @Override
    public PaymentResult processPayment(double amount, PaymentDetails details) {
        // Credit card processing logic
        return new PaymentResult(true, "CC-" + System.currentTimeMillis());
    }
}

public class PayPalProcessor implements PaymentProcessor {
    @Override
    public PaymentResult processPayment(double amount, PaymentDetails details) {
        // PayPal processing logic
        return new PaymentResult(true, "PP-" + System.currentTimeMillis());
    }
}

// ISP: Segregated interfaces for different responsibilities
public interface OrderRepository {
    void save(Order order);
    Order findById(String orderId);
}

public interface NotificationSender {
    void sendOrderConfirmation(Order order);
}

public interface InventoryService {
    boolean isAvailable(String productId, int quantity);
    void reserveItems(List<OrderItem> items);
}

// SRP: Order processing service
public class OrderProcessor {
    private final OrderRepository orderRepository;
    private final PaymentProcessor paymentProcessor;
    private final NotificationSender notificationSender;
    private final InventoryService inventoryService;
    
    // DIP: Dependencies injected through constructor
    public OrderProcessor(
        OrderRepository orderRepository,
        PaymentProcessor paymentProcessor,
        NotificationSender notificationSender,
        InventoryService inventoryService
    ) {
        this.orderRepository = orderRepository;
        this.paymentProcessor = paymentProcessor;
        this.notificationSender = notificationSender;
        this.inventoryService = inventoryService;
    }
    
    public OrderResult processOrder(Order order, PaymentDetails paymentDetails) {
        // Check inventory
        if (!checkInventory(order)) {
            return new OrderResult(false, "Insufficient inventory");
        }
        
        // Process payment
        PaymentResult paymentResult = paymentProcessor.processPayment(
            order.getTotalAmount(), paymentDetails
        );
        
        if (!paymentResult.isSuccessful()) {
            return new OrderResult(false, "Payment failed");
        }
        
        // Reserve inventory
        inventoryService.reserveItems(order.getItems());
        
        // Update order status
        order.setStatus(OrderStatus.CONFIRMED);
        
        // Save order
        orderRepository.save(order);
        
        // Send notification
        notificationSender.sendOrderConfirmation(order);
        
        return new OrderResult(true, "Order processed successfully");
    }
    
    private boolean checkInventory(Order order) {
        return order.getItems().stream()
                   .allMatch(item -> inventoryService.isAvailable(
                       item.getProductId(), item.getQuantity()));
    }
}

// Supporting classes
public class PaymentDetails {
    private final String cardNumber;
    private final String expiryDate;
    
    public PaymentDetails(String cardNumber, String expiryDate) {
        this.cardNumber = cardNumber;
        this.expiryDate = expiryDate;
    }
    
    public String getCardNumber() { return cardNumber; }
    public String getExpiryDate() { return expiryDate; }
}

public class PaymentResult {
    private final boolean successful;
    private final String transactionId;
    
    public PaymentResult(boolean successful, String transactionId) {
        this.successful = successful;
        this.transactionId = transactionId;
    }
    
    public boolean isSuccessful() { return successful; }
    public String getTransactionId() { return transactionId; }
}

public class OrderResult {
    private final boolean successful;
    private final String message;
    
    public OrderResult(boolean successful, String message) {
        this.successful = successful;
        this.message = message;
    }
    
    public boolean isSuccessful() { return successful; }
    public String getMessage() { return message; }
}
```

## Benefits and Trade-offs

### Benefits of SOLID Principles

**Maintainability:**

- Changes to one part don't break other parts
- Easier to locate and fix bugs
- Code is more organized and readable

**Testability:**

- Each class has a single responsibility
- Dependencies can be mocked easily
- Unit tests are more focused

**Flexibility:**

- Easy to add new features without modifying existing code
- Components can be swapped easily
- Better support for different implementations

**Reusability:**

- Smaller, focused classes are easier to reuse
- Interfaces allow for multiple implementations
- Loose coupling enables component reuse

### Trade-offs and Considerations

**Increased Complexity:**

- More classes and interfaces
- Additional abstraction layers
- Steeper learning curve for new developers

**Over-engineering Risk:**

- Can lead to unnecessary abstractions
- May complicate simple problems
- Premature optimization concerns

**Performance Overhead:**

- Additional method calls through interfaces
- Memory overhead from extra objects
- Potential impact on performance-critical code

### When to Apply SOLID Principles

**Apply SOLID when:**

- Building medium to large applications
- Code will be maintained by multiple developers
- Requirements are likely to change
- Testing is important
- Code reusability is desired

**Consider simpler approaches when:**

- Building small, simple applications
- Rapid prototyping
- Performance is critical
- Team has limited OOP experience
- Requirements are very stable

### Best Practices

1. **Start Simple:** Don't over-engineer from the beginning
2. **Refactor Gradually:** Apply SOLID principles as code grows
3. **Balance:** Don't sacrifice simplicity for strict adherence
4. **Context Matters:** Consider your specific use case
5. **Team Agreement:** Ensure team understands and agrees on approach

The SOLID principles are guidelines, not rigid rules. Use them to improve code quality while maintaining practical considerations for your specific context.