---
area: Algorithms
tags:
  - algorithms
  - sorting
type: 
created: 2025-08-17 11:53
---

# Overview



## Algorithms for #Unordered Data

These algorithms work efficiently on completely random, unsorted data without any assumptions about initial order.

### #Quick Sort

**Best for:** General-purpose sorting of unordered data **Steps:**

1. Choose a pivot element
2. Partition array around pivot
3. Recursively sort sub-arrays
4. Combine results

```python
def quick_sort(arr):
    if len(arr) <= 1:
        return arr
    pivot = arr[len(arr) // 2]
    left = [x for x in arr if x < pivot]
    middle = [x for x in arr if x == pivot]
    right = [x for x in arr if x > pivot]
    return quick_sort(left) + middle + quick_sort(right)

# In-place version
def quick_sort_inplace(arr, low=0, high=None):
    if high is None:
        high = len(arr) - 1
    
    if low < high:
        pi = partition(arr, low, high)
        quick_sort_inplace(arr, low, pi - 1)
        quick_sort_inplace(arr, pi + 1, high)

def partition(arr, low, high):
    pivot = arr[high]
    i = low - 1
    for j in range(low, high):
        if arr[j] <= pivot:
            i += 1
            arr[i], arr[j] = arr[j], arr[i]
    arr[i + 1], arr[high] = arr[high], arr[i + 1]
    return i + 1
```

**Complexity:** O(n log n) average, O(n²) worst case, O(log n) space **Use Cases:** Random data, general-purpose sorting, when average performance matters

### #Merge Sort

**Best for:** Guaranteed performance on unordered data **Steps:**

1. Divide array into halves
2. Recursively sort each half
3. Merge sorted halves
4. Return merged result

```python
def merge_sort(arr):
    if len(arr) <= 1:
        return arr
    
    mid = len(arr) // 2
    left = merge_sort(arr[:mid])
    right = merge_sort(arr[mid:])
    
    return merge(left, right)

def merge(left, right):
    result = []
    i = j = 0
    while i < len(left) and j < len(right):
        if left[i] <= right[j]:
            result.append(left[i])
            i += 1
        else:
            result.append(right[j])
            j += 1
    result.extend(left[i:])
    result.extend(right[j:])
    return result
```

**Complexity:** O(n log n) time, O(n) space **Use Cases:** Stable sorting required, guaranteed performance, large unordered datasets

### #Heap Sort

**Best for:** Memory-constrained unordered data **Steps:**

1. Build max heap from array
2. Extract maximum element
3. Place at end of array
4. Heapify remaining elements
5. Repeat until sorted

```python
def heap_sort(arr):
    def heapify(arr, n, i):
        largest = i
        left = 2 * i + 1
        right = 2 * i + 2
        
        if left < n and arr[left] > arr[largest]:
            largest = left
        if right < n and arr[right] > arr[largest]:
            largest = right
            
        if largest != i:
            arr[i], arr[largest] = arr[largest], arr[i]
            heapify(arr, n, largest)
    
    n = len(arr)
    # Build max heap
    for i in range(n // 2 - 1, -1, -1):
        heapify(arr, n, i)
    
    # Extract elements one by one
    for i in range(n - 1, 0, -1):
        arr[0], arr[i] = arr[i], arr[0]
        heapify(arr, i, 0)
    
    return arr
```

**Complexity:** O(n log n) time, O(1) space **Use Cases:** Memory-constrained environments, guaranteed performance on random data

### #Selection Sort

**Best for:** Simple implementation for small unordered datasets **Steps:**

1. Find minimum element in unsorted portion
2. Swap with first unsorted element
3. Move boundary of sorted portion
4. Repeat until entire array is sorted

```python
def selection_sort(arr):
    n = len(arr)
    for i in range(n):
        min_idx = i
        for j in range(i+1, n):
            if arr[j] < arr[min_idx]:
                min_idx = j
        arr[i], arr[min_idx] = arr[min_idx], arr[i]
    return arr
```

**Complexity:** O(n²) time, O(1) space **Use Cases:** Small unordered datasets, simple implementation, memory-constrained

## Algorithms for #Ordered/Partially #Ordered Data

These algorithms perform exceptionally well when data has some existing order or is nearly sorted.

### #Insertion Sort

**Best for:** Nearly sorted or small datasets **Steps:**

1. Start with second element
2. Compare with previous elements
3. Insert in correct position
4. Repeat for all remaining elements

```python
def insertion_sort(arr):
    for i in range(1, len(arr)):
        key = arr[i]
        j = i - 1
        while j >= 0 and arr[j] > key:
            arr[j + 1] = arr[j]
            j -= 1
        arr[j + 1] = key
    return arr

# Optimized for nearly sorted data
def binary_insertion_sort(arr):
    for i in range(1, len(arr)):
        key = arr[i]
        # Binary search for insertion position
        left, right = 0, i
        while left < right:
            mid = (left + right) // 2
            if arr[mid] > key:
                right = mid
            else:
                left = mid + 1
        
        # Shift elements and insert
        for j in range(i, left, -1):
            arr[j] = arr[j-1]
        arr[left] = key
    return arr
```

