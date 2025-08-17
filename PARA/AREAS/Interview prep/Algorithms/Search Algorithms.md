---
area: Algorithms
tags:
  - algorithms
  - search
type: 
created: 2025-08-17 11:41
---




## #LinearSearch

### Algorithm Steps:

1. Start from the first element of the array
2. Compare each element with the target value
3. If match found, return the index
4. If end of array reached without match, return -1

### Code Implementation:

```java
public static int linearSearch(int[] arr, int target) {
    for (int i = 0; i < arr.length; i++) {
        if (arr[i] == target) {
            return i;  // Found at index i
        }
    }
    return -1;  // Not found
}
```

### Complexity Analysis:

- **Time Complexity**:
    - Best Case: O(1) - element found at first position
    - Average Case: O(n) - element found at middle
    - Worst Case: O(n) - element at last position or not found
- **Space Complexity**: O(1) - constant extra space
- **Stability**: N/A (not a sorting algorithm)

### Use Cases:

- **Small datasets** where simplicity matters
- **Unsorted arrays** (only option for unsorted data)
- **Linked lists** where random access is not available
- **One-time searches** where preprocessing overhead isn't justified
- **Real-time systems** with predictable O(n) behavior

---

## #Binary Search

### Algorithm Steps:

1. Ensure array is sorted (prerequisite)
2. Set left = 0, right = array.length - 1
3. While left ≤ right:
    - Calculate mid = left + (right - left) / 2
    - If arr[mid] == target, return mid
    - If arr[mid] < target, search right half (left = mid + 1)
    - If arr[mid] > target, search left half (right = mid - 1)
4. Return -1 if not found

### Code Implementation:

```java
public static int binarySearch(int[] arr, int target) {
    int left = 0, right = arr.length - 1;
    
    while (left <= right) {
        int mid = left + (right - left) / 2;  // Avoid overflow
        
        if (arr[mid] == target) {
            return mid;
        } else if (arr[mid] < target) {
            left = mid + 1;   // Search right half
        } else {
            right = mid - 1;  // Search left half
        }
    }
    return -1;
}
```

### Recursive Implementation:

```java
public static int binarySearchRecursive(int[] arr, int target, int left, int right) {
    if (left > right) return -1;
    
    int mid = left + (right - left) / 2;
    
    if (arr[mid] == target) return mid;
    if (arr[mid] > target) return binarySearchRecursive(arr, target, left, mid - 1);
    return binarySearchRecursive(arr, target, mid + 1, right);
}
```

### Complexity Analysis:

- **Time Complexity**:
    - Best Case: O(1) - element found at middle
    - Average Case: O(log n) - logarithmic search
    - Worst Case: O(log n) - maximum log₂(n) comparisons
- **Space Complexity**:
    - Iterative: O(1) - constant space
    - Recursive: O(log n) - recursion stack
- **Prerequisite**: Array must be sorted

### Use Cases:

- **Large sorted datasets** (arrays, lists)
- **Database indexing** (B-trees, B+ trees)
- **Dictionary lookups** and phone books
- **Finding insertion point** in sorted arrays
- **Range queries** (finding boundaries)
- **Mathematical computations** (square root, etc.)

---

## #Jump Search

### Algorithm Steps:

1. Choose optimal jump size = √n
2. Jump by step size until arr[jump] ≥ target
3. Perform linear search in the identified block
4. Return index if found, -1 otherwise

### Code Implementation:

```java
public static int jumpSearch(int[] arr, int target) {
    int n = arr.length;
    int step = (int) Math.sqrt(n);  // Optimal jump size
    int prev = 0;
    
    // Jump to find the block
    while (arr[Math.min(step, n) - 1] < target) {
        prev = step;
        step += (int) Math.sqrt(n);
        if (prev >= n) return -1;
    }
    
    // Linear search in identified block
    while (arr[prev] < target) {
        prev++;
        if (prev == Math.min(step, n)) return -1;
    }
    
    return (arr[prev] == target) ? prev : -1;
}
```

### Complexity Analysis:

- **Time Complexity**: O(√n) - optimal jump size
- **Space Complexity**: O(1) - constant space
- **Optimal Jump Size**: √n provides best performance
- **Comparisons**: At most √n + √n = 2√n

