# Chapter 12. Tree Basics (트리 기초)

> Understanding tree data structure terminology, binary trees, and tree traversal algorithms.

---

## Tree Terminology (트리 용어)

### Concept (개념)

A **Tree** (트리) is a hierarchical data structure consisting of nodes connected by edges. Unlike arrays and linked lists, trees are non-linear data structures.

```
                    [A]           <-- Root (루트)
                   / | \
                  /  |  \
               [B]  [C]  [D]      <-- Internal Nodes (내부 노드)
               / \       / \
             [E] [F]   [G] [H]    <-- Leaf Nodes (리프/단말 노드)
```

### Tree Terms (트리 용어 정의)

```c
#include <stdio.h>
#include <stdlib.h>

// Basic tree node structure
typedef struct TreeNode {
    int data;
    struct TreeNode* firstChild;   // First child
    struct TreeNode* nextSibling;  // Next sibling
} TreeNode;

int main(void) {
    printf("=== Tree Terminology (트리 용어) ===\n\n");

    printf("| Term (English)    | Term (Korean) | Definition                            |\n");
    printf("|-------------------|---------------|---------------------------------------|\n");
    printf("| Root              | 루트          | Topmost node with no parent           |\n");
    printf("| Node              | 노드          | Element containing data               |\n");
    printf("| Edge              | 간선          | Connection between nodes              |\n");
    printf("| Parent            | 부모          | Node with children below it           |\n");
    printf("| Child             | 자식          | Node directly below a parent          |\n");
    printf("| Sibling           | 형제          | Nodes sharing same parent             |\n");
    printf("| Leaf              | 리프/단말     | Node with no children                 |\n");
    printf("| Internal Node     | 내부 노드     | Node with at least one child          |\n");
    printf("| Ancestor          | 조상          | Any node on path to root              |\n");
    printf("| Descendant        | 자손          | Any node in subtree below             |\n");
    printf("| Depth             | 깊이          | Distance from root (root = 0)         |\n");
    printf("| Height            | 높이          | Max depth of any node                 |\n");
    printf("| Level             | 레벨          | Set of nodes at same depth            |\n");
    printf("| Degree            | 차수          | Number of children of a node          |\n");
    printf("| Subtree           | 서브트리      | Tree formed by node and descendants   |\n");

    return 0;
}
```

**Visual Explanation of Terms:**
```
                       Height = 3
           Level 0:        [A]  <-- Root, Depth=0
                          /   \
           Level 1:     [B]   [C]  <-- Depth=1, Siblings
                       / \     |
           Level 2:  [D] [E]  [F]  <-- Depth=2
                     |         |
           Level 3: [G]       [H]  <-- Leaves, Depth=3

Node Analysis for 'B':
  - Parent: A
  - Children: D, E
  - Siblings: C
  - Ancestors: A
  - Descendants: D, E, G
  - Depth: 1
  - Degree: 2

Leaf nodes: G, E, H (degree = 0)
Internal nodes: A, B, C, D, F (degree >= 1)
```

---

## Binary Tree (이진 트리)

### Concept (개념)

A **Binary Tree** (이진 트리) is a tree where each node has at most two children, referred to as **left child** and **right child**.

```c
#include <stdio.h>
#include <stdlib.h>

// Binary Tree Node
typedef struct BTNode {
    int data;
    struct BTNode* left;   // Left child (왼쪽 자식)
    struct BTNode* right;  // Right child (오른쪽 자식)
} BTNode;

// Create a new node
BTNode* createNode(int data) {
    BTNode* newNode = (BTNode*)malloc(sizeof(BTNode));
    if (newNode == NULL) {
        printf("Memory allocation failed!\n");
        return NULL;
    }
    newNode->data = data;
    newNode->left = NULL;
    newNode->right = NULL;
    return newNode;
}

// Calculate height of tree
int getHeight(BTNode* node) {
    if (node == NULL) {
        return -1;  // Height of empty tree is -1
    }
    int leftHeight = getHeight(node->left);
    int rightHeight = getHeight(node->right);
    return 1 + (leftHeight > rightHeight ? leftHeight : rightHeight);
}

// Count total nodes
int countNodes(BTNode* node) {
    if (node == NULL) {
        return 0;
    }
    return 1 + countNodes(node->left) + countNodes(node->right);
}

// Count leaf nodes
int countLeaves(BTNode* node) {
    if (node == NULL) {
        return 0;
    }
    if (node->left == NULL && node->right == NULL) {
        return 1;  // This is a leaf
    }
    return countLeaves(node->left) + countLeaves(node->right);
}

// Free tree memory
void freeTree(BTNode* node) {
    if (node == NULL) return;
    freeTree(node->left);
    freeTree(node->right);
    free(node);
}

int main(void) {
    printf("=== Binary Tree (이진 트리) ===\n\n");

    // Build a sample binary tree
    //        1
    //       / \
    //      2   3
    //     / \   \
    //    4   5   6

    BTNode* root = createNode(1);
    root->left = createNode(2);
    root->right = createNode(3);
    root->left->left = createNode(4);
    root->left->right = createNode(5);
    root->right->right = createNode(6);

    printf("Tree Structure:\n");
    printf("       1\n");
    printf("      / \\\n");
    printf("     2   3\n");
    printf("    / \\   \\\n");
    printf("   4   5   6\n\n");

    printf("Tree Properties:\n");
    printf("  Total nodes: %d\n", countNodes(root));
    printf("  Height: %d\n", getHeight(root));
    printf("  Leaf nodes: %d\n", countLeaves(root));

    freeTree(root);
    return 0;
}
```

