---
area: java
tags:
  - java
  - memory-management
type: 
created: 2025-08-17 11:58
---

# Overview

## #Stack Memory

### Root Structure

```c++
// JVM Stack implementation (simplified)
struct JVMStack {
    StackFrame* frames;         // Array of stack frames
    StackFrame* currentFrame;   // Currently executing frame
    size_t maxSize;            // Maximum stack size (-Xss)
    size_t currentSize;        // Current stack usage
    Thread* ownerThread;       // Thread that owns this stack
};

// Individual Stack Frame
struct StackFrame {
    LocalVariableArray localVars;  // Method parameters + local variables
    OperandStack operands;          // Calculation stack for operations
    FrameData frameData;            // Method metadata and return info
    StackFrame* previous;           // Previous frame in call chain
    Method* method;                 // Method being executed
};

// Local Variable Array
struct LocalVariableArray {
    Slot* variables;               // Array of variable slots
    int maxLocals;                // Maximum local variables
    int currentIndex;             // Current variable index
};

// Operand Stack  
struct OperandStack {
    Slot* stack;                  // Array for operands
    int maxStack;                 // Maximum stack depth
    int top;                      // Top of stack pointer
};

// Variable Slot (can hold different types)
union Slot {
    int intValue;
    long longValue;
    float floatValue;
    double doubleValue;
    ObjectReference objRef;
    ReturnAddress returnAddr;
};
```

### Basic Structure

- **Thread-specific**: Each thread has its own stack
- **LIFO (Last In, First Out)**: Stack frames pushed/popped during method calls
- **Fixed size**: Set at thread creation (default 512KB-1MB)
- **Fast access**: Direct memory addressing, no GC overhead
- **Automatic cleanup**: Frames automatically removed when methods return

### Memory Layout

```
Stack Memory Layout (grows downward):
┌─────────────────────────────────┐ ← Stack Base (High Address)
│         Stack Frame N           │
├─────────────────────────────────┤
│       Local Variables           │   int a = 10;
│       [0] this reference        │   String s = "hello";
│       [1] parameter 1           │
│       [2] local variable 1      │
├─────────────────────────────────┤
│        Operand Stack           │   
│       [Top] → value 3          │   Calculation stack
│              value 2           │   for bytecode operations
│              value 1           │
├─────────────────────────────────┤
│        Frame Data              │
│       - Return Address         │   Where to return after method
│       - Exception Table        │   Exception handling info
│       - Method Reference       │   Current method metadata
├─────────────────────────────────┤
│         Stack Frame N-1        │
├─────────────────────────────────┤
│              ...               │
├─────────────────────────────────┤
│         Stack Frame 1          │   main() method frame
└─────────────────────────────────┘ ← Stack Pointer (Low Address)
```

### Core Operations

#### Method Call (Push Frame):

```java
// Java method call
public void methodA() {
    int x = 5;
    methodB(x, "test");
}

public void methodB(int param1, String param2) {
    int localVar = param1 * 2;
    // Method execution
}
```

```c++
// Low-level stack operations
void pushStackFrame(Method* method, Object* args[]) {
    // 1. Calculate frame size
    size_t frameSize = calculateFrameSize(method);
    
    // 2. Check stack overflow
    if (currentSize + frameSize > maxSize) {
        throw StackOverflowError();
    }
    
    // 3. Create new frame
    StackFrame* newFrame = allocateFrame(frameSize);
    newFrame->method = method;
    newFrame->previous = currentFrame;
    
    // 4. Initialize local variables
    initializeLocalVariables(newFrame, args);
    
    // 5. Initialize operand stack
    initializeOperandStack(newFrame);
    
    // 6. Update current frame
    currentFrame = newFrame;
    currentSize += frameSize;
}
```

#### Method Return (Pop Frame):

```c++
void popStackFrame() {
    if (currentFrame == NULL) return;
    
    // 1. Get return value from operand stack
    Slot returnValue = currentFrame->operands.pop();
    
    // 2. Store previous frame reference
    StackFrame* previousFrame = currentFrame->previous;
    
    // 3. Deallocate current frame
    deallocateFrame(currentFrame);
    currentSize -= currentFrame->size;
    
    // 4. Restore previous frame
    currentFrame = previousFrame;
    
    // 5. Push return value to caller's operand stack
    if (previousFrame != NULL) {
        previousFrame->operands.push(returnValue);
    }
}
```

#### Local Variable Access:

```c++
// Bytecode: iload_1 (load int from local variable 1)
void iload(int index) {
    Slot value = currentFrame->localVars.variables[index];
    currentFrame->operands.push(value);
}

// Bytecode: istore_2 (store int to local variable 2)  
void istore(int index) {
    Slot value = currentFrame->operands.pop();
    currentFrame->localVars.variables[index] = value;
}
```

### Memory Characteristics

#### Size and Limits:

```bash
# Stack size configuration
-Xss512k     # 512KB stack per thread
-Xss1m       # 1MB stack per thread (default)
-Xss2m       # 2MB stack per thread

# Memory calculation
Stack Memory Per Thread = Frame Size × Max Call Depth
Typical Frame Size = 200-1000 bytes
Max Call Depth = Stack Size / Average Frame Size
```

#### Performance Characteristics:

```
Stack Operations:
├── Push Frame         O(1) - Constant time
├── Pop Frame          O(1) - Constant time  
├── Local Variable     O(1) - Direct array access
├── Operand Stack      O(1) - Array operations
└── Exception Unwind   O(n) - Pop multiple frames
```

### Stack Frame Examples

#### Simple Method Call:

```java
public class StackExample {
    public static void main(String[] args) {    // Frame 1
        int result = calculate(5, 10);          
    }
    
    public static int calculate(int a, int b) { // Frame 2
        int sum = a + b;                        // Local variable
        return sum;
    }
}
```

```
Stack State During Execution:

Frame 2 (calculate method):
┌─────────────────────────────────┐
│ Local Variables:                │
│ [0] a = 5                      │
│ [1] b = 10                     │  
│ [2] sum = 15                   │
├─────────────────────────────────┤
│ Operand Stack:                 │
│ [Top] → 15 (return value)      │
├─────────────────────────────────┤
│ Frame Data:                    │
│ - Return to main() line 3      │
│ - Method: calculate()          │
└─────────────────────────────────┘

Frame 1 (main method):
┌─────────────────────────────────┐
│ Local Variables:                │
│ [0] args = String[] reference   │
│ [1] result = (uninitialized)   │
├─────────────────────────────────┤
│ Operand Stack:                 │
│ (empty, waiting for return)    │
├─────────────────────────────────┤
│ Frame Data:                    │
│ - Return to JVM                │
│ - Method: main()               │
└─────────────────────────────────┘
```

### Recursive Call Stack:

```java
public int factorial(int n) {
    if (n <= 1) return 1;           // Base case
    return n * factorial(n - 1);    // Recursive call
}

// factorial(5) call sequence
```

```
Stack Growth During Recursion:

factorial(5) - Frame 5:
┌─────────────────────────────────┐
│ [0] n = 5                      │
│ Return: 5 * factorial(4)       │
└─────────────────────────────────┘

factorial(4) - Frame 4:
┌─────────────────────────────────┐
│ [0] n = 4                      │
│ Return: 4 * factorial(3)       │
└─────────────────────────────────┘

factorial(3) - Frame 3:
┌─────────────────────────────────┐
│ [0] n = 3                      │
│ Return: 3 * factorial(2)       │
└─────────────────────────────────┘

factorial(2) - Frame 2:
┌─────────────────────────────────┐
│ [0] n = 2                      │
│ Return: 2 * factorial(1)       │
└─────────────────────────────────┘

factorial(1) - Frame 1:
┌─────────────────────────────────┐
│ [0] n = 1                      │
│ Return: 1 (base case)          │
└─────────────────────────────────┘

Stack unwinding: 1 → 2 → 6 → 24 → 120
```

---

## #Heap Memory

### Root Structure

```c++
// Heap Memory Structure
struct HeapMemory {
    YoungGeneration* youngGen;      // New objects
    OldGeneration* oldGen;          // Long-lived objects  
    MetaspaceArea* metaspace;       // Class metadata (Java 8+)
    DirectMemory* directMem;        // Off-heap memory
    
    size_t totalSize;               // Total heap size
    size_t usedSize;               // Currently used memory
    float gcThreshold;             // GC trigger threshold
    GarbageCollector* gc;          // GC implementation
};

// Young Generation Layout
struct YoungGeneration {
    EdenSpace* eden;               // 80% - New object allocation
    SurvivorSpace* survivor0;      // 10% - GC survivor space
    SurvivorSpace* survivor1;      // 10% - GC survivor space
    
    size_t totalSize;              // Young gen total size
    int minorGCCount;             // Number of minor GCs
};

// Old Generation
struct OldGeneration {
    TenuredSpace* tenured;         // Long-lived objects
    size_t totalSize;              // Old gen total size
    int majorGCCount;             // Number of major GCs
    float occupancyThreshold;     // Trigger for major GC
};

// Object Header (every object in heap)
struct ObjectHeader {
    MarkWord markWord;            // 8 bytes - GC/locking info
    ClassPointer klassOop;        // 8 bytes - Class metadata pointer
    // Object data follows...
};

// Mark Word bit layout (64-bit JVM)
struct MarkWord {
    unsigned long hash : 25;       // Identity hash code
    unsigned long age : 4;         // GC generation age
    unsigned long biased : 1;      // Biased locking flag
    unsigned long lock : 2;        // Lock state
    unsigned long unused : 32;     // Unused bits
};
```

