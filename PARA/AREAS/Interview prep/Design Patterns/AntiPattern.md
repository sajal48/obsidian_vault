---
area: Design Patterns
tags:
  - design-patterns
  - anti-patterns
type: 
created: 2025-08-17 12:12
---

# Overview



A quick, practical guide to common software anti‑patterns with tight Java snippets and safer alternatives.


---

## #GodObject

**Smell:** One class knows/does too much.

```java
class OrderService { // God class
    private final Map<Long, Order> db = new HashMap<>();
    void save(Order o) { db.put(o.id(), o); }
    Order find(long id) { return db.get(id); }
    void chargeCard(Order o) { /* payment logic */ }
    void sendEmail(Order o) { /* email logic */ }
    void printInvoice(Order o) { /* printing logic */ }
}
```

**Better:** Separate responsibilities.

```java
class OrderRepository { void save(Order o) { /* ... */ } Order find(long id){/*...*/} }
class PaymentService { void charge(Order o) { /* ... */ } }
class NotificationService { void email(Order o) { /* ... */ } }
```

---

## #Spaghetti Code

**Smell:** Complex, nested conditionals hard to follow.

```java
if (u != null) {
    if (u.role() != null) {
        if (u.role().equals("ADMIN")) {
            // ... many lines ...
        }
    }
}
```

**Better:** Guard clauses + polymorphism.

```java
if (u == null || u.role() == null) return;
switch (u.role()) {
    case "ADMIN" -> adminHandler.handle(u);
    case "USER"  -> userHandler.handle(u);
}
```

---

## #GoldenHammer

**Smell:** Using the same tool for everything (e.g., `Optional` for flow control).

```java
Optional<Order> o = repo.find(id);
o.ifPresentOrElse(this::ship, () -> { throw new NotFound(); });
```

**Better:** Use exceptions for exceptional flows; simple returns for normal.

```java
Order o = repo.find(id).orElseThrow(NotFound::new);
ship(o);
```

---

## #LavaFlow

**Smell:** Dead code kept “just in case”.

```java
// legacy calculateDiscount - DO NOT DELETE
int calc(int price){ return price; } // never called
```

**Better:** Remove; add tests + version control to recover when needed.

---

## #PrematureOptimization

**Smell:** Micro‑optimizing before measuring.

```java
String s = "";
for (String p: parts) { s += p; } // naive concat for “speed”
```

**Better:** Write clear code, then profile.

```java
String result = String.join("", parts);
```

---

## #Magic Numbers / Strings

**Smell:** Unnamed literals.

```java
if (status == 3) process();
```

**Better:** Named constants / enums.

```java
enum Status { NEW, PROCESSING, DONE }
if (status == Status.DONE) process();
```

---

## #Feature #Envy

**Smell:** A method uses another class’s data more than its own.

```java
class InvoicePrinter {
    String print(Invoice i){
        return i.getCustomerName()+" owes "+i.total();
    }
}
```

**Better:** Move behavior to the data owner.

```java
class Invoice {
    String pretty(){ return customerName + " owes " + total(); }
}
```

---

## #ShotgunSurgery

**Smell:** One change requires edits in many places.

```java
// Discount logic scattered in cart, checkout, invoice...
```

**Better:** Centralize policy.

```java
class DiscountPolicy { Money apply(Cart c){ /* single place */ } }
```

---

## #ConstantInterface

**Smell:** Using interfaces to hold constants.

```java
interface AppConsts { String VERSION = "1.0"; }
```

**Better:** Use final class.

```java
final class AppConstants { private AppConstants(){} public static final String VERSION = "1.0"; }
```

---

## #SingletonOveruse

**Smell:** Global state hidden via Singleton.

```java
class Logger { private static final Logger I = new Logger(); private Logger(){}
    static Logger get(){ return I; }
}
```

**Better:** Prefer dependency injection (constructor injection).

```java
class Service { private final Logger log; Service(Logger log){ this.log = log; } }
```

---

## #NullCheckingEverywhere

**Smell:** Repetitive `if (x != null)` chains.

```java
if (user != null && user.getProfile() != null) {
    // ...
}
```

**Better:** Use Optional thoughtfully.

```java
userOpt.flatMap(User::profile).ifPresent(this::useProfile);
```

---

## #CopyPaste Programming

**Smell:** Duplicated logic across methods.

```java
void saveUser(User u){ /* validate, map, persist */ }
void saveAdmin(Admin a){ /* same steps copied */ }
```

**Better:** Extract common behavior.

```java
<T> void save(T entity, Mapper<T> m){ validate(entity); repo.save(m.map(entity)); }
```

---

## #BigBallofMud

**Smell:** No clear architecture; tight coupling.

**Better:** Adopt modules and boundaries (e.g., hexagonal architecture): domain, application, adapters.

---

## #SwallowingExceptions

**Smell:** Hiding errors.

```java
try { repo.save(o); } catch (Exception ignored) {}
```

**Better:** Handle or propagate with context.

```java
try { repo.save(o); } catch (SQLException e) { throw new OrderStoreException(id, e); }
```

---

## #PrimitiveObsession

**Smell:** Using primitives for domain concepts.

```java
void register(String email){ /* validate later */ }
```

**Better:** Value objects.

```java
record Email(String value){
    public Email { Objects.requireNonNull(value); /* validate format */ }
}
```

---

## #OverLogging / #UnderLogging

**Smell:** Either log everything (noise) or nothing (no trace).

**Better:** Log events with context and levels; avoid sensitive data.

```java
log.info("order.created id={} customer={}", order.id(), order.customerId());
```

---

## #ExcessiveStatics

**Smell:** Static methods everywhere blocking testability.

```java
class Tax { static Money calc(Cart c){ /* ... */ } }
```

**Better:** Inject strategies.

```java
interface TaxPolicy { Money apply(Cart c); }
class DefaultTax implements TaxPolicy { public Money apply(Cart c){ /* ... */ } }
```

---

## #ReflectionOveruse

**Smell:** Dynamic hacks where simple code works.

```java
Field f = obj.getClass().getDeclaredField(name); f.setAccessible(true);
```

**Better:** Prefer explicit APIs or mappers (e.g., MapStruct / records).

---

## #CircularDependencies

**Smell:** A depends on B; B depends on A.

**Better:** Introduce interfaces or events to break cycles.

```java
interface PaymentPort { void charge(Order o); }
class CheckoutService { private final PaymentPort pay; /* ... */ }
```

---

## #AnemicDomain Model

**Smell:** Entities with only data; logic elsewhere (procedural OO).

```java
class Order { List<Item> items; BigDecimal total; }
class OrderService { BigDecimal computeTotal(Order o){ /* ... */ } }
```

**Better:** Put invariants/behavior in the entity.

```java
class Order {
    private final List<Item> items = new ArrayList<>();
    BigDecimal total(){ return items.stream().map(Item::subtotal).reduce(BigDecimal.ZERO, BigDecimal::add); }
    void add(Item i){ items.add(i); }
}
```

---

