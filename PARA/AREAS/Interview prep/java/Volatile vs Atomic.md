---
area: java
tags:
  - java
  - concurrency
type: 
created: 2025-08-17 12:00
---

# Overview


## Overview

Both `volatile` and atomic classes solve concurrency issues but in different ways:

- **volatile**: Ensures visibility and prevents reordering
- **Atomic**: Provides lock-free thread-safe operations with compare-and-swap (CAS)

## Java #Volatile

### What it Does

- Guarantees visibility of changes across threads
- Prevents instruction reordering around volatile variables
- Makes reads/writes atomic for references and most primitives

### Limitations

- **No atomicity for compound operations** (read-modify-write)
- Cannot perform atomic increment, decrement, or conditional updates

```java
// Problem with volatile
private volatile int counter = 0;

public void increment() {
    counter++;  // NOT atomic! (read -> increment -> write)
}
```

## Java #Atomic Classes

### Available Classes

- `AtomicInteger`, `AtomicLong`, `AtomicBoolean`
- `AtomicReference<T>`, `AtomicReferenceArray<T>`
- `AtomicIntegerArray`, `AtomicLongArray`
- `AtomicStampedReference`, `AtomicMarkableReference`

### How They Work

- Use **Compare-And-Swap (CAS)** operations
- Hardware-level atomic operations
- Lock-free and non-blocking

## Key Differences

|Feature|volatile|Atomic Classes|
|---|---|---|
|**Visibility**|✅ Guaranteed|✅ Guaranteed|
|**Atomicity**|❌ Single operations only|✅ Complex operations|
|**Compound Operations**|❌ Not supported|✅ Supported|
|**Performance**|🟡 Lightweight|🟡 CAS overhead|
|**Blocking**|✅ Non-blocking|✅ Non-blocking|
|**Memory Overhead**|✅ No extra memory|🟡 Slight overhead|

## Performance Comparison

### volatile Performance

```java
// Fast - direct memory access
private volatile boolean flag = false;

public void setFlag() {
    flag = true;  // Simple write
}
```

### Atomic Performance

```java
// Slightly slower - CAS operations
private AtomicBoolean flag = new AtomicBoolean(false);

public void setFlag() {
    flag.set(true);  // CAS operation
}
```

**Performance Notes:**

- volatile: ~1-2ns overhead
- Atomic: ~10-20ns overhead (varies by operation)
- Both much faster than synchronized blocks

## Use Cases

### When to Use volatile

#### 1. Simple Flags/Status

```java
public class ServiceStatus {
    private volatile boolean running = false;
    
    public void start() { running = true; }
    public void stop() { running = false; }
    public boolean isRunning() { return running; }
}
```

#### 2. Publishing Immutable Objects

```java
public class ConfigManager {
    private volatile Configuration config;
    
    public void updateConfig(Configuration newConfig) {
        config = newConfig;  // Atomic reference assignment
    }
    
    public Configuration getConfig() { return config; }
}
```

#### 3. Double-Checked Locking

```java
public class Singleton {
    private static volatile Singleton instance;
    
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) {
                    instance = new Singleton();
                }
            }
        }
        return instance;
    }
}
```

### When to Use Atomic Classes

#### 1. Counters and Accumulators

```java
public class Statistics {
    private final AtomicLong requestCount = new AtomicLong(0);
    private final AtomicLong errorCount = new AtomicLong(0);
    
    public void recordRequest() {
        requestCount.incrementAndGet();
    }
    
    public void recordError() {
        errorCount.incrementAndGet();
    }
    
    public long getSuccessRate() {
        long requests = requestCount.get();
        long errors = errorCount.get();
        return requests > 0 ? ((requests - errors) * 100) / requests : 0;
    }
}
```

#### 2. Lock-Free Data Structures