### Types of Binary Trees (이진 트리 종류)

```
1. Full Binary Tree (정이진 트리)
   - Every node has 0 or 2 children
         1
        / \
       2   3
      / \
     4   5

2. Complete Binary Tree (완전 이진 트리)
   - All levels filled except possibly last
   - Last level filled from left to right
         1
        / \
       2   3
      / \  /
     4  5 6

3. Perfect Binary Tree (포화 이진 트리)
   - All internal nodes have 2 children
   - All leaves at same level
         1
        / \
       2   3
      / \ / \
     4  5 6  7

4. Degenerate/Skewed Tree (편향 트리)
   - Each node has only one child
     1
      \
       2
        \
         3
          \
           4
```

---

## Tree Traversal (트리 순회)

### Concept (개념)

**Tree Traversal** (트리 순회) is the process of visiting all nodes in a tree in a specific order.

There are four main traversal methods:
1. **Preorder** (전위 순회): Root -> Left -> Right
2. **Inorder** (중위 순회): Left -> Root -> Right
3. **Postorder** (후위 순회): Left -> Right -> Root
4. **Level-order** (레벨 순회): Level by level, left to right

```
Sample Tree for Traversal:
         1
        / \
       2   3
      / \   \
     4   5   6

Preorder:   1 -> 2 -> 4 -> 5 -> 3 -> 6  (Root first)
Inorder:    4 -> 2 -> 5 -> 1 -> 3 -> 6  (Left first)
Postorder:  4 -> 5 -> 2 -> 6 -> 3 -> 1  (Children first)
Level-order: 1 -> 2 -> 3 -> 4 -> 5 -> 6 (Level by level)
```

### Recursive Implementation (재귀 구현)

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct BTNode {
    int data;
    struct BTNode* left;
    struct BTNode* right;
} BTNode;

BTNode* createNode(int data) {
    BTNode* newNode = (BTNode*)malloc(sizeof(BTNode));
    newNode->data = data;
    newNode->left = NULL;
    newNode->right = NULL;
    return newNode;
}

// Preorder Traversal: Root -> Left -> Right
// 전위 순회: 루트 -> 왼쪽 -> 오른쪽
void preorder(BTNode* node) {
    if (node == NULL) return;

    printf("%d ", node->data);  // Visit root
    preorder(node->left);       // Traverse left subtree
    preorder(node->right);      // Traverse right subtree
}

// Inorder Traversal: Left -> Root -> Right
// 중위 순회: 왼쪽 -> 루트 -> 오른쪽
void inorder(BTNode* node) {
    if (node == NULL) return;

    inorder(node->left);        // Traverse left subtree
    printf("%d ", node->data);  // Visit root
    inorder(node->right);       // Traverse right subtree
}

// Postorder Traversal: Left -> Right -> Root
// 후위 순회: 왼쪽 -> 오른쪽 -> 루트
void postorder(BTNode* node) {
    if (node == NULL) return;

    postorder(node->left);      // Traverse left subtree
    postorder(node->right);     // Traverse right subtree
    printf("%d ", node->data);  // Visit root
}

void freeTree(BTNode* node) {
    if (node == NULL) return;
    freeTree(node->left);
    freeTree(node->right);
    free(node);
}

