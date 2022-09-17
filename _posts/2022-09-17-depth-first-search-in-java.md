---
title: "Depth First Search in Java"
categories:
  - Blog
tags:
  - Algorithms
  - Java
---

### Depth First Search \[DFS]

It's very popular traversal algorithm that is really worth knowing, its used in for both Tree and Graph data structures. 
The depth-first search goes deep in each branch before moving to explore another branch.

Here you could get familiar with 
* [Trees](https://matthewonsoftware.com/) 
* [Graphs](https://matthewonsoftware.com/blog/graphs) 

data structures.



#### Different types of implementation of DFS

There are three different orders for traversing a tree using DFS:

1. Inorder Traversal
2. Preorder Traversal
3. Postorder Traversal

<br>

#### Tree Depth First Search 

##### Inorder Traversal

For inorder traversal, we traverse the left subtree first, then the root, then finally the right subtree.

Inorder &#8594; Left, Root, Right.

Inorder traversal for a BST means that we're traversing the nodes in increasing order of their values.

There are two ways of implementing it, recursive and iterative.

**Recursive:**
```java
public void inorderRecursive(Node root) {
    if(root != null) {
        dfsTraverseInorder(root.left);
        visit(root.value)
        dfsTraverseInorder(root.right);
    }
```

<br>

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

<br>

##### Preorder Traversal

For preorder traversal, we traverse the root first, then the left subtree, then finally the right subtree.

Preorder &#8594; Root, Left, Right.

To implement it iteratively we will need a Stack and following steps:

* Push root in our stack
* While stack is not empty
    * Pop current node
    * Visit current node
    * Push right child, then left child to stack

**Recursive:**
```java
public void traversePreorderRecursive(Node root) {
    if(root != null) {
        visit(root.value);
        dfsTraversePreorder(root.left);
        dfsTraversePreorder(root.right);
    }
}
```


<br>

**Iterative:**

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

For postorder traversal, we traverse the left subtree first, then finally the right subtree, then finally the root.

Post order &#8594; Left, Right, Root.

To implement postorder traversal without recursion:

* Push root node in stack
* While stack is not empty
    * Check if we already traversed left and right subtree
    * If not then push right child and left child onto stack

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


<br>

**Iterative:**

```java
public void traversePostorderIterative() {
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
