---
title: "Depth First Search in Java"
categories:
  - Data Structures
tags:
  - Algorithms
  - Java
  - DFS
  - Graph Traversal
  - Trees
---

### Depth First Search [DFS]

Imagine you are searching for a file buried somewhere deep in a nested folder structure. You would not check every folder at the top level first. Instead, you would pick a folder and keep opening subfolders until you hit a dead end, then backtrack and try the next path. That is exactly how **Depth First Search** works.

DFS is one of two fundamental traversal algorithms (the other being Breadth-First Search). It explores as deep as possible along each branch before backtracking. DFS applies to both trees and graphs, though the implementation differs slightly: graph DFS needs to track visited nodes to avoid revisiting them in cyclic structures. This post covers both.

If you want to brush up on the underlying data structures first:
* [Trees](/posts/trees/)
* [Graphs](/posts/graphs/)

### Tree DFS Traversals

There are three different orders for traversing a tree using DFS:

1. **Inorder Traversal** (Left, Root, Right)
2. **Preorder Traversal** (Root, Left, Right)
3. **Postorder Traversal** (Left, Right, Root)

Each version is recursive by nature, but can also be implemented iteratively using an explicit **Stack**. The iterative versions essentially simulate what the call stack does for you during recursion. Each recursive call pushes a frame onto the call stack; the iterative approach makes this explicit by pushing nodes onto a `Stack` data structure. Understanding this connection is useful because it shows that recursion and iteration with a stack are fundamentally interchangeable.

#### Inorder Traversal

For inorder traversal, we traverse the left subtree first, then the root, then finally the right subtree.

Inorder &#8594; Left, Root, Right.

Inorder traversal for a BST means that we're traversing the nodes in increasing order of their values.

**Time complexity:** O(n), where n is the number of nodes.
**Space complexity:** O(h), where h is the height of the tree (O(log n) for balanced trees, O(n) for skewed trees).

There are two ways of implementing it, recursive and iterative.

**Recursive:**
```java
public void inorderRecursive(Node root) {
    if(root != null) {
        inorderRecursive(root.left);
        visit(root.value);
        inorderRecursive(root.right);
    }
}
```

**Iterative:**

To implement it iteratively we will need a Stack and following steps:

* Initialize a current node with root
* While current is not null or stack is not empty
  * Keep pushing left child onto a stack, till we reach a current node's left-most child
  * Pop and visit the left-most node from stack
  * Set current to the right child of the popped node


```java
public void inorderIterative(Node root) {
    Stack<Node> stack = new Stack<Node>();
    Node current = root;

    while(!stack.isEmpty() || current != null) {
        while (current != null) {
            stack.push(current);
            current = current.left;
        }

        Node top = stack.pop();
        visit(top.value);
        current = top.right;
    }
}
```

#### Preorder Traversal

For preorder traversal, we traverse the root first, then the left subtree, then finally the right subtree.

Preorder &#8594; Root, Left, Right.

**Time complexity:** O(n), where n is the number of nodes.
**Space complexity:** O(h), where h is the height of the tree.

**Recursive:**
```java
public void traversePreorderRecursive(Node root) {
    if(root != null) {
        visit(root.value);
        traversePreorderRecursive(root.left);
        traversePreorderRecursive(root.right);
    }
}
```

**Iterative:**

To implement it iteratively we will need a Stack and following steps:

* Push root in our stack
* While stack is not empty
    * Pop current node
    * Visit current node
    * Push right child, then left child to stack

```java
public void traversePreorderIterative(Node root) {
    Stack<Node> stack = new Stack<Node>();
    Node current = root;
    stack.push(root);
    while(!stack.isEmpty()) {
        current = stack.pop();
        visit(current.value);

        if(current.right != null) {
            stack.push(current.right);
        }
        if(current.left != null) {
            stack.push(current.left);
        }
    }
}
```

#### Postorder Traversal

For postorder traversal, we traverse the left subtree first, then the right subtree, and finally the root.

Post order &#8594; Left, Right, Root.

**Time complexity:** O(n), where n is the number of nodes.
**Space complexity:** O(h), where h is the height of the tree.

**Recursive:**
```java
public void traversePostorderRecursive(Node root) {
    if (root != null) {
        traversePostorderRecursive(root.left);
        traversePostorderRecursive(root.right);
        visit(root.value);
    }
}
```

**Iterative:**

To implement postorder traversal without recursion:

* Push root node in stack
* While stack is not empty
    * Check if we already traversed left and right subtree
    * If not then push right child and left child onto stack