int main(void) {
    printf("=== Tree Traversal with Recursion (재귀를 이용한 트리 순회) ===\n\n");

    // Build tree
    //        1
    //       / \
    //      2   3
    //     / \   \
    //    4   5   6

    BTNode* root = createNode(1);
    root->left = createNode(2);
    root->right = createNode(3);
    root->left->left = createNode(4);
    root->left->right = createNode(5);
    root->right->right = createNode(6);

    printf("Tree Structure:\n");
    printf("       1\n");
    printf("      / \\\n");
    printf("     2   3\n");
    printf("    / \\   \\\n");
    printf("   4   5   6\n\n");

    printf("Preorder (전위):   ");
    preorder(root);
    printf("\n");

    printf("Inorder (중위):    ");
    inorder(root);
    printf("\n");

    printf("Postorder (후위):  ");
    postorder(root);
    printf("\n");

    freeTree(root);
    return 0;
}
```

**Visualization of Recursive Calls:**
```
Preorder(1):  Print 1
              +-> Preorder(2): Print 2
              |   +-> Preorder(4): Print 4
              |   |   +-> Preorder(NULL): return
              |   |   +-> Preorder(NULL): return
              |   +-> Preorder(5): Print 5
              |       +-> Preorder(NULL): return
              |       +-> Preorder(NULL): return
              +-> Preorder(3): Print 3
                  +-> Preorder(NULL): return
                  +-> Preorder(6): Print 6
                      +-> Preorder(NULL): return
                      +-> Preorder(NULL): return

Output: 1 2 4 5 3 6
```

### Level-order Traversal (레벨 순회)

Level-order traversal requires a **queue** data structure.

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct BTNode {
    int data;
    struct BTNode* left;
    struct BTNode* right;
} BTNode;

// Queue implementation for level-order traversal
typedef struct QueueNode {
    BTNode* treeNode;
    struct QueueNode* next;
} QueueNode;

typedef struct Queue {
    QueueNode* front;
    QueueNode* rear;
} Queue;

Queue* createQueue() {
    Queue* q = (Queue*)malloc(sizeof(Queue));
    q->front = q->rear = NULL;
    return q;
}

int isEmpty(Queue* q) {
    return q->front == NULL;
}

void enqueue(Queue* q, BTNode* node) {
    QueueNode* newNode = (QueueNode*)malloc(sizeof(QueueNode));
    newNode->treeNode = node;
    newNode->next = NULL;

    if (q->rear == NULL) {
        q->front = q->rear = newNode;
        return;
    }
    q->rear->next = newNode;
    q->rear = newNode;
}

BTNode* dequeue(Queue* q) {
    if (isEmpty(q)) return NULL;

    QueueNode* temp = q->front;
    BTNode* node = temp->treeNode;
    q->front = q->front->next;

    if (q->front == NULL) {
        q->rear = NULL;
    }
    free(temp);
    return node;
}

BTNode* createNode(int data) {
    BTNode* newNode = (BTNode*)malloc(sizeof(BTNode));
    newNode->data = data;
    newNode->left = NULL;
    newNode->right = NULL;
    return newNode;
}

// Level-order Traversal using Queue
// 레벨 순회 (큐 사용)
void levelOrder(BTNode* root) {
    if (root == NULL) return;

    Queue* q = createQueue();
    enqueue(q, root);

    while (!isEmpty(q)) {
        BTNode* current = dequeue(q);
        printf("%d ", current->data);

        if (current->left != NULL) {
            enqueue(q, current->left);
        }
        if (current->right != NULL) {
            enqueue(q, current->right);
        }
    }
    free(q);
}

// Level-order with level indication
void levelOrderByLevel(BTNode* root) {
    if (root == NULL) return;

    Queue* q = createQueue();
    enqueue(q, root);
    int level = 0;

    while (!isEmpty(q)) {
        int levelSize = 0;

        // Count nodes at current level
        QueueNode* temp = q->front;
        while (temp != NULL) {
            levelSize++;
            temp = temp->next;
        }

        printf("Level %d: ", level);

        // Process all nodes at current level
        for (int i = 0; i < levelSize; i++) {
            BTNode* current = dequeue(q);
            printf("%d ", current->data);

            if (current->left != NULL) {
                enqueue(q, current->left);
            }
            if (current->right != NULL) {
                enqueue(q, current->right);
            }
        }
        printf("\n");
        level++;
    }
    free(q);
}

void freeTree(BTNode* node) {
    if (node == NULL) return;
    freeTree(node->left);
    freeTree(node->right);
    free(node);
}

int main(void) {
    printf("=== Level-order Traversal (레벨 순회) ===\n\n");

    // Build tree
    //        1
    //       / \
    //      2   3
    //     / \   \
    //    4   5   6

    BTNode* root = createNode(1);
    root->left = createNode(2);
    root->right = createNode(3);
    root->left->left = createNode(4);
    root->left->right = createNode(5);
    root->right->right = createNode(6);

    printf("Tree Structure:\n");
    printf("       1\n");
    printf("      / \\\n");
    printf("     2   3\n");
    printf("    / \\   \\\n");
    printf("   4   5   6\n\n");

    printf("Level-order (레벨 순회): ");
    levelOrder(root);
    printf("\n\n");

    printf("Level-order by Level:\n");
    levelOrderByLevel(root);

    freeTree(root);
    return 0;
}
```