### Basic Structure

- **Shared memory**: All threads share the same heap
- **Object storage**: All objects and instance variables stored here
- **Garbage collected**: Automatic memory management
- **Variable size**: Can grow/shrink during runtime (within limits)
- **Generational**: Divided into young and old generations

### Memory Layout

```
Heap Memory Layout:
┌─────────────────────────────────────────────────────────────┐
│                    HEAP MEMORY                              │
├─────────────────────────────────────────────────────────────┤
│                YOUNG GENERATION (1/3 of heap)              │
│  ┌─────────────────┬─────────────┬─────────────────────────┐│
│  │   EDEN SPACE    │ SURVIVOR 0  │     SURVIVOR 1         ││
│  │     (80%)       │    (10%)    │        (10%)           ││
│  │                 │             │                        ││
│  │ [New Objects]   │[GC Survivors│   [GC Survivors]       ││
│  │ Object A ────── │ Object D ───│                        ││
│  │ Object B        │ Object E    │                        ││
│  │ Object C        │             │                        ││
│  └─────────────────┴─────────────┴─────────────────────────┘│
├─────────────────────────────────────────────────────────────┤
│                OLD GENERATION (2/3 of heap)                │
│  ┌─────────────────────────────────────────────────────────┐│
│  │              TENURED SPACE                              ││
│  │                                                         ││
│  │ [Long-lived Objects]                                    ││
│  │ Object X (age 15) ──────────────────────────            ││
│  │ Object Y (age 10) ────────────                          ││
│  │ Object Z (age 8) ──────                                 ││
│  │                                                         ││
│  └─────────────────────────────────────────────────────────┘│
├─────────────────────────────────────────────────────────────┤
│                 METASPACE (Native Memory)                  │
│  ┌─────────────────────────────────────────────────────────┐│
│  │ [Class Metadata, Constant Pool, Method Bytecode]       ││
│  │ Class MyClass                                           ││
│  │ ├── Method info                                         ││
│  │ ├── Field info                                          ││
│  │ └── Constant pool                                       ││
│  └─────────────────────────────────────────────────────────┘│
└─────────────────────────────────────────────────────────────┘
```

### Object Allocation Process

#### Fast Path Allocation (Eden):

```c++
Object* allocateObject(Class* clazz, size_t objectSize) {
    // 1. Try fast allocation in Eden space
    Object* obj = eden->allocate(objectSize);
    if (obj != NULL) {
        // 2. Initialize object header
        initializeObjectHeader(obj, clazz);
        return obj;
    }
    
    // 3. Eden is full, trigger minor GC
    minorGC();
    
    // 4. Try allocation again after GC
    obj = eden->allocate(objectSize);
    if (obj != NULL) {
        initializeObjectHeader(obj, clazz);
        return obj;
    }
    
    // 5. Still can't allocate, try old generation
    return allocateInOldGeneration(clazz, objectSize);
}

// Thread-Local Allocation Buffers (TLAB)
struct TLAB {
    char* start;                  // Start of TLAB
    char* current;               // Current allocation pointer
    char* end;                   // End of TLAB
    Thread* owner;               // Owning thread
};

Object* tlabAllocate(size_t size) {
    // Fast, lock-free allocation within TLAB
    if (current + size <= end) {
        Object* obj = (Object*)current;
        current += size;
        return obj;
    }
    // TLAB exhausted, get new TLAB or fall back to shared allocation
    return allocateSlowPath(size);
}
```

#### Object Layout in Memory:

```c++
// Example: String object layout
class String {
    // Object Header (16 bytes on 64-bit JVM)
    MarkWord markWord;        // 8 bytes
    ClassPointer klass;       // 8 bytes (or 4 with compressed OOPs)
    
    // Instance fields
    char[] value;             // 8 bytes - reference to char array
    int hash;                 // 4 bytes - cached hash code
    // Padding to 8-byte alignment
};

// Memory layout example
String object at address 0x7f8a1c001000:
┌─────────────────────────────────────┐
│ 0x7f8a1c001000: Mark Word           │ 8 bytes
├─────────────────────────────────────┤  
│ 0x7f8a1c001008: Class Pointer       │ 8 bytes
├─────────────────────────────────────┤
│ 0x7f8a1c001010: value (char[] ref)  │ 8 bytes
├─────────────────────────────────────┤
│ 0x7f8a1c001018: hash (int)          │ 4 bytes
├─────────────────────────────────────┤
│ 0x7f8a1c00101c: Padding             │ 4 bytes
└─────────────────────────────────────┘
Total object size: 32 bytes
```

### Garbage Collection Process

#### Minor GC (Young Generation):

```c++
void minorGC() {
    // 1. Mark all reachable objects in young generation
    markReachableObjects(youngGen);
    
    // 2. Copy live objects from Eden to Survivor space
    for (Object* obj : eden->liveObjects) {
        if (obj->age < maxAge) {
            copyToSurvivor(obj);
            obj->age++;
        } else {
            promoteToOldGen(obj);  // Promotion to old generation
        }
    }
    
    // 3. Swap survivor spaces (from-space ↔ to-space)
    swapSurvivorSpaces();
    
    // 4. Clear Eden space
    eden->clear();
    
    // 5. Update GC statistics
    youngGen->minorGCCount++;
}
```

#### Major GC (Full Heap):

```c++
void majorGC() {
    // 1. Stop all application threads (Stop-the-World)
    stopAllThreads();
    
    // 2. Mark phase - find all reachable objects
    markReachableObjects(heap);
    
    // 3. Sweep phase - deallocate unreachable objects
    sweepUnreachableObjects(heap);
    
    // 4. Compact phase - defragment memory (optional)
    compactHeap(heap);
    
    // 5. Resume application threads
    resumeAllThreads();
    
    oldGen->majorGCCount++;
}
```

#### Object Aging and Promotion:

```java
// Object lifecycle example
String str = new String("Hello");  // Allocated in Eden

// After minor GC #1: 
// - Object survives, moved to Survivor 0, age = 1

// After minor GC #2:
// - Object survives, moved to Survivor 1, age = 2

// ... (continues until age reaches threshold)

// After minor GC #15 (assuming max age = 15):
// - Object promoted to Old Generation
```

```c++
// Promotion logic
void checkPromotion(Object* obj) {
    const int PROMOTION_AGE_THRESHOLD = 15;
    
    if (obj->age >= PROMOTION_AGE_THRESHOLD) {
        promoteToOldGeneration(obj);
    } else if (survivorSpaceUtilization > 50%) {
        // Early promotion due to space pressure
        promoteToOldGeneration(obj);
    } else {
        moveToNextSurvivorSpace(obj);
        obj->age++;
    }
}
```

### Memory Management Strategies

#### Generational Hypothesis:

```
Object Lifetime Distribution:
├── 90% of objects die young (within few GC cycles)
├── Objects that survive become long-lived
├── Young generation GC is frequent but fast
└── Old generation GC is infrequent but slow

Allocation Patterns:
├── Eden Space:     Fast allocation, frequent collection
├── Survivor Space: Temporary storage during GC
├── Old Generation: Stable, infrequent collection
└── Metaspace:     Class metadata, rarely collected
```

#### TLAB (Thread-Local Allocation Buffers):

```c++
// Lock-free allocation per thread
struct ThreadAllocation {
    TLAB tlab;                    // Thread's private allocation buffer
    size_t tlabSize;             // Current TLAB size
    size_t tlabRefills;          // Number of TLAB refills
    
    Object* allocate(size_t size) {
        // Fast path: allocate from TLAB
        if (tlab.current + size <= tlab.end) {
            Object* obj = (Object*)tlab.current;
            tlab.current += size;
            return obj;
        }
        
        // Slow path: refill TLAB or shared allocation
        return refillTLABAndAllocate(size);
    }
};

// TLAB statistics
TLAB Performance:
├── Allocation Speed:     ~10-50x faster than synchronized
├── Cache Locality:       Better CPU cache utilization  
├── Allocation Rate:      ~1-10 GB/sec per thread
└── GC Pressure:         Reduced allocation contention
```

### Memory Monitoring and Tuning

#### Heap Size Configuration:

```bash
# Heap size settings
-Xms2g           # Initial heap size (2GB)
-Xmx8g           # Maximum heap size (8GB)  
-XX:NewRatio=3   # Old/Young generation ratio (3:1)
-XX:NewSize=1g   # Initial young generation size
-XX:MaxNewSize=2g # Maximum young generation size

# TLAB settings
-XX:TLABSize=256k        # TLAB size per thread
-XX:+ResizeTLAB          # Dynamic TLAB sizing
-XX:TLABRefillWasteFraction=64  # TLAB waste threshold
```

#### GC Tuning Parameters:

```bash
# Generational settings
-XX:MaxTenuringThreshold=15      # Object promotion age
-XX:TargetSurvivorRatio=50       # Survivor space utilization target
-XX:SurvivorRatio=8              # Eden/Survivor ratio (8:1:1)

# GC algorithm selection
-XX:+UseG1GC                     # G1 garbage collector
-XX:MaxGCPauseMillis=200         # Target pause time (200ms)
-XX:G1HeapRegionSize=16m         # G1 region size

# Parallel GC settings  
-XX:+UseParallelGC               # Parallel garbage collector
-XX:ParallelGCThreads=8          # GC thread count
```

### Performance Characteristics

#### Allocation Performance:

```
Memory Allocation Speed:
├── TLAB Allocation:      0.1-1 nanoseconds
├── Shared Eden:          10-100 nanoseconds
├── Old Generation:       100-1000 nanoseconds
└── Off-heap (Direct):    50-500 nanoseconds

Throughput (objects/second):
├── Small objects (<100 bytes):   10-100 million/sec
├── Medium objects (1KB):         1-10 million/sec  
├── Large objects (>10KB):        100K-1M/sec
└── Huge objects (>1MB):          1K-100K/sec
```

#### GC Performance Impact:

```
Garbage Collection Pause Times:
├── Minor GC (Young Gen):
│   ├── Serial GC:        1-10ms
│   ├── Parallel GC:      1-50ms
│   └── G1 GC:           1-10ms
├── Major GC (Full Heap):
│   ├── Serial GC:        100ms-10s
│   ├── Parallel GC:      10ms-1s
│   └── G1 GC:           10-200ms
└── Concurrent GC:        1-10ms (low-latency collectors)

Application Impact:
├── Allocation Rate:      1-10 GB/sec (typical application)
├── GC Overhead:         5-15% (well-tuned application)
├── Memory Fragmentation: <5% (with compacting GC)
└── Cache Misses:        Reduced with generational collection
```

## #Stack vs #Heap Comparison

### Detailed Comparison Table:

|Aspect|Stack Memory|Heap Memory|
|---|---|---|
|**Storage**|Local variables, method parameters|Objects, instance variables, arrays|
|**Access Speed**|Very fast (CPU registers/L1 cache)|Slower (main memory access)|
|**Size**|Small (512KB-1MB per thread)|Large (GB range, configurable)|
|**Memory Management**|Automatic (LIFO)|Garbage collection|
|**Thread Safety**|Thread-local (isolated)|Shared (synchronization needed)|
|**Allocation**|O(1) - move stack pointer|O(1) fast path, O(log n) slow path|
|**Deallocation**|O(1) - automatic on return|O(n) - garbage collection|
|**Memory Layout**|Contiguous, organized|Fragmented, objects scattered|
|**Lifetime**|Method duration|Until garbage collected|
|**Error Conditions**|StackOverflowError|OutOfMemoryError|

### Memory Access Patterns:

```java
public class MemoryExample {
    // Stored in heap (class-level)
    private static String globalString = "Global";
    private List<Integer> numbers = new ArrayList<>();
    
    public void demonstrateMemory() {
        // Stack variables
        int localInt = 42;              // Stack: primitive value
        String localString = "Local";   // Stack: reference, Heap: object
        Object[] array = new Object[10]; // Stack: reference, Heap: array
        
        // All object creations go to heap
        StringBuilder sb = new StringBuilder(); // Heap: object + internal char[]
        numbers.add(localInt);          // Heap: ArrayList expansion, Integer boxing
        
        // Method call creates new stack frame
        processData(localInt, localString);
        
        // Stack frame automatically cleaned up on return
        // Heap objects remain until GC
    }
    
    private void processData(int value, String text) {
        // New stack frame with its own local variables
        char[] chars = text.toCharArray(); // Heap: new char array
        // Stack frame destroyed on method return
    }
}
```

This comprehensive documentation covers the low-level implementation details of both stack and heap memory, providing insights into allocation strategies, garbage collection, performance characteristics, and tuning parameters essential for Java performance optimization.