```java
public void traversePostorderIterative(Node root) {
    Stack<Node> stack = new Stack<Node>();
    Node prev = root;
    Node current = root;
    stack.push(root);

    while (!stack.isEmpty()) {
        current = stack.peek();
        boolean hasChild = (current.left != null || current.right != null);
        boolean isPrevLastChild = (prev == current.right ||
          (prev == current.left && current.right == null));

        if (!hasChild || isPrevLastChild) {
            current = stack.pop();
            visit(current.value);
            prev = current;
        } else {
            if (current.right != null) {
                stack.push(current.right);
            }
            if (current.left != null) {
                stack.push(current.left);
            }
        }
    }
}
```

### Graph DFS

Tree DFS is straightforward because trees are acyclic by definition. Every path leads to a leaf, and there is no way to visit the same node twice. Graphs are different. A graph can have cycles, which means a naive DFS would revisit the same nodes forever and never terminate.

The solution is a **visited set**. Before exploring a node, we check whether we have already seen it. If so, we skip it. This single addition turns tree DFS into graph DFS.

Here is a graph DFS implementation using an adjacency list representation:

**Recursive:**
```java
public void dfs(Map<Integer, List<Integer>> graph, int start) {
    Set<Integer> visited = new HashSet<>();
    dfsHelper(graph, start, visited);
}

private void dfsHelper(Map<Integer, List<Integer>> graph, int node, Set<Integer> visited) {
    visited.add(node);
    visit(node);

    for (int neighbor : graph.getOrDefault(node, Collections.emptyList())) {
        if (!visited.contains(neighbor)) {
            dfsHelper(graph, neighbor, visited);
        }
    }
}
```

**Iterative:**
```java
public void dfsIterative(Map<Integer, List<Integer>> graph, int start) {
    Set<Integer> visited = new HashSet<>();
    Stack<Integer> stack = new Stack<>();

    stack.push(start);

    while (!stack.isEmpty()) {
        int node = stack.pop();

        if (visited.contains(node)) {
            continue;
        }

        visited.add(node);
        visit(node);

        for (int neighbor : graph.getOrDefault(node, Collections.emptyList())) {
            if (!visited.contains(neighbor)) {
                stack.push(neighbor);
            }
        }
    }
}
```

The time complexity for graph DFS is **O(V + E)**, where V is the number of vertices and E is the number of edges. We visit each vertex once and examine each edge once. The space complexity is **O(V)** for the visited set plus the recursion stack (or explicit stack).

### DFS Limitations

DFS is a powerful tool, but it has real limitations worth understanding:

- **Infinite loops without a visited set.** On a cyclic graph, DFS without visited tracking will loop forever. This is probably the most common bug when implementing graph DFS for the first time.
- **No shortest path guarantee.** DFS finds *a* path, not necessarily the *shortest* path. If you need the shortest path in an unweighted graph, BFS is the right choice.
- **Stack overflow on deep structures.** Recursive DFS on a very deep graph (or a skewed tree) can blow the call stack. The iterative version avoids this by using a heap-allocated stack instead.
- **Not ideal for level-order problems.** Anything that requires processing nodes level by level (finding all nodes at depth k, for example) is naturally suited to BFS, not DFS.

### DFS vs BFS

DFS and **Breadth-First Search (BFS)** are the two fundamental approaches to tree and graph traversal. DFS dives deep into one branch before exploring siblings, while BFS explores all nodes at the current depth level before moving deeper.

In terms of complexity, both run in O(V + E) time for graphs and O(n) for trees, but their space usage differs: DFS uses O(h) space (the height of the tree or the longest path in a graph), while BFS uses O(w) space (the maximum width). For a balanced binary tree, the width can be up to n/2 at the deepest level, making BFS more memory-intensive in that scenario.

#### A Concrete Example

Consider a social network where you want to find whether a connection exists between two users.

**Use DFS when:** you want to check if any path exists between user A and user B, or when searching for something that might be far from the starting point. For instance, checking if two users are connected at all (even through many degrees of separation). DFS will quickly drill through chains of connections.

**Use BFS when:** you want to find the shortest connection between two users. If you need to answer "what is the smallest number of hops between user A and user B?" (think LinkedIn's "2nd degree connection" feature), BFS is the only correct choice among the two. BFS explores all 1st-degree connections first, then all 2nd-degree, and so on, so the first time it reaches the target, it has found the shortest path.

Another way to think about it: DFS is like exploring a maze by always turning left until you hit a dead end, then backtracking. BFS is like a spreading flood that fills every corridor at the same distance before moving further. The maze explorer finds *an* exit; the flood finds the *closest* exit.

> **Related posts**: [Graphs](/posts/graphs/), [Trees](/posts/trees/), [Stacks and Queues](/posts/stacks-and-queues/)