**Level-order Traversal Process:**
```
Queue-based Level Order Traversal:

Initial:  Queue: [1]

Step 1:   Dequeue 1, print 1
          Enqueue children 2, 3
          Queue: [2, 3]
          Output: 1

Step 2:   Dequeue 2, print 2
          Enqueue children 4, 5
          Queue: [3, 4, 5]
          Output: 1 2

Step 3:   Dequeue 3, print 3
          Enqueue child 6
          Queue: [4, 5, 6]
          Output: 1 2 3

Step 4:   Dequeue 4, print 4
          No children
          Queue: [5, 6]
          Output: 1 2 3 4

Step 5:   Dequeue 5, print 5
          No children
          Queue: [6]
          Output: 1 2 3 4 5

Step 6:   Dequeue 6, print 6
          No children
          Queue: []
          Output: 1 2 3 4 5 6

Final Output: 1 2 3 4 5 6
```

---

## Traversal Applications (순회 활용)

### When to Use Which Traversal

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

typedef struct FileNode {
    char name[50];
    int size;
    struct FileNode* left;   // First child
    struct FileNode* right;  // Sibling
} FileNode;

typedef struct ExprNode {
    char op;                   // Operator or operand
    int value;                 // Value if operand
    struct ExprNode* left;
    struct ExprNode* right;
} ExprNode;

int main(void) {
    printf("=== Traversal Applications (순회 활용) ===\n\n");

    printf("1. PREORDER (전위 순회) - Copy tree, Prefix expression\n");
    printf("   Process root before children\n");
    printf("   Use case: Copy/clone a tree, create prefix notation\n\n");

    printf("   Expression Tree:    *\n");
    printf("                      / \\\n");
    printf("                     +   3\n");
    printf("                    / \\\n");
    printf("                   1   2\n");
    printf("   Prefix notation (Preorder): * + 1 2 3\n\n");

    printf("2. INORDER (중위 순회) - Sorted order in BST\n");
    printf("   Process left, then root, then right\n");
    printf("   Use case: Get sorted sequence from BST\n\n");

    printf("   BST:        5\n");
    printf("              / \\\n");
    printf("             3   7\n");
    printf("            / \\ / \\\n");
    printf("           1  4 6  8\n");
    printf("   Inorder: 1 3 4 5 6 7 8 (sorted!)\n\n");

    printf("3. POSTORDER (후위 순회) - Delete tree, Postfix expression\n");
    printf("   Process children before root\n");
    printf("   Use case: Delete tree, calculate directory size\n\n");

    printf("   Directory Tree:   /root\n");
    printf("                     /   \\\n");
    printf("                  /dir1  /dir2\n");
    printf("                   |       |\n");
    printf("                 file.c  data.txt\n");
    printf("   Postorder deletion: file.c -> dir1 -> data.txt -> dir2 -> root\n\n");

    printf("4. LEVEL-ORDER (레벨 순회) - BFS, Find shortest path\n");
    printf("   Process level by level\n");
    printf("   Use case: Find minimum depth, serialize tree\n\n");

    return 0;
}
```

### Complete Traversal Example with Expression Tree

```c
#include <stdio.h>
#include <stdlib.h>
#include <ctype.h>

typedef struct ExprNode {
    char data;  // Operator or operand
    struct ExprNode* left;
    struct ExprNode* right;
} ExprNode;

ExprNode* createExprNode(char data) {
    ExprNode* node = (ExprNode*)malloc(sizeof(ExprNode));
    node->data = data;
    node->left = NULL;
    node->right = NULL;
    return node;
}

// Preorder: Prefix notation (Polish notation)
void prefix(ExprNode* node) {
    if (node == NULL) return;
    printf("%c ", node->data);
    prefix(node->left);
    prefix(node->right);
}

// Inorder: Infix notation (normal math)
void infix(ExprNode* node) {
    if (node == NULL) return;
    if (node->left || node->right) printf("(");
    infix(node->left);
    printf("%c", node->data);
    infix(node->right);
    if (node->left || node->right) printf(")");
}

