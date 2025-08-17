---
area: java
tags:
  - java
  - internals
type: 
created: 2025-08-17 11:59
---

# Overview


## #JRE (Java Runtime Environment)

### Structure Overview

```
JRE
├── JVM (Java Virtual Machine)
│   ├── Class Loader Subsystem
│   ├── Runtime Data Areas
│   └── Execution Engine
├── Java Standard Library (rt.jar)
├── Native Method Libraries
└── Configuration Files
```

### Core Components

- **JVM**: The actual runtime that executes Java bytecode
- **Java Class Libraries**: Standard APIs (java.lang, java.util, etc.)
- **Native Libraries**: Platform-specific code (JNI libraries)
- **Configuration Files**: Properties and settings

### Low-Level Details

- **Size**: ~200-300 MB (varies by version and platform)
- **Key Files**:
    - `rt.jar` (Runtime JAR) - core classes
    - `jvm.dll/.so` - JVM implementation
    - Native libraries for OS interaction

### Memory Footprint

```
Typical JRE Installation:
├── bin/           (~50 MB)  - Executables and DLLs
├── lib/           (~150 MB) - JAR files and native libraries
│   ├── rt.jar     (~65 MB)  - Core runtime classes
│   ├── tools.jar  (~15 MB)  - Development tools
│   └── ext/       (~10 MB)  - Extension libraries
└── conf/          (~1 MB)   - Configuration files
```

---

## #JDK (Java Development Kit)

### Structure Overview

```
JDK
├── JRE (Complete Runtime Environment)
├── Development Tools
│   ├── javac (Compiler)
│   ├── jar (Archive Tool)
│   ├── javadoc (Documentation Generator)
│   └── debugger tools
├── Header Files (for JNI)
├── Source Code (src.zip)
└── Development Libraries
```

### Development Tools Breakdown

#### Core Compilation Tools:

```bash
# javac - Java Compiler
javac -cp <classpath> -d <output> *.java
# Converts .java → .class (bytecode)

# jar - Java Archive Tool  
jar cf myapp.jar *.class
# Packages classes into JAR files

# javap - Class File Disassembler
javap -c MyClass
# Shows bytecode of compiled classes
```

#### Debugging & Profiling Tools:

```bash
# jdb - Java Debugger
jdb -classpath . MyProgram

# jstack - Thread Stack Trace
jstack <pid>

# jmap - Memory Map
jmap -dump:format=b,file=heap.hprof <pid>

# jstat - JVM Statistics
jstat -gc <pid> 1s
```

### Low-Level File Structure

```
JDK Directory Layout:
├── bin/           (~100 MB) - All executables
│   ├── javac.exe           - Java compiler
│   ├── java.exe            - Java launcher  
│   ├── jar.exe             - Archive tool
│   ├── javadoc.exe         - Documentation tool
│   └── jconsole.exe        - Monitoring tool
├── include/       (~2 MB)   - JNI header files
│   ├── jni.h               - Main JNI header
│   └── platform-specific/ - OS-specific headers
├── lib/           (~200 MB) - Libraries and tools
│   ├── tools.jar           - Development tools
│   └── dt.jar              - Design time libraries
├── jre/           (~300 MB) - Complete JRE
└── src.zip        (~50 MB)  - Java source code
```

### Memory Requirements

- **Installation Size**: 300-500 MB
- **Runtime Memory**: Additional 50-100 MB for development tools
- **Compilation Memory**: javac can use 100-500 MB for large projects

---

## #JVM (Java Virtual Machine)

### Architecture Overview