```java
public class LockFreeStack<T> {
    private final AtomicReference<Node<T>> head = new AtomicReference<>();
    
    public void push(T item) {
        Node<T> newNode = new Node<>(item);
        Node<T> currentHead;
        do {
            currentHead = head.get();
            newNode.next = currentHead;
        } while (!head.compareAndSet(currentHead, newNode));
    }
    
    public T pop() {
        Node<T> currentHead;
        Node<T> newHead;
        do {
            currentHead = head.get();
            if (currentHead == null) return null;
            newHead = currentHead.next;
        } while (!head.compareAndSet(currentHead, newHead));
        return currentHead.item;
    }
    
    private static class Node<T> {
        T item;
        Node<T> next;
        Node(T item) { this.item = item; }
    }
}
```

#### 3. Conditional Updates

```java
public class PriceManager {
    private final AtomicReference<BigDecimal> currentPrice = 
        new AtomicReference<>(BigDecimal.ZERO);
    
    public boolean updatePriceIfHigher(BigDecimal newPrice) {
        return currentPrice.updateAndGet(current -> 
            newPrice.compareTo(current) > 0 ? newPrice : current
        ).equals(newPrice);
    }
    
    public boolean setPriceIfExpected(BigDecimal expected, BigDecimal newPrice) {
        return currentPrice.compareAndSet(expected, newPrice);
    }
}
```

## Code Examples

### Problem: Race Condition

```java
// ❌ BROKEN - Race condition
class BrokenCounter {
    private volatile int count = 0;
    
    public void increment() {
        count++;  // Read-modify-write is NOT atomic
    }
}

// ✅ FIXED with Atomic
class SafeCounter {
    private final AtomicInteger count = new AtomicInteger(0);
    
    public void increment() {
        count.incrementAndGet();  // Atomic operation
    }
    
    public int get() {
        return count.get();
    }
}
```

### Advanced: Copy-on-Write with Atomic

```java
public class CopyOnWriteList<T> {
    private final AtomicReference<List<T>> list = 
        new AtomicReference<>(new ArrayList<>());
    
    public void add(T item) {
        List<T> current, updated;
        do {
            current = list.get();
            updated = new ArrayList<>(current);
            updated.add(item);
        } while (!list.compareAndSet(current, updated));
    }
    
    public List<T> getSnapshot() {
        return new ArrayList<>(list.get());
    }
}
```

### Memory Visibility Example

```java
public class VisibilityDemo {
    // volatile ensures changes are visible across threads
    private volatile boolean shutdownRequested = false;
    private final AtomicInteger taskCount = new AtomicInteger(0);
    
    public void requestShutdown() {
        shutdownRequested = true;  // Visible immediately
    }
    
    public void workerThread() {
        while (!shutdownRequested) {
            processTask();
            taskCount.incrementAndGet();  // Thread-safe increment
        }
    }
    
    private void processTask() {
        // Simulate work
        try { Thread.sleep(100); } catch (InterruptedException e) {}
    }
}
```

## Best Practices

### Choosing Between volatile and Atomic

**Use volatile for:**

- Simple boolean flags
- Publishing immutable objects
- Single variable updates
- Status indicators

**Use Atomic for:**

- Counters and metrics
- Lock-free algorithms
- Conditional updates
- Complex state management

### Performance Tips

1. **Prefer volatile for simple cases** - lower overhead
2. **Use atomic for compound operations** - avoid race conditions
3. **Consider batching** - group operations when possible
4. **Monitor contention** - high contention may require different approaches

### Common Patterns

```java
// Pattern 1: Shutdown flag
private volatile boolean shutdown = false;

// Pattern 2: Reference publishing
private volatile Configuration config;

// Pattern 3: Atomic counter
private final AtomicLong counter = new AtomicLong();

// Pattern 4: Conditional atomic update
atomicRef.updateAndGet(current -> computeNewValue(current));
```

### Anti-Patterns to Avoid

```java
// ❌ Don't use volatile for compound operations
volatile int count;
count++;  // Race condition!

// ❌ Don't use atomic for simple flags (overkill)
AtomicBoolean flag = new AtomicBoolean();  // volatile boolean is better

// ❌ Don't mix volatile and synchronized on same variable
volatile int value;
synchronized(this) { value++; }  // Confusing and unnecessary
```