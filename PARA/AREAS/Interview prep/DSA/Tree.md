---
area: DSA
tags:
  - data-structures
  - trees
type: 
created: 2025-08-17 11:57
---

# Overview

## #Tree

### Root Structure

```java
// Generic Tree Node
class TreeNode<T> {
    T data;
    List<TreeNode<T>> children;
    TreeNode<T> parent;
    
    TreeNode(T data) {
        this.data = data;
        this.children = new ArrayList<>();
        this.parent = null;
    }
    
    // Add child
    void addChild(TreeNode<T> child) {
        children.add(child);
        child.parent = this;
    }
    
    // Remove child
    boolean removeChild(TreeNode<T> child) {
        if (children.remove(child)) {
            child.parent = null;
            return true;
        }
        return false;
    }
}
```

### Basic Structure

- Hierarchical data structure with nodes connected by edges
- Each node has a parent (except root) and zero or more children
- No cycles allowed (acyclic graph)
- Root node has no parent, leaf nodes have no children

### Key Properties:

- **Root**: Top node with no parent
- **Leaf**: Node with no children
- **Internal Node**: Node with at least one child
- **Height**: Longest path from node to leaf
- **Depth/Level**: Distance from root to node
- **Degree**: Number of children of a node

### Tree Traversals:

- **Pre-order**: Root → Left → Right (N-L-R)
- **Post-order**: Left → Right → Root (L-R-N)
- **Level-order**: Level by level (BFS)

### Applications:

- File systems (directories and files)
- Organization hierarchies
- Expression parsing
- Decision trees
- Database indexing

---

## #BinaryTree

### Root Structure

```java
class BinaryTreeNode<T> {
    T data;
    BinaryTreeNode<T> left;
    BinaryTreeNode<T> right;
    BinaryTreeNode<T> parent; // Optional for some implementations
    
    BinaryTreeNode(T data) {
        this.data = data;
        this.left = null;
        this.right = null;
        this.parent = null;
    }
    
    BinaryTreeNode(T data, BinaryTreeNode<T> left, BinaryTreeNode<T> right) {
        this.data = data;
        this.left = left;
        this.right = right;
        if (left != null) left.parent = this;
        if (right != null) right.parent = this;
    }
}
```

### Basic Structure

- Each node has at most **2 children**: left child and right child
- Binary tree is either empty or consists of root with left and right subtrees
- Maximum nodes at level i: **2^i**
- Maximum nodes in tree of height h: **2^(h+1) - 1**

### Types of Binary Trees:

#### Full Binary Tree:

- Every node has either 0 or 2 children
- No node has exactly 1 child

#### Complete Binary Tree:

- All levels filled except possibly the last
- Last level filled from left to right
- Used in heap implementations

#### Perfect Binary Tree:

- All internal nodes have 2 children
- All leaves at same level
- Total nodes = 2^h - 1 where h is height

#### Balanced Binary Tree:

- Height of left and right subtrees differ by at most 1
- For every node in the tree

### Traversal Implementations:

```java
// In-order traversal (L-N-R)
void inorderTraversal(BinaryTreeNode<T> root) {
    if (root != null) {
        inorderTraversal(root.left);
        System.out.print(root.data + " ");
        inorderTraversal(root.right);
    }
}

// Pre-order traversal (N-L-R)
void preorderTraversal(BinaryTreeNode<T> root) {
    if (root != null) {
        System.out.print(root.data + " ");
        preorderTraversal(root.left);
        preorderTraversal(root.right);
    }
}

// Post-order traversal (L-R-N)
void postorderTraversal(BinaryTreeNode<T> root) {
    if (root != null) {
        postorderTraversal(root.left);
        postorderTraversal(root.right);
        System.out.print(root.data + " ");
    }
}

// Level-order traversal (BFS)
void levelOrderTraversal(BinaryTreeNode<T> root) {
    if (root == null) return;
    
    Queue<BinaryTreeNode<T>> queue = new LinkedList<>();
    queue.offer(root);
    
    while (!queue.isEmpty()) {
        BinaryTreeNode<T> current = queue.poll();
        System.out.print(current.data + " ");
        
        if (current.left != null) queue.offer(current.left);
        if (current.right != null) queue.offer(current.right);
    }
}
```