```
JVM Architecture:
┌─────────────────────────────────────────┐
│              JVM Process                │
├─────────────────────────────────────────┤
│          Class Loader Subsystem         │
│  ┌─────────┬─────────┬─────────────────┐│
│  │Bootstrap│Extension│Application      ││
│  │Loader   │Loader   │Class Loader     ││
│  └─────────┴─────────┴─────────────────┘│
├─────────────────────────────────────────┤
│           Runtime Data Areas            │
│  ┌─────────────────┬──────────────────┐ │
│  │   Per-JVM       │    Per-Thread    │ │
│  │  ┌────────────┐ │ ┌──────────────┐ │ │
│  │  │Method Area │ │ │ProgramCounter│ │ │
│  │  │Heap Memory │ │ │JVM Stack     │ │ │
│  │  │Direct Mem. │ │ │NativeMethod  │ │ │
│  │  └────────────┘ │ │Stack         │ │ │
│  └─────────────────┴─┴──────────────┴─┘ │
├─────────────────────────────────────────┤
│            Execution Engine             │
│  ┌─────────┬─────────┬─────────────────┐│
│  │Interpreter│JIT    │Garbage          ││
│  │          │Compiler│Collector        ││
│  └─────────┴─────────┴─────────────────┘│
└─────────────────────────────────────────┘
```

### Class Loader Subsystem

#### Loading Process:

```java
// 1. Loading - Find and load .class files
ClassLoader loader = MyClass.class.getClassLoader();

// 2. Linking - Verification, Preparation, Resolution
//    - Verification: Bytecode verification
//    - Preparation: Memory allocation for static variables  
//    - Resolution: Symbolic references → Direct references

// 3. Initialization - Execute static initializers
```

#### Class Loader Hierarchy:

```
Bootstrap Class Loader (C++)
├── Loads core JVM classes (rt.jar)
├── No parent loader
└── Platform-specific implementation

Extension Class Loader (Java)  
├── Parent: Bootstrap
├── Loads extension libraries (lib/ext/)
└── URLClassLoader implementation

Application Class Loader (Java)
├── Parent: Extension  
├── Loads application classes (CLASSPATH)
└── Also called System Class Loader
```

### Runtime Data Areas

#### Method Area (Metaspace in Java 8+):

```c++
// Low-level structure (conceptual)
struct MethodArea {
    ClassMetadata* classes;      // Class information
    ConstantPool* constantPools; // String/numeric constants
    MethodInfo* methods;         // Method bytecode
    FieldInfo* fields;          // Field information
    RuntimeConstantPool* rtcp;   // Runtime constant pool
};

// Memory layout
MethodArea Memory:
├── Class Metadata        (~10-50 KB per class)
├── Constant Pool Data    (~5-20 KB per class)  
├── Method Bytecode       (~1-10 KB per method)
└── JIT Compiled Code     (~5-50 KB per method)
```

#### #HeapMemoryStructure:

```c++
// Generational Heap Layout
struct HeapMemory {
    YoungGeneration youngGen;
    OldGeneration oldGen;
    PermanentGeneration permGen; // Java 7 and below
};

// Young Generation (33% of heap)
struct YoungGeneration {
    EdenSpace eden;       // 80% of young gen
    SurvivorSpace s0;     // 10% of young gen  
    SurvivorSpace s1;     // 10% of young gen
};

// Memory sizes (typical)
Heap Layout:
├── Young Generation    (~1/3 of heap)
│   ├── Eden Space     (~80% of young gen)
│   ├── Survivor S0    (~10% of young gen)
│   └── Survivor S1    (~10% of young gen)
├── Old Generation     (~2/3 of heap)
└── Metaspace          (Native memory, unlimited)
```

#### Thread-Specific Areas:

```c++
// Per-thread memory structure
struct ThreadArea {
    ProgramCounter pc;           // Current instruction pointer
    JVMStack stack;             // Method call frames
    NativeMethodStack nativeStack; // JNI method calls
};

// JVM Stack Frame
struct StackFrame {
    LocalVariableArray localVars;  // Method parameters & local vars
    OperandStack operands;          // Calculation stack
    FrameData frameData;            // Return address, exception info
};

// Memory sizes per thread
Thread Memory:
├── JVM Stack          (512 KB - 1 MB default)
├── Native Stack       (128 KB - 1 MB default)
└── Program Counter    (4-8 bytes)
```

### Execution Engine

#### Interpreter vs JIT Compilation:

```c++
// Execution flow
ExecutionEngine {
    // 1. Initial execution - Interpreted
    interpretBytecode(method);
    
    // 2. Hotspot detection (>10,000 invocations)
    if (method.invocationCount > threshold) {
        compileToNative(method);
    }
    
    // 3. Execute native code
    executeNativeCode(method);
}
```

#### #JIT Compiler Tiers:

```
Tiered Compilation:
├── Tier 0: Interpreter           (Immediate execution)
├── Tier 1: C1 Client Compiler    (Fast compilation, basic optimization)
├── Tier 2: C1 with profiling     (Collect runtime data)
├── Tier 3: C1 with full profiling (More detailed profiling)
└── Tier 4: C2 Server Compiler    (Aggressive optimization)

Performance progression:
Interpreter → C1 (3-5x faster) → C2 (10-50x faster)
```

#### Garbage Collection Algorithms:

##### #Serial #GC:

```c++
// Single-threaded collection
void serialGC() {
    stopAllThreads();           // Stop-the-world pause
    markReachableObjects();     // Mark phase
    sweepUnreachableObjects();  // Sweep phase
    compactHeap();             // Compact phase (optional)
    resumeAllThreads();
}
```

##### #Parallel #GC:

```c++
// Multi-threaded collection  
void parallelGC() {
    stopAllThreads();
    
    // Use multiple GC threads
    for (int i = 0; i < gcThreads; i++) {
        gcThread[i].markAndSweep(heapRegion[i]);
    }
    
    resumeAllThreads();
}
```

##### #G1 #GC (Low-latency):

```c++
// Incremental collection
void g1GC() {
    // Concurrent marking (with application)
    concurrentMark();
    
    // Short pause for collection
    stopAllThreads();
    evacuateRegions();         // Copy live objects
    resumeAllThreads();
    
    // Continue concurrent cleanup
    concurrentCleanup();
}
```

### Memory Management

#### Object Allocation:

```c++
// Object creation process
Object* allocateObject(Class* clazz) {
    size_t objectSize = calculateObjectSize(clazz);
    
    // 1. Try Eden space (fast path)
    Object* obj = edenSpace.allocate(objectSize);
    if (obj != NULL) return obj;
    
    // 2. Trigger minor GC
    minorGC();
    obj = edenSpace.allocate(objectSize);
    if (obj != NULL) return obj;
    
    // 3. Try old generation
    obj = oldGeneration.allocate(objectSize);
    if (obj != NULL) return obj;
    
    // 4. Trigger major GC
    majorGC();
    obj = oldGeneration.allocate(objectSize);
    if (obj != NULL) return obj;
    
    // 5. OutOfMemoryError
    throw OutOfMemoryError();
}
```

#### Object Header Structure:

```c++
// 64-bit JVM object layout
struct ObjectHeader {
    markWord mark;      // 8 bytes - GC info, hash, lock
    klassOop klass;     // 8 bytes - Class metadata pointer
    
    // Compressed OOPs (when enabled)
    // klassOop becomes 4 bytes, saving memory
};

// Mark word bit layout (64-bit)
Mark Word (64 bits):
├── Hash Code      (25 bits)
├── Age            (4 bits)  - GC generation count
├── Biased Lock    (1 bit)
├── Lock State     (2 bits)  - Unlocked/Locked/Marked for GC
└── Unused         (32 bits)
```

### JVM Startup Process

#### Initialization Sequence:

```c++
// JVM startup (simplified)
int startJVM(char* mainClass) {
    // 1. Load JVM library
    loadJVMLibrary();
    
    // 2. Create JVM instance  
    JavaVM* jvm = createJVM();
    
    // 3. Initialize class loaders
    initializeClassLoaders();
    
    // 4. Load system classes
    loadSystemClasses();        // Object, String, Class, etc.
    
    // 5. Initialize heap
    initializeHeap();
    
    // 6. Start GC threads
    startGarbageCollector();
    
    // 7. Load main class
    Class* mainClazz = loadClass(mainClass);
    
    // 8. Find main method
    Method* mainMethod = findMainMethod(mainClazz);
    
    // 9. Execute main method
    executeMethod(mainMethod);
    
    return 0;
}
```