// Postorder: Postfix notation (Reverse Polish)
void postfix(ExprNode* node) {
    if (node == NULL) return;
    postfix(node->left);
    postfix(node->right);
    printf("%c ", node->data);
}

// Evaluate expression tree
int evaluate(ExprNode* node) {
    if (node == NULL) return 0;

    // If leaf node (operand)
    if (node->left == NULL && node->right == NULL) {
        return node->data - '0';  // Convert char to int
    }

    int leftVal = evaluate(node->left);
    int rightVal = evaluate(node->right);

    switch (node->data) {
        case '+': return leftVal + rightVal;
        case '-': return leftVal - rightVal;
        case '*': return leftVal * rightVal;
        case '/': return leftVal / rightVal;
    }
    return 0;
}

void freeTree(ExprNode* node) {
    if (node == NULL) return;
    freeTree(node->left);
    freeTree(node->right);
    free(node);
}

int main(void) {
    printf("=== Expression Tree Traversal (수식 트리 순회) ===\n\n");

    // Build expression tree for: (3 + 4) * 2
    //        *
    //       / \
    //      +   2
    //     / \
    //    3   4

    ExprNode* root = createExprNode('*');
    root->left = createExprNode('+');
    root->right = createExprNode('2');
    root->left->left = createExprNode('3');
    root->left->right = createExprNode('4');

    printf("Expression Tree:\n");
    printf("       *\n");
    printf("      / \\\n");
    printf("     +   2\n");
    printf("    / \\\n");
    printf("   3   4\n\n");

    printf("Prefix (Preorder):   ");
    prefix(root);
    printf("\n");

    printf("Infix (Inorder):     ");
    infix(root);
    printf("\n");

    printf("Postfix (Postorder): ");
    postfix(root);
    printf("\n\n");

    printf("Evaluation Result: %d\n", evaluate(root));
    printf("(3 + 4) * 2 = 7 * 2 = 14\n");

    freeTree(root);
    return 0;
}
```

---

## Traversal Comparison (순회 비교)

```
         1
        / \
       2   3
      / \   \
     4   5   6

+------------------+-------------------+----------------------------+
| Traversal        | Order             | Output                     |
+------------------+-------------------+----------------------------+
| Preorder (전위)  | Root->Left->Right | 1 -> 2 -> 4 -> 5 -> 3 -> 6 |
| Inorder (중위)   | Left->Root->Right | 4 -> 2 -> 5 -> 1 -> 3 -> 6 |
| Postorder (후위) | Left->Right->Root | 4 -> 5 -> 2 -> 6 -> 3 -> 1 |
| Level-order      | Level by Level    | 1 -> 2 -> 3 -> 4 -> 5 -> 6 |
+------------------+-------------------+----------------------------+

Time Complexity: O(n) for all traversals
Space Complexity: O(h) where h is height of tree
                  O(log n) for balanced tree
                  O(n) for skewed tree
```

---

## Summary (요약)

| Topic | Key Points |
|-------|------------|
| Tree Terms | Root, Node, Leaf, Parent, Child, Depth, Height |
| Binary Tree | Each node has at most 2 children (left, right) |
| Preorder | Root -> Left -> Right, good for copying trees |
| Inorder | Left -> Root -> Right, sorted order for BST |
| Postorder | Left -> Right -> Root, good for deletion |
| Level-order | Level by level using queue, BFS approach |

### Traversal Selection Guide (순회 선택 가이드)

```
What do you need to do?
        |
        +-> Copy/Clone tree?        --> Use Preorder
        |
        +-> Get sorted data (BST)?  --> Use Inorder
        |
        +-> Delete tree?            --> Use Postorder
        |
        +-> Find by level/depth?    --> Use Level-order
        |
        +-> Evaluate expression?    --> Use Postorder
```

### Time and Space Complexity

| Traversal | Time | Space (Balanced) | Space (Skewed) |
|-----------|------|------------------|----------------|
| Preorder | O(n) | O(log n) | O(n) |
| Inorder | O(n) | O(log n) | O(n) |
| Postorder | O(n) | O(log n) | O(n) |
| Level-order | O(n) | O(n) | O(1) |

---

## Practice Exercises (연습 문제)

1. Implement a function to find the maximum element in a binary tree.
2. Write a function to check if two binary trees are identical.
3. Implement mirror/invert of a binary tree.
4. Find the lowest common ancestor (LCA) of two nodes.
5. Convert a binary tree to its mirror image using postorder traversal.