### Performance Characteristics:

|Operation|Average Case|Worst Case|Best Case|
|---|---|---|---|
|Search|O(n)|O(n)|O(log n)|
|Insertion|O(log n)|O(n)|O(1)|
|Deletion|O(log n)|O(n)|O(1)|
|Traversal|O(n)|O(n)|O(n)|

---

## #BinarySearch #BST

### Root Structure

```java
class BSTNode<T extends Comparable<T>> {
    T data;
    BSTNode<T> left;
    BSTNode<T> right;
    BSTNode<T> parent; // Optional for easier deletion
    
    BSTNode(T data) {
        this.data = data;
        this.left = null;
        this.right = null;
        this.parent = null;
    }
}

class BinarySearchTree<T extends Comparable<T>> {
    private BSTNode<T> root;
    private int size;
    
    public BinarySearchTree() {
        this.root = null;
        this.size = 0;
    }
}
```

### Basic Structure

- Binary tree with **ordering property**
- **Left subtree**: All values < parent's value
- **Right subtree**: All values > parent's value
- **In-order traversal** gives sorted sequence
- No duplicate values (in standard implementation)

### Core Operations:

#### Search Operation:

```java
public boolean search(T key) {
    return searchNode(root, key) != null;
}

private BSTNode<T> searchNode(BSTNode<T> node, T key) {
    if (node == null || node.data.equals(key)) {
        return node;
    }
    
    if (key.compareTo(node.data) < 0) {
        return searchNode(node.left, key);
    } else {
        return searchNode(node.right, key);
    }
}
```

#### Insert Operation:

```java
public void insert(T key) {
    root = insertNode(root, key);
    size++;
}

private BSTNode<T> insertNode(BSTNode<T> node, T key) {
    if (node == null) {
        return new BSTNode<>(key);
    }
    
    if (key.compareTo(node.data) < 0) {
        node.left = insertNode(node.left, key);
        if (node.left != null) node.left.parent = node;
    } else if (key.compareTo(node.data) > 0) {
        node.right = insertNode(node.right, key);
        if (node.right != null) node.right.parent = node;
    }
    // Duplicate keys not allowed
    
    return node;
}
```

#### Delete Operation:

```java
public void delete(T key) {
    root = deleteNode(root, key);
    size--;
}

private BSTNode<T> deleteNode(BSTNode<T> node, T key) {
    if (node == null) return null;
    
    if (key.compareTo(node.data) < 0) {
        node.left = deleteNode(node.left, key);
    } else if (key.compareTo(node.data) > 0) {
        node.right = deleteNode(node.right, key);
    } else {
        // Node to be deleted found
        
        // Case 1: Node with no children (leaf)
        if (node.left == null && node.right == null) {
            return null;
        }
        
        // Case 2: Node with one child
        if (node.left == null) {
            return node.right;
        }
        if (node.right == null) {
            return node.left;
        }
        
        // Case 3: Node with two children
        // Find inorder successor (smallest in right subtree)
        BSTNode<T> successor = findMin(node.right);
        node.data = successor.data;
        node.right = deleteNode(node.right, successor.data);
    }
    
    return node;
}

private BSTNode<T> findMin(BSTNode<T> node) {
    while (node.left != null) {
        node = node.left;
    }
    return node;
}
```

### Performance Characteristics:

|Operation|Average Case|Worst Case|Best Case|
|---|---|---|---|
|Search|O(log n)|O(n)|O(1)|
|Insert|O(log n)|O(n)|O(1)|
|Delete|O(log n)|O(n)|O(1)|
|Min/Max|O(log n)|O(n)|O(1)|

### BST Properties:

- **In-order traversal**: Always gives sorted sequence
- **Height**: Best case O(log n), worst case O(n)
- **Space complexity**: O(n)
- **Degenerates to linked list** in worst case (sorted input)

---

## #Self-Balancing

### Basic Concept

- Automatically maintain balanced height during insertions and deletions
- Guarantee O(log n) operations in worst case
- Use **rotations** to maintain balance
- Common types: AVL Tree, Red-Black Tree, Splay Tree

### Rotation Operations:

#### Left Rotation:

```java
private BSTNode<T> rotateLeft(BSTNode<T> x) {
    BSTNode<T> y = x.right;
    x.right = y.left;
    if (y.left != null) {
        y.left.parent = x;
    }
    y.parent = x.parent;
    if (x.parent == null) {
        root = y;
    } else if (x == x.parent.left) {
        x.parent.left = y;
    } else {
        x.parent.right = y;
    }
    y.left = x;
    x.parent = y;
    return y;
}
```

#### Right Rotation:

```java
private BSTNode<T> rotateRight(BSTNode<T> y) {
    BSTNode<T> x = y.left;
    y.left = x.right;
    if (x.right != null) {
        x.right.parent = y;
    }
    x.parent = y.parent;
    if (y.parent == null) {
        root = x;
    } else if (y == y.parent.right) {
        y.parent.right = x;
    } else {
        y.parent.left = x;
    }
    x.right = y;
    y.parent = x;
    return x;
}
```

### AVL Tree:

- **Balance Factor**: height(left) - height(right) ∈ {-1, 0, 1}
- **Stricter balancing** than Red-Black trees
- **Faster lookups** due to better balance
- **More rotations** needed for insertions/deletions

### Splay Tree:

- **Self-adjusting** binary search tree
- **Recently accessed** elements moved to root
- **Amortized** O(log n) performance
- Good for **temporal locality** access patterns

---

## #Red-BlackTree

### Root Structure

```java
enum Color { RED, BLACK }

class RBNode<T extends Comparable<T>> {
    T data;
    Color color;
    RBNode<T> left;
    RBNode<T> right;
    RBNode<T> parent;
    
    RBNode(T data, Color color) {
        this.data = data;
        this.color = color;
        this.left = null;
        this.right = null;
        this.parent = null;
    }
}

class RedBlackTree<T extends Comparable<T>> {
    private RBNode<T> root;
    private RBNode<T> NIL; // Sentinel node for null references
    private int size;
    
    public RedBlackTree() {
        NIL = new RBNode<>(null, Color.BLACK);
        root = NIL;
        size = 0;
    }
}
```

### Basic Structure

- Binary search tree with additional **color property**
- Each node is either **RED** or **BLACK**
- Uses **NIL nodes** (sentinel) for null references
- Maintains balance through **red-black properties**

### Red-Black Properties:

1. **Every node** is either red or black
2. **Root** is always black
3. **All NIL nodes** (leaves) are black
4. **Red node** cannot have red children (no two red nodes adjacent)
5. **Every path** from node to descendant NIL has same number of black nodes

### Core Operations:

#### Insert Operation:

```java
public void insert(T key) {
    RBNode<T> newNode = new RBNode<>(key, Color.RED);
    RBNode<T> parent = NIL;
    RBNode<T> current = root;
    
    // Standard BST insertion
    while (current != NIL) {
        parent = current;
        if (key.compareTo(current.data) < 0) {
            current = current.left;
        } else {
            current = current.right;
        }
    }
    
    newNode.parent = parent;
    if (parent == NIL) {
        root = newNode; // Tree was empty
    } else if (key.compareTo(parent.data) < 0) {
        parent.left = newNode;
    } else {
        parent.right = newNode;
    }
    
    newNode.left = NIL;
    newNode.right = NIL;
    newNode.color = Color.RED;
    
    // Fix red-black violations
    insertFixup(newNode);
    size++;
}

private void insertFixup(RBNode<T> node) {
    while (node.parent.color == Color.RED) {
        if (node.parent == node.parent.parent.left) {
            RBNode<T> uncle = node.parent.parent.right;
            
            if (uncle.color == Color.RED) {
                // Case 1: Uncle is red - recolor
                node.parent.color = Color.BLACK;
                uncle.color = Color.BLACK;
                node.parent.parent.color = Color.RED;
                node = node.parent.parent;
            } else {
                if (node == node.parent.right) {
                    // Case 2: Uncle is black, node is right child
                    node = node.parent;
                    rotateLeft(node);
                }
                // Case 3: Uncle is black, node is left child
                node.parent.color = Color.BLACK;
                node.parent.parent.color = Color.RED;
                rotateRight(node.parent.parent);
            }
        } else {
            // Symmetric cases for right side
            RBNode<T> uncle = node.parent.parent.left;
            
            if (uncle.color == Color.RED) {
                node.parent.color = Color.BLACK;
                uncle.color = Color.BLACK;
                node.parent.parent.color = Color.RED;
                node = node.parent.parent;
            } else {
                if (node == node.parent.left) {
                    node = node.parent;
                    rotateRight(node);
                }
                node.parent.color = Color.BLACK;
                node.parent.parent.color = Color.RED;
                rotateLeft(node.parent.parent);
            }
        }
    }
    root.color = Color.BLACK; // Root is always black
}
```