### Use Cases:

- **Large sorted arrays** where binary search might have cache misses
- **Systems with expensive comparisons** (better than linear)
- **When binary search is overkill** but linear is too slow
- **Block-based storage systems**

---

## #Interpolation Search

### Algorithm Steps:

1. Ensure array is sorted and uniformly distributed
2. Calculate position using interpolation formula: `pos = low + [(target - arr[low]) × (high - low)] / [arr[high] - arr[low]]`
3. If arr[pos] == target, return pos
4. If arr[pos] < target, search right: low = pos + 1
5. If arr[pos] > target, search left: high = pos - 1
6. Repeat until found or bounds cross

### Code Implementation:

```java
public static int interpolationSearch(int[] arr, int target) {
    int low = 0, high = arr.length - 1;
    
    while (low <= high && target >= arr[low] && target <= arr[high]) {
        // If only one element
        if (low == high) {
            return (arr[low] == target) ? low : -1;
        }
        
        // Interpolation formula to find position
        int pos = low + ((target - arr[low]) * (high - low)) / (arr[high] - arr[low]);
        
        if (arr[pos] == target) {
            return pos;
        } else if (arr[pos] < target) {
            low = pos + 1;
        } else {
            high = pos - 1;
        }
    }
    return -1;
}
```

### Complexity Analysis:

- **Time Complexity**:
    - Best/Average Case: O(log log n) - for uniformly distributed data
    - Worst Case: O(n) - for non-uniform distribution
- **Space Complexity**: O(1) - constant space
- **Prerequisite**: Sorted array with uniform distribution

### Use Cases:

- **Uniformly distributed data** (phone directories, dictionaries)
- **Large datasets** with predictable distribution
- **Numerical data** with linear progression
- **When data follows a pattern** (timestamps, IDs)

---

## #Exponential Search

### Algorithm Steps:

1. Check if first element is the target
2. Find range by doubling index: 1, 2, 4, 8, 16, ...
3. Continue until arr[i] > target or end of array
4. Perform binary search in the identified range [i/2, min(i, n-1)]

### Code Implementation:

```java
public static int exponentialSearch(int[] arr, int target) {
    int n = arr.length;
    
    // If target is at first position
    if (arr[0] == target) return 0;
    
    // Find range for binary search by doubling
    int i = 1;
    while (i < n && arr[i] <= target) {
        i *= 2;
    }
    
    // Binary search in identified range
    return binarySearch(arr, target, i / 2, Math.min(i, n - 1));
}

private static int binarySearch(int[] arr, int target, int left, int right) {
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] == target) return mid;
        if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}
```

### Complexity Analysis:

- **Time Complexity**: O(log n) - finding range O(log n) + binary search O(log n)
- **Space Complexity**: O(1) - constant space
- **Range Finding**: O(log n) to find the range
- **Binary Search**: O(log n) in the identified range

### Use Cases:

- **Unbounded/infinite arrays** where size is unknown
- **When target is close to beginning** (better than binary search)
- **Linked lists** where jumping is expensive
- **Streaming data** with unknown bounds

---

## #Ternary Search

### Algorithm Steps:

1. Divide array into three parts using two mid points:
    - mid1 = left + (right - left) / 3
    - mid2 = right - (right - left) / 3
2. Compare target with arr[mid1] and arr[mid2]
3. Eliminate 1/3 of array based on comparison
4. Recursively search in remaining 2/3
5. Continue until found or bounds cross

### Code Implementation:

```java
public static int ternarySearch(int[] arr, int target, int left, int right) {
    if (left > right) return -1;
    
    // Two middle points
    int mid1 = left + (right - left) / 3;
    int mid2 = right - (right - left) / 3;
    
    if (arr[mid1] == target) return mid1;
    if (arr[mid2] == target) return mid2;
    
    // Target in first third
    if (target < arr[mid1]) {
        return ternarySearch(arr, target, left, mid1 - 1);
    }
    // Target in third third
    else if (target > arr[mid2]) {
        return ternarySearch(arr, target, mid2 + 1, right);
    }
    // Target in middle third
    else {
        return ternarySearch(arr, target, mid1 + 1, mid2 - 1);
    }
}
```

### Complexity Analysis:

- **Time Complexity**: O(log₃ n) ≈ O(log n) - base 3 logarithm
- **Space Complexity**: O(log n) - recursion stack
- **Comparisons**: More comparisons per iteration than binary search
- **Practical Performance**: Usually slower than binary search due to more comparisons

### Use Cases:

- **Theoretical interest** (demonstrates divide-and-conquer)
- **Unimodal functions** (finding maximum/minimum)
- **Mathematical optimization** problems
- **Educational purposes** (algorithm analysis)

---

## #Fibonacci Search

### Algorithm Steps:

1. Find smallest Fibonacci number ≥ array length
2. Use Fibonacci numbers to divide array into golden ratio
3. Compare target with element at Fibonacci position
4. Eliminate part of array and adjust Fibonacci numbers
5. Continue until found or array exhausted

### Code Implementation:

```java
public static int fibonacciSearch(int[] arr, int target) {
    int n = arr.length;
    
    // Generate Fibonacci numbers
    int fib2 = 0, fib1 = 1, fib = 1;
    while (fib < n) {
        fib2 = fib1;
        fib1 = fib;
        fib = fib1 + fib2;
    }
    
    int offset = -1;
    
    while (fib > 1) {
        // Check if fib2 is valid index
        int i = Math.min(offset + fib2, n - 1);
        
        if (arr[i] < target) {
            fib = fib1;
            fib1 = fib2;
            fib2 = fib - fib1;
            offset = i;
        } else if (arr[i] > target) {
            fib = fib2;
            fib1 = fib1 - fib2;
            fib2 = fib - fib1;
        } else {
            return i;
        }
    }
    
    // Check last element
    if (fib1 == 1 && offset + 1 < n && arr[offset + 1] == target) {
        return offset + 1;
    }
    
    return -1;
}
```

### Complexity Analysis:

- **Time Complexity**: O(log n) - similar to binary search
- **Space Complexity**: O(1) - constant space
- **Advantage**: No division operations (useful for some processors)
- **Disadvantage**: More complex implementation

### Use Cases:

- **Systems without division operator** (embedded systems)
- **Processors where division is expensive**
- **Theoretical algorithm studies**
- **Golden ratio based applications**

---

## #Depth-First Search (DFS)

### Algorithm Steps:

1. Start from source node
2. Mark current node as visited
3. Process current node
4. Recursively visit all unvisited adjacent nodes
5. Backtrack when no unvisited neighbors exist

### Code Implementation:

```java
// Graph representation using adjacency list
class Graph {
    private int vertices;
    private List<List<Integer>> adjList;
    
    public Graph(int vertices) {
        this.vertices = vertices;
        adjList = new ArrayList<>(vertices);
        for (int i = 0; i < vertices; i++) {
            adjList.add(new ArrayList<>());
        }
    }
    
    public void addEdge(int src, int dest) {
        adjList.get(src).add(dest);
    }
    
    public boolean dfsSearch(int start, int target) {
        boolean[] visited = new boolean[vertices];
        return dfsUtil(start, target, visited);
    }
    
    private boolean dfsUtil(int node, int target, boolean[] visited) {
        visited[node] = true;
        
        if (node == target) return true;
        
        for (int neighbor : adjList.get(node)) {
            if (!visited[neighbor]) {
                if (dfsUtil(neighbor, target, visited)) {
                    return true;
                }
            }
        }
        return false;
    }
}
```

### Complexity Analysis:

- **Time Complexity**: O(V + E) - visit all vertices and edges
- **Space Complexity**: O(V) - recursion stack + visited array
- **Complete**: Yes, finds solution if exists
- **Optimal**: No, may not find shortest path

### Use Cases:

- **Path finding** in mazes and puzzles
- **Topological sorting** in DAGs
- **Cycle detection** in graphs
- **Connected components** finding
- **Tree traversals** (preorder, inorder, postorder)
- **Solving puzzles** (N-Queens, Sudoku)

---

## #Breadth-First Search (BFS)

### Algorithm Steps:

1. Start from source node, add to queue
2. Mark source as visited
3. While queue is not empty:
    - Dequeue front node
    - Process current node
    - Add all unvisited neighbors to queue
    - Mark neighbors as visited