**Complexity:** O(n) best case, O(n²) worst case, O(1) space **Use Cases:** Nearly sorted data, small datasets, online sorting, adaptive sorting

### #Bubble Sort (Optimized)

**Best for:** Educational purposes with nearly sorted data **Steps:**

1. Compare adjacent elements
2. Swap if they're in wrong order
3. Optimize by stopping if no swaps occur
4. Reduce range each pass

```python
def bubble_sort_optimized(arr):
    n = len(arr)
    for i in range(n):
        swapped = False
        for j in range(0, n-i-1):
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j+1], arr[j]
                swapped = True
        # If no swapping occurred, array is sorted
        if not swapped:
            break
    return arr
```

**Complexity:** O(n) best case, O(n²) worst case, O(1) space **Use Cases:** Educational purposes, nearly sorted data, detecting if already sorted

### #Tim Sort (Hybrid)

**Best for:** Real-world data with natural ordering **Steps:**

1. Identify natural runs in data
2. Extend short runs using insertion sort
3. Merge runs using merge sort strategy
4. Optimize for real-world patterns

```python
def tim_sort_simplified(arr):
    MIN_MERGE = 32
    
    def insertion_sort_range(arr, left, right):
        for i in range(left + 1, right + 1):
            key = arr[i]
            j = i - 1
            while j >= left and arr[j] > key:
                arr[j + 1] = arr[j]
                j -= 1
            arr[j + 1] = key
    
    def merge_ranges(arr, left, mid, right):
        left_arr = arr[left:mid+1]
        right_arr = arr[mid+1:right+1]
        
        i = j = 0
        k = left
        
        while i < len(left_arr) and j < len(right_arr):
            if left_arr[i] <= right_arr[j]:
                arr[k] = left_arr[i]
                i += 1
            else:
                arr[k] = right_arr[j]
                j += 1
            k += 1
        
        while i < len(left_arr):
            arr[k] = left_arr[i]
            i += 1
            k += 1
        
        while j < len(right_arr):
            arr[k] = right_arr[j]
            j += 1
            k += 1
    
    n = len(arr)
    
    # Sort individual subarrays of size MIN_MERGE
    for start in range(0, n, MIN_MERGE):
        end = min(start + MIN_MERGE - 1, n - 1)
        insertion_sort_range(arr, start, end)
    
    # Start merging
    size = MIN_MERGE
    while size < n:
        for start in range(0, n, size * 2):
            mid = start + size - 1
            end = min(start + size * 2 - 1, n - 1)
            
            if mid < end:
                merge_ranges(arr, start, mid, end)
        
        size *= 2
    
    return arr
```

**Complexity:** O(n) best case, O(n log n) worst case, O(n) space **Use Cases:** Real-world data, Python's built-in sort, data with natural patterns

## #Specialized Algorithms (Data Type Dependent)

These work best with specific data types regardless of initial order.

### #Counting Sort

**Best for:** Small range integers **Steps:**

1. Find maximum element
2. Create count array
3. Count occurrences of each element
4. Calculate cumulative count
5. Place elements in output array

```python
def counting_sort(arr):
    if not arr:
        return arr
    
    max_val = max(arr)
    min_val = min(arr)
    range_val = max_val - min_val + 1
    
    count = [0] * range_val
    output = [0] * len(arr)
    
    # Count occurrences
    for num in arr:
        count[num - min_val] += 1
    
    # Calculate cumulative count
    for i in range(1, len(count)):
        count[i] += count[i - 1]
    
    # Build output array
    for i in range(len(arr) - 1, -1, -1):
        output[count[arr[i] - min_val] - 1] = arr[i]
        count[arr[i] - min_val] -= 1
    
    return output
```

**Complexity:** O(n + k) time, O(k) space (k = range of input) **Use Cases:** Small range integers, counting elements, stable sorting needed

### #Radix Sort

**Best for:** Fixed-width integers **Steps:**

1. Find maximum number of digits
2. Sort by each digit (least to most significant)
3. Use stable sort for each digit position
4. Repeat for all digit positions

```python
def radix_sort(arr):
    if not arr:
        return arr
    
    max_num = max(arr)
    exp = 1
    
    while max_num // exp > 0:
        counting_sort_by_digit(arr, exp)
        exp *= 10
    
    return arr

def counting_sort_by_digit(arr, exp):
    n = len(arr)
    output = [0] * n
    count = [0] * 10
    
    # Count occurrences of digits
    for num in arr:
        index = num // exp
        count[index % 10] += 1
    
    # Calculate cumulative count
    for i in range(1, 10):
        count[i] += count[i - 1]
    
    # Build output array
    for i in range(n - 1, -1, -1):
        index = arr[i] // exp
        output[count[index % 10] - 1] = arr[i]
        count[index % 10] -= 1
    
    # Copy output array to arr
    for i in range(n):
        arr[i] = output[i]
```

