---
area: DSA
tags:
  - data-structures
  - fundamentals
type: 
created: 2025-08-17 11:37
---

# Overview

## #ArrayList

### Root Structure

java

```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable {
    
    // Default initial capacity
    private static final int DEFAULT_CAPACITY = 10;
    
    // Shared empty array instance used for empty instances
    private static final Object[] EMPTY_ELEMENTDATA = {};
    
    // Shared empty array instance used for default sized empty instances
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
    
    // Maximum array size (some VMs reserve header words)
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    
    // The array buffer into which the elements are stored
    transient Object[] elementData; // non-private for package access
    
    // The size of the ArrayList (number of elements it contains)
    private int size;
    
    // Inherited from AbstractList:
    // protected transient int modCount = 0;
}
```

### Basic Structure

- ArrayList in Java uses array of Object; objects stay in heap memory
- We can pass initial capacity while creating object or default capacity is **10**
- Implements List interface, extends AbstractList
- Not thread-safe (use Vector or Collections.synchronizedList() for thread safety)

### Core Operations

#### `Add` Operation:

- Increases the `modCount` (for fail-fast iterators)
- Checks if `size == array.length`, then grow; else puts data in `array[size]`
- Increases `size++`
- Time Complexity: O(1) amortized, O(n) worst case (when resizing)

#### `grow` Operation:

- Creates new array with increased capacity
- Default growth: `newCapacity = oldCapacity + (oldCapacity >> 1)` (1.5x growth)
- Copies all elements from old array to new array using `System.arraycopy()`
- Updates the internal array reference
- If calculated capacity exceeds `MAX_ARRAY_SIZE`, uses `Integer.MAX_VALUE - 8`

#### `get(index)` Operation:

- Performs bounds checking: `if (index >= size) throw IndexOutOfBoundsException`
- Returns `array[index]`
- Time Complexity: O(1)

#### `set(index, element)` Operation:

- Performs bounds checking
- Stores old value, sets new value at `array[index]`
- Returns old value
- Time Complexity: O(1)

#### `remove(index)` Operation:

- Performs bounds checking
- Stores element to be removed
- Calculates number of elements to shift: `numMoved = size - index - 1`
- Uses `System.arraycopy()` to shift elements left
- Sets last element to null: `array[--size] = null`
- Increases `modCount`
- Time Complexity: O(n)

#### `remove(Object)` Operation:

- Iterates through array to find first occurrence
- Calls `fastRemove(index)` when found
- Returns true if removed, false if not found
- Time Complexity: O(n)

### Key Properties

#### Capacity vs Size:

- **Capacity**: Length of internal array
- **Size**: Number of actual elements stored
- Capacity ≥ Size always

#### ModCount:

- Tracks structural modifications
- Used by iterators for fail-fast behavior
- Incremented on add, remove, clear operations

#### Memory Management:

- Elements stored as Object references
- Actual objects live in heap
- Array itself also in heap
- Automatic resizing prevents overflow

### Performance Characteristics

|Operation|Time Complexity|Notes|
|---|---|---|
|get(index)|O(1)|Direct array access|
|set(index, element)|O(1)|Direct array access|
|add(element)|O(1) amortized|O(n) when resizing needed|
|add(index, element)|O(n)|Requires shifting elements|
|remove(index)|O(n)|Requires shifting elements|
|remove(Object)|O(n)|Linear search + shifting|
|contains(Object)|O(n)|Linear search|
|indexOf(Object)|O(n)|Linear search|
|clear()|O(n)|Sets all elements to null|

### Space Complexity:

- O(n) where n is the capacity (not size)
- Wasted space when size << capacity

### Iterator Behavior:

- **Fail-fast**: Throws `ConcurrentModificationException` if list modified during iteration
- Checks `modCount` at each iteration step
- Use `ListIterator` for safe modification during iteration

### Common Methods:

#### Bulk Operations:

- `addAll(Collection)`: O(m) where m is size of collection
- `removeAll(Collection)`: O(n*m)
- `retainAll(Collection)`: O(n*m)
- `toArray()`: O(n)

#### Search Operations:

- `indexOf(Object)`: O(n) - first occurrence
- `lastIndexOf(Object)`: O(n) - last occurrence
- `contains(Object)`: O(n) - uses indexOf internally

#### Utility Methods:

- `isEmpty()`: O(1) - checks if size == 0
- `size()`: O(1) - returns size field
- `trimToSize()`: O(n) - reduces capacity to current size

### Best Practices:

1. **Initialize with estimated capacity** if size is known
2. **Use for frequent random access** (get/set operations)
3. **Avoid frequent insertions/deletions** in middle
4. **Consider LinkedList** for frequent insertions/deletions
5. **Use ensureCapacity()** before bulk additions
6. **Be careful with iterators** and concurrent modifications

### Memory Considerations:

- Default capacity of 10 might be wasteful for small lists
- Growth factor of 1.5x can lead to memory fragmentation
- Call `trimToSize()` after bulk operations to reclaim memory
- Each element reference takes 4-8 bytes (depending on JVM architecture)

---

## #LinkedList

### Root Structure

java

```java
public class LinkedList<E> extends AbstractSequentialList<E>
        implements List<E>, Deque<E>, Cloneable, java.io.Serializable {
    
    // Number of elements in the list
    transient int size = 0;
    
    // Pointer to first node
    transient Node<E> first;
    
    // Pointer to last node  
    transient Node<E> last;
    
    // Inherited from AbstractList:
    // protected transient int modCount = 0;
    
    // Inner Node class
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;
        
        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }
}
```

### Core Operations

#### `add(element)` - Add to end:

- Creates new node with `prev = last, next = null`
- Updates `last.next = newNode`
- Updates `last = newNode`
- If list was empty, `first = newNode`
- Increases `size++` and `modCount++`
- Time Complexity: O(1)

#### `add(index, element)` - Add at index:

- If `index == size`, calls `add(element)`
- Else finds node at index using `node(index)`
- Creates new node and updates links
- Time Complexity: O(n) due to traversal

#### `node(index)` - Internal traversal:

- If `index < size/2`: traverse from first
- Else: traverse from last (optimization)
- Time Complexity: O(n/2) = O(n)

#### `get(index)`:

- Calls `node(index).item`
- Time Complexity: O(n)

#### `set(index, element)`:

- Calls `node(index)` to find node
- Updates `node.item = element`
- Returns old value
- Time Complexity: O(n)

#### `remove(index)`:

- Calls `node(index)` to find node
- Calls `unlink(node)` to remove
- Time Complexity: O(n)

#### `unlink(node)` - Internal removal:

- Updates `prev.next = node.next`
- Updates `next.prev = node.prev`
- Handles edge cases for first/last nodes
- Sets node references to null for GC
- Time Complexity: O(1)

#### `remove(Object)`:

- Traverses list to find object
- Calls `unlink(node)` when found
- Time Complexity: O(n)

### Deque Operations (Efficient):

|Operation|Time Complexity|Description|
|---|---|---|
|`addFirst(e)`|O(1)|Add to beginning|
|`addLast(e)`|O(1)|Add to end|
|`removeFirst()`|O(1)|Remove from beginning|
|`removeLast()`|O(1)|Remove from end|
|`getFirst()`|O(1)|Get first element|
|`getLast()`|O(1)|Get last element|
|`peekFirst()`|O(1)|Get first (null if empty)|
|`peekLast()`|O(1)|Get last (null if empty)|

### Performance Characteristics

|Operation|LinkedList|ArrayList|Notes|
|---|---|---|---|
|get(index)|O(n)|O(1)|ArrayList wins|
|set(index)|O(n)|O(1)|ArrayList wins|
|add(element)|O(1)|O(1) amortized|Both good|
|add(index, element)|O(n)|O(n)|LinkedList better for start/end|
|remove(index)|O(n)|O(n)|LinkedList better for start/end|
|addFirst/Last|O(1)|O(n)|LinkedList wins|
|removeFirst/Last|O(1)|O(n)|LinkedList wins|

### Memory Considerations:

- **Overhead**: Each node has 2 extra references (next, prev)
- **Memory per element**: Object reference + 2 pointers = ~24 bytes per node
- **No wasted capacity** like ArrayList
- **Better memory locality** in ArrayList due to array structure

### Best Use Cases:

- Frequent insertions/deletions at beginning/end
- Implementing stacks, queues, deques
- When you don't know the size in advance
- When you need constant-time insertion/deletion at known positions

---

## #HashMap

### Root Structure

java