4. Continue until target found or queue empty

### Code Implementation:

```java
public boolean bfsSearch(int start, int target) {
    boolean[] visited = new boolean[vertices];
    Queue<Integer> queue = new LinkedList<>();
    
    visited[start] = true;
    queue.offer(start);
    
    while (!queue.isEmpty()) {
        int node = queue.poll();
        
        if (node == target) return true;
        
        for (int neighbor : adjList.get(node)) {
            if (!visited[neighbor]) {
                visited[neighbor] = true;
                queue.offer(neighbor);
            }
        }
    }
    return false;
}

// BFS with path tracking
public List<Integer> bfsPath(int start, int target) {
    boolean[] visited = new boolean[vertices];
    int[] parent = new int[vertices];
    Queue<Integer> queue = new LinkedList<>();
    
    Arrays.fill(parent, -1);
    visited[start] = true;
    queue.offer(start);
    
    while (!queue.isEmpty()) {
        int node = queue.poll();
        
        if (node == target) {
            return constructPath(parent, start, target);
        }
        
        for (int neighbor : adjList.get(node)) {
            if (!visited[neighbor]) {
                visited[neighbor] = true;
                parent[neighbor] = node;
                queue.offer(neighbor);
            }
        }
    }
    return new ArrayList<>(); // No path found
}
```

### Complexity Analysis:

- **Time Complexity**: O(V + E) - visit all vertices and edges
- **Space Complexity**: O(V) - queue + visited array
- **Complete**: Yes, finds solution if exists
- **Optimal**: Yes, finds shortest path (unweighted graphs)

### Use Cases:

- **Shortest path** in unweighted graphs
- **Level-order traversal** of trees
- **Finding connected components**
- **Social network analysis** (degrees of separation)
- **Web crawling** (crawl by levels)
- **GPS navigation** (shortest route)

---

## Algorithm Comparison Table

|Algorithm|Time Complexity|Space Complexity|Prerequisite|Best Use Case|
|---|---|---|---|---|
|**Linear Search**|O(n)|O(1)|None|Small/unsorted arrays|
|**Binary Search**|O(log n)|O(1)|Sorted array|Large sorted arrays|
|**Jump Search**|O(√n)|O(1)|Sorted array|Cache-conscious searches|
|**Interpolation**|O(log log n)|O(1)|Uniform distribution|Phone books, dictionaries|
|**Exponential**|O(log n)|O(1)|Sorted array|Unbounded arrays|
|**Ternary**|O(log₃ n)|O(log n)|Sorted array|Mathematical optimization|
|**Fibonacci**|O(log n)|O(1)|Sorted array|No division operations|
|**DFS**|O(V + E)|O(V)|Graph|Path finding, puzzles|
|**BFS**|O(V + E)|O(V)|Graph|Shortest path, levels|

## Performance Comparison (1M elements)

|Algorithm|Best Case|Average Case|Worst Case|Memory Usage|
|---|---|---|---|---|
|Linear|1 comparison|500K comparisons|1M comparisons|Minimal|
|Binary|1 comparison|20 comparisons|20 comparisons|Minimal|
|Jump|1 comparison|2K comparisons|2K comparisons|Minimal|
|Interpolation|1 comparison|5 comparisons|1M comparisons|Minimal|
|Exponential|1 comparison|20 comparisons|20 comparisons|Minimal|

## Selection Guidelines

### Choose Linear Search when:

- Array size < 100 elements
- Array is unsorted
- Searching once in unsorted data
- Memory is extremely limited

### Choose Binary Search when:

- Array is sorted
- Multiple searches on same data
- Array size > 1000 elements
- Memory is not a constraint

### Choose Jump Search when:

- Large sorted arrays with cache concerns
- Binary search causes too many cache misses
- Need balance between linear and binary

### Choose Interpolation Search when:

- Data is uniformly distributed
- Large datasets with predictable patterns
- Numerical data with linear progression

### Choose DFS when:

- Finding any path (not necessarily shortest)
- Memory is limited (compared to BFS)
- Exploring deep paths is preferred
- Recursive solution is natural

### Choose BFS when:

- Finding shortest path is required
- Level-by-level exploration needed
- All nodes at distance k before k+1
- Memory for queue is available