#### Delete Operation:

```java
public void delete(T key) {
    RBNode<T> nodeToDelete = searchNode(root, key);
    if (nodeToDelete == NIL) return;
    
    RBNode<T> y = nodeToDelete;
    RBNode<T> x;
    Color originalColor = y.color;
    
    if (nodeToDelete.left == NIL) {
        x = nodeToDelete.right;
        transplant(nodeToDelete, nodeToDelete.right);
    } else if (nodeToDelete.right == NIL) {
        x = nodeToDelete.left;
        transplant(nodeToDelete, nodeToDelete.left);
    } else {
        y = findSuccessor(nodeToDelete.right);
        originalColor = y.color;
        x = y.right;
        
        if (y.parent == nodeToDelete) {
            x.parent = y;
        } else {
            transplant(y, y.right);
            y.right = nodeToDelete.right;
            y.right.parent = y;
        }
        
        transplant(nodeToDelete, y);
        y.left = nodeToDelete.left;
        y.left.parent = y;
        y.color = nodeToDelete.color;
    }
    
    if (originalColor == Color.BLACK) {
        deleteFixup(x);
    }
    size--;
}

private void deleteFixup(RBNode<T> x) {
    while (x != root && x.color == Color.BLACK) {
        if (x == x.parent.left) {
            RBNode<T> sibling = x.parent.right;
            
            // Case 1: Sibling is red
            if (sibling.color == Color.RED) {
                sibling.color = Color.BLACK;
                x.parent.color = Color.RED;
                rotateLeft(x.parent);
                sibling = x.parent.right;
            }
            
            // Case 2: Sibling is black with black children
            if (sibling.left.color == Color.BLACK && 
                sibling.right.color == Color.BLACK) {
                sibling.color = Color.RED;
                x = x.parent;
            } else {
                // Case 3: Sibling is black, left child red, right child black
                if (sibling.right.color == Color.BLACK) {
                    sibling.left.color = Color.BLACK;
                    sibling.color = Color.RED;
                    rotateRight(sibling);
                    sibling = x.parent.right;
                }
                
                // Case 4: Sibling is black with red right child
                sibling.color = x.parent.color;
                x.parent.color = Color.BLACK;
                sibling.right.color = Color.BLACK;
                rotateLeft(x.parent);
                x = root;
            }
        } else {
            // Symmetric cases for right side
            // ... (similar logic with left/right swapped)
        }
    }
    x.color = Color.BLACK;
}
```

### Performance Characteristics:

|Operation|Time Complexity|Space Complexity|
|---|---|---|
|Search|O(log n)|O(1)|
|Insert|O(log n)|O(1)|
|Delete|O(log n)|O(1)|
|Min/Max|O(log n)|O(1)|
|Traversal|O(n)|O(log n) recursion|

### Red-Black vs AVL Comparison:

|Property|Red-Black Tree|AVL Tree|
|---|---|---|
|Balance Factor|Color constraints|Height difference ≤ 1|
|Height|≤ 2⋅log₂(n+1)|≤ 1.44⋅log₂(n+2)|
|Insertions|Fewer rotations|More rotations|
|Deletions|Fewer rotations|More rotations|
|Lookups|Slightly slower|Faster|
|Memory|Extra bit for color|Extra space for height|

### Applications:

- **Java TreeMap/TreeSet**: Red-Black tree implementation
- **C++ std::map/std::set**: Usually Red-Black tree
- **Linux kernel**: Process scheduling, memory management
- **Database indexing**: B+ trees (variant of Red-Black)

### Key Advantages:

1. **Guaranteed O(log n)** worst-case performance
2. **Fewer rotations** compared to AVL during modifications
3. **Good balance** between search speed and update cost
4. **Widely used** in production systems
5. **Simpler implementation** than some alternatives