### Platform-Specific Implementation

#### JVM Implementations:

```
HotSpot JVM (Oracle/OpenJDK):
├── C++ implementation (~2M lines of code)
├── Platform-specific assembly optimizations
├── Adaptive optimization with runtime profiling
└── Most widely used implementation

OpenJ9 (Eclipse/IBM):
├── Focuses on startup time and memory footprint
├── Ahead-of-time compilation support
├── Class data sharing
└── Better for cloud/container environments

GraalVM:
├── Polyglot runtime (Java, JS, Python, etc.)
├── Native image compilation
├── Aggressive optimizations
└── Experimental/research-oriented
```

#### Native Method Interface (JNI):

```c++
// JNI function signature
JNIEXPORT jstring JNICALL
Java_MyClass_nativeMethod(JNIEnv *env, jobject obj, jstring input) {
    // Convert Java string to C string
    const char* nativeString = (*env)->GetStringUTFChars(env, input, 0);
    
    // Perform native operations
    char* result = processString(nativeString);
    
    // Release memory
    (*env)->ReleaseStringUTFChars(env, input, nativeString);
    
    // Return new Java string
    return (*env)->NewStringUTF(env, result);
}

// Memory bridge between Java and native
JNI Memory Management:
├── Java Heap ←→ Native Heap copying
├── Direct ByteBuffers (shared memory)
├── Critical sections (temporary pinning)
└── Global/Local references management
```

### Performance Characteristics

#### Startup Performance:

```
JVM Startup Phases:
├── JVM Initialization    (~50-100ms)
├── Class Loading         (~100-500ms)
├── JIT Warm-up          (~1-10 seconds)
└── Steady State         (Optimal performance)

Memory Usage Progression:
├── Initial Heap         (~8-32 MB)
├── After Class Loading  (~50-100 MB)
├── After Warm-up        (~100-500 MB)
└── Steady State         (Application dependent)
```

#### Runtime Performance:

```
Execution Speed:
├── Interpreter          (Baseline)
├── C1 Compiler          (3-5x faster)
├── C2 Compiler          (10-50x faster)
└── Native Code          (Reference speed)

GC Pause Times:
├── Serial GC            (10ms - 1s+)
├── Parallel GC          (10ms - 500ms)  
├── CMS GC              (5ms - 100ms)
├── G1 GC               (1ms - 10ms)
└── ZGC/Shenandoah      (<1ms - 10ms)
```

### Monitoring and Debugging

#### #JVMFlags for Low-Level Tuning:

```bash
# Memory settings
-Xms2g -Xmx8g                    # Initial/Maximum heap
-XX:NewRatio=2                   # Old/Young generation ratio
-XX:MaxMetaspaceSize=256m        # Metaspace limit

# GC tuning  
-XX:+UseG1GC                     # Use G1 garbage collector
-XX:MaxGCPauseMillis=200         # Target pause time
-XX:G1HeapRegionSize=16m         # G1 region size

# JIT compilation
-XX:CompileThreshold=10000       # JIT compilation threshold
-XX:+TieredCompilation           # Enable tiered compilation

# Debugging
-XX:+PrintGC                     # Print GC information
-XX:+PrintGCDetails              # Detailed GC logging
-XX:+HeapDumpOnOutOfMemoryError  # Generate heap dump on OOM
```

#### #Low-Level #Monitoring Tools:

```bash
# Native memory tracking
-XX:NativeMemoryTracking=detail
jcmd <pid> VM.native_memory

# Class loading monitoring  
-XX:+TraceClassLoading
-XX:+TraceClassUnloading

# JIT compilation monitoring
-XX:+PrintCompilation
-XX:+UnlockDiagnosticVMOptions -XX:+PrintInlining

# GC analysis
-XX:+PrintGCApplicationStoppedTime
-XX:+PrintStringDeduplicationStatistics
```

This low-level documentation covers the internal architecture, memory management, and performance characteristics of the JVM ecosystem, providing insight into how Java applications execute at the system level.