**Complexity:** O(d × (n + k)) time, O(n + k) space **Use Cases:** Integers, fixed-length strings, when range is reasonable

### #Bucket Sort

**Best for:** Uniformly distributed data **Steps:**

1. Create empty buckets
2. Distribute elements into buckets
3. Sort individual buckets
4. Concatenate sorted buckets

```python
def bucket_sort(arr):
    if not arr:
        return arr
    
    bucket_count = len(arr)
    max_val = max(arr)
    min_val = min(arr)
    
    # Create empty buckets
    buckets = [[] for _ in range(bucket_count)]
    
    # Distribute elements into buckets
    for num in arr:
        if max_val == min_val:
            bucket_index = 0
        else:
            bucket_index = int((num - min_val) / (max_val - min_val + 1) * bucket_count)
            bucket_index = min(bucket_index, bucket_count - 1)
        buckets[bucket_index].append(num)
    
    # Sort individual buckets and concatenate
    result = []
    for bucket in buckets:
        result.extend(insertion_sort(bucket))  # Use insertion sort for small buckets
    
    return result
```

**Complexity:** O(n + k) average, O(n²) worst case, O(n + k) space **Use Cases:** Uniformly distributed data, floating-point numbers

## Selection Guide by Data Characteristics

### For Completely Random/Unordered Data:

|Size|Best Choice|Alternative|
|---|---|---|
|Small (< 50)|Selection Sort|Insertion Sort|
|Medium (50-10K)|Quick Sort|Merge Sort|
|Large (> 10K)|Merge Sort|Heap Sort|

### For Partially Ordered Data:

|Order Level|Best Choice|Complexity|
|---|---|---|
|Nearly Sorted|Insertion Sort|O(n)|
|Some Order|Tim Sort|O(n) to O(n log n)|
|Reverse Sorted|Quick Sort|O(n log n)|

### For Specific Data Types:

|Data Type|Best Choice|Requirements|
|---|---|---|
|Small Range Integers|Counting Sort|Range ≤ 10⁶|
|Large Integers|Radix Sort|Fixed width|
|Floating Point|Bucket Sort|Uniform distribution|
|Strings|Radix Sort|Fixed length|

### Performance on Different Data Orders:

|Algorithm|Random|Nearly Sorted|Reverse Sorted|Many Duplicates|
|---|---|---|---|---|
|Quick Sort|Excellent|Good|Poor*|Good|
|Merge Sort|Excellent|Good|Excellent|Excellent|
|Heap Sort|Excellent|Good|Excellent|Good|
|Insertion Sort|Poor|Excellent|Poor|Good|
|Tim Sort|Excellent|Excellent|Excellent|Excellent|
|Counting Sort|Excellent|Excellent|Excellent|Excellent|

*Poor due to O(n²) worst case, but can be improved with random pivot selection

## #Complexity Comparison

|Algorithm|Best Case|Average Case|Worst Case|Space|Stable|
|---|---|---|---|---|---|
|Bubble Sort|O(n)|O(n²)|O(n²)|O(1)|Yes|
|Quick Sort|O(n log n)|O(n log n)|O(n²)|O(log n)|No|
|Merge Sort|O(n log n)|O(n log n)|O(n log n)|O(n)|Yes|
|Heap Sort|O(n log n)|O(n log n)|O(n log n)|O(1)|No|
|Linear Search|O(1)|O(n)|O(n)|O(1)|-|
|Binary Search|O(1)|O(log n)|O(log n)|O(1)|-|
|Hash Search|O(1)|O(1)|O(n)|O(n)|-|

## #UseCase Recommendations

### Sorting

- **Small datasets (< 50 elements):** Insertion Sort or Selection Sort
- **General purpose:** Quick Sort or built-in sort
- **Stability required:** Merge Sort
- **Memory constrained:** Heap Sort
- **Nearly sorted data:** Insertion Sort or Bubble Sort
- **Large datasets:** Merge Sort or Heap Sort

### Searching

- **Unsorted data:** Linear Search or Hash Table
- **Sorted data:** Binary Search
- **Key-value pairs:** Hash Table
- **Graph/Tree structures:** DFS or BFS
- **Frequent searches:** Hash Table or Binary Search Tree
- **Range queries:** Binary Search or specialized trees

## Performance Tips

1. **Choose appropriate algorithm** based on data size and characteristics
2. **Consider stability** requirements for sorting
3. **Use built-in functions** when possible (usually optimized)
4. **Preprocess data** if multiple operations needed
5. **Consider space-time tradeoffs** based on constraints
6. **Profile your code** to identify bottlenecks
7. **Use appropriate data structures** for your access patterns