```java
public class HashMap<K,V> extends AbstractMap<K,V>
        implements Map<K,V>, Cloneable, Serializable {
    
    // Static constants
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // 16
    static final int MAXIMUM_CAPACITY = 1 << 30;
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    static final int TREEIFY_THRESHOLD = 8;
    static final int UNTREEIFY_THRESHOLD = 6;
    static final int MIN_TREEIFY_CAPACITY = 64;
    
    // Instance fields
    transient Node<K,V>[] table;           // The hash table
    transient Set<Map.Entry<K,V>> entrySet; // Cached entry set
    transient int size;                     // Number of key-value pairs
    transient int modCount;                 // Structural modifications count
    int threshold;                          // Resize threshold (capacity * load factor)
    final float loadFactor;                 // Load factor for hash table
    
    // Basic Node for hash table
    static class Node<K,V> implements Map.Entry<K,V> {
        final int hash;
        final K key;
        V value;
        Node<K,V> next;
        
        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }
        
        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }
        
        public final boolean equals(Object o) {
            if (o == this) return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
        
        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }
    }
    
    // TreeNode for red-black tree (simplified)
    static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
        TreeNode<K,V> parent;
        TreeNode<K,V> left;
        TreeNode<K,V> right;
        TreeNode<K,V> prev;    // needed to unlink next upon deletion
        boolean red;
        
        TreeNode(int hash, K key, V val, Node<K,V> next) {
            super(hash, key, val, next);
        }
    }
}
```

### Core Operations

#### `put(key, value)`:

1. Calculates hash: `hash(key.hashCode())`
2. Finds bucket: `(n-1) & hash` where n = table length
3. If bucket empty: create new node
4. If key exists: update value
5. If collision: add to linked list or tree
6. Check if resize needed: `size > threshold`
7. Time Complexity: O(1) average, O(log n) worst case (with trees)

#### Hash Function:

java

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

- XORs higher bits with lower bits for better distribution

#### `get(key)`:

1. Calculates hash
2. Finds bucket
3. If first node matches: return value
4. Else traverse linked list or search tree
5. Time Complexity: O(1) average, O(log n) worst case

#### `resize()`:

- Triggered when `size > capacity * load_factor`
- Doubles the capacity
- Rehashes all existing entries
- Time Complexity: O(n)

### Load Factor and Resizing:

- **Load Factor**: ratio of size to capacity
- **Default**: 0.75 (good balance between time and space)
- **Higher load factor**: more space-efficient, slower access
- **Lower load factor**: faster access, more memory usage

### Collision Resolution:

#### Separate Chaining:

- Uses linked lists in buckets
- When list gets too long (>8), converts to red-black tree
- Tree improves worst-case from O(n) to O(log n)

#### Tree Conversion:

- **Treeify threshold**: 8 nodes in bucket
- **Untreeify threshold**: 6 nodes in bucket
- **Minimum table size for treeify**: 64

### Performance Characteristics:

|Operation|Average Case|Worst Case|Notes|
|---|---|---|---|
|get(key)|O(1)|O(log n)|With good hash function|
|put(key, value)|O(1)|O(log n)|May trigger resize O(n)|
|remove(key)|O(1)|O(log n)|Tree traversal worst case|
|containsKey(key)|O(1)|O(log n)|Same as get|
|containsValue(value)|O(n)|O(n)|Must scan all entries|

### Key Methods:

#### Bulk Operations:

- `putAll(Map)`: O(m) where m is size of input map
- `clear()`: O(n) - sets all buckets to null
- `keySet()`: O(1) - returns view (lazy)
- `values()`: O(1) - returns view (lazy)
- `entrySet()`: O(1) - returns view (lazy)

#### Iteration:

- Iterates through all buckets, then chains/trees
- Time: O(capacity + size)
- Fail-fast iterators throw `ConcurrentModificationException`

### Important Properties:

#### Hash Code Contract:

- If `a.equals(b)` then `a.hashCode() == b.hashCode()`
- Objects used as keys should be immutable
- Override both `equals()` and `hashCode()` together

#### Null Handling:

- **One null key** allowed (stored at index 0)
- **Multiple null values** allowed
- `get(null)` and `put(null, value)` work correctly

### Capacity and Load Factor Tuning:

#### Initial Capacity:

- Should be power of 2
- Set based on expected number of entries
- `initialCapacity = expectedEntries / loadFactor + 1`

#### Load Factor:

- **0.75**: Default, good balance
- **Higher (0.9)**: More space-efficient, slower access
- **Lower (0.5)**: Faster access, more memory usage

### Memory Considerations:

- **Overhead**: Each entry has hash, key, value, next reference
- **Wasted space**: Empty buckets consume memory
- **Resize cost**: Temporary double memory usage during resize

### Best Practices:

1. **Implement proper hashCode() and equals()** for custom keys
2. **Use immutable objects** as keys
3. **Set initial capacity** if size is known
4. **Consider load factor** based on memory vs speed requirements
5. **Use LinkedHashMap** if insertion order matters
6. **Use TreeMap** if sorting is needed

### Common Pitfalls:

- **Mutable keys**: Can break hash table invariants
- **Poor hash function**: Causes clustering and poor performance
- **Not overriding equals/hashCode**: Inconsistent behavior
- **Using objects with expensive hashCode()**: Performance impact