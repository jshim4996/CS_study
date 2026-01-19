# Chapter 13. Binary Search Tree (이진 탐색 트리)

> Understanding Binary Search Tree concept, operations (insert, search, delete), and complexity analysis.

---

## BST Concept (BST 개념)

### Concept (개념)

A **Binary Search Tree (BST)** (이진 탐색 트리) is a binary tree with the following properties:

1. Each node has at most two children
2. **Left subtree** contains only nodes with values **less than** the parent
3. **Right subtree** contains only nodes with values **greater than** the parent
4. Both left and right subtrees are also BSTs

```
BST Property:
            node
           /    \
     < node      > node
     (left)      (right)
```

**Example of Valid BST:**
```
           50
          /  \
        30    70
       /  \   /  \
     20   40 60   80

For node 50:
  - Left subtree (30, 20, 40): All values < 50
  - Right subtree (70, 60, 80): All values > 50

For node 30:
  - Left subtree (20): 20 < 30
  - Right subtree (40): 40 > 30
```

**NOT a Valid BST:**
```
           50
          /  \
        30    70
       /  \
     20   60    <-- INVALID! 60 > 50 but in left subtree
```

### BST Node Structure

```c
#include <stdio.h>
#include <stdlib.h>

// BST Node Structure
typedef struct BSTNode {
    int data;
    struct BSTNode* left;
    struct BSTNode* right;
} BSTNode;

// Create a new node
BSTNode* createNode(int data) {
    BSTNode* newNode = (BSTNode*)malloc(sizeof(BSTNode));
    if (newNode == NULL) {
        printf("Memory allocation failed!\n");
        return NULL;
    }
    newNode->data = data;
    newNode->left = NULL;
    newNode->right = NULL;
    return newNode;
}

// Check if tree is valid BST
int isBSTUtil(BSTNode* node, int min, int max) {
    if (node == NULL) return 1;  // Empty tree is BST

    // Check current node
    if (node->data <= min || node->data >= max) {
        return 0;
    }

    // Check subtrees with updated bounds
    return isBSTUtil(node->left, min, node->data) &&
           isBSTUtil(node->right, node->data, max);
}

int isBST(BSTNode* root) {
    return isBSTUtil(root, INT_MIN, INT_MAX);
}

int main(void) {
    printf("=== BST Concept (BST 개념) ===\n\n");

    printf("BST Property (BST 속성):\n");
    printf("1. Left child < Parent\n");
    printf("2. Right child > Parent\n");
    printf("3. No duplicate values (typically)\n\n");

    printf("Advantages of BST:\n");
    printf("- Efficient search: O(log n) average\n");
    printf("- Ordered data storage\n");
    printf("- Dynamic size\n");
    printf("- Easy range queries\n");

    return 0;
}
```

---

## BST Insert (BST 삽입)

### Concept (개념)

To insert a new value into a BST:
1. Start at the root
2. Compare the value with current node
3. If smaller, go left; if larger, go right
4. Repeat until finding an empty spot
5. Insert the new node at that spot

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct BSTNode {
    int data;
    struct BSTNode* left;
    struct BSTNode* right;
} BSTNode;

BSTNode* createNode(int data) {
    BSTNode* newNode = (BSTNode*)malloc(sizeof(BSTNode));
    newNode->data = data;
    newNode->left = NULL;
    newNode->right = NULL;
    return newNode;
}

// Recursive Insert
// 재귀적 삽입
BSTNode* insert(BSTNode* root, int data) {
    // Base case: empty tree or found insertion point
    if (root == NULL) {
        return createNode(data);
    }

    // Recursive case: navigate to correct position
    if (data < root->data) {
        root->left = insert(root->left, data);
    } else if (data > root->data) {
        root->right = insert(root->right, data);
    }
    // If data == root->data, ignore duplicate (or handle as needed)

    return root;
}

// Iterative Insert
// 반복적 삽입
BSTNode* insertIterative(BSTNode* root, int data) {
    BSTNode* newNode = createNode(data);

    // Empty tree
    if (root == NULL) {
        return newNode;
    }

    BSTNode* current = root;
    BSTNode* parent = NULL;

    // Find insertion point
    while (current != NULL) {
        parent = current;
        if (data < current->data) {
            current = current->left;
        } else if (data > current->data) {
            current = current->right;
        } else {
            // Duplicate found
            free(newNode);
            return root;
        }
    }

    // Insert at found position
    if (data < parent->data) {
        parent->left = newNode;
    } else {
        parent->right = newNode;
    }

    return root;
}

// Inorder traversal to verify BST
void inorder(BSTNode* root) {
    if (root == NULL) return;
    inorder(root->left);
    printf("%d ", root->data);
    inorder(root->right);
}

// Print tree structure (rotated 90 degrees)
void printTree(BSTNode* root, int level) {
    if (root == NULL) return;

    printTree(root->right, level + 1);

    for (int i = 0; i < level; i++) {
        printf("    ");
    }
    printf("%d\n", root->data);

    printTree(root->left, level + 1);
}

void freeTree(BSTNode* root) {
    if (root == NULL) return;
    freeTree(root->left);
    freeTree(root->right);
    free(root);
}

int main(void) {
    printf("=== BST Insert (BST 삽입) ===\n\n");

    BSTNode* root = NULL;

    // Insert values: 50, 30, 70, 20, 40, 60, 80
    int values[] = {50, 30, 70, 20, 40, 60, 80};
    int n = sizeof(values) / sizeof(values[0]);

    printf("Inserting values: ");
    for (int i = 0; i < n; i++) {
        printf("%d ", values[i]);
        root = insert(root, values[i]);
    }
    printf("\n\n");

    printf("BST Structure (rotated 90 degrees):\n");
    printTree(root, 0);

    printf("\nInorder Traversal (should be sorted): ");
    inorder(root);
    printf("\n");

    freeTree(root);
    return 0;
}
```

**Insert Visualization:**
```
Inserting: 50, 30, 70, 20, 40, 60, 80

Step 1: Insert 50 (root)
        50

Step 2: Insert 30 (30 < 50, go left)
        50
       /
      30

Step 3: Insert 70 (70 > 50, go right)
        50
       /  \
      30   70

Step 4: Insert 20 (20 < 50, 20 < 30, go left)
        50
       /  \
      30   70
     /
    20

Step 5: Insert 40 (40 < 50, 40 > 30, go right)
        50
       /  \
      30   70
     /  \
    20  40

Step 6: Insert 60 (60 > 50, 60 < 70, go left)
        50
       /  \
      30   70
     /  \  /
    20  40 60

Step 7: Insert 80 (80 > 50, 80 > 70, go right)
        50
       /  \
      30   70
     /  \  /  \
    20  40 60  80
```

---

## BST Search (BST 탐색)

### Concept (개념)

Searching in a BST is efficient because we can eliminate half of the remaining tree at each step:
1. Start at root
2. If target equals current node, found!
3. If target < current, search left subtree
4. If target > current, search right subtree
5. If reach NULL, not found

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct BSTNode {
    int data;
    struct BSTNode* left;
    struct BSTNode* right;
} BSTNode;

BSTNode* createNode(int data) {
    BSTNode* newNode = (BSTNode*)malloc(sizeof(BSTNode));
    newNode->data = data;
    newNode->left = NULL;
    newNode->right = NULL;
    return newNode;
}

BSTNode* insert(BSTNode* root, int data) {
    if (root == NULL) return createNode(data);
    if (data < root->data) root->left = insert(root->left, data);
    else if (data > root->data) root->right = insert(root->right, data);
    return root;
}

// Recursive Search
// 재귀적 탐색
BSTNode* search(BSTNode* root, int target) {
    // Base cases
    if (root == NULL || root->data == target) {
        return root;
    }

    // Recursive cases
    if (target < root->data) {
        return search(root->left, target);
    }
    return search(root->right, target);
}

// Iterative Search
// 반복적 탐색
BSTNode* searchIterative(BSTNode* root, int target) {
    BSTNode* current = root;

    while (current != NULL) {
        if (target == current->data) {
            return current;  // Found
        } else if (target < current->data) {
            current = current->left;
        } else {
            current = current->right;
        }
    }

    return NULL;  // Not found
}

// Search with path tracking
BSTNode* searchWithPath(BSTNode* root, int target) {
    printf("Search path for %d: ", target);

    BSTNode* current = root;
    while (current != NULL) {
        printf("%d", current->data);

        if (target == current->data) {
            printf(" (FOUND)\n");
            return current;
        }

        printf(" -> ");

        if (target < current->data) {
            current = current->left;
        } else {
            current = current->right;
        }
    }

    printf("NULL (NOT FOUND)\n");
    return NULL;
}

// Find minimum value node
BSTNode* findMin(BSTNode* root) {
    if (root == NULL) return NULL;

    BSTNode* current = root;
    while (current->left != NULL) {
        current = current->left;
    }
    return current;
}

// Find maximum value node
BSTNode* findMax(BSTNode* root) {
    if (root == NULL) return NULL;

    BSTNode* current = root;
    while (current->right != NULL) {
        current = current->right;
    }
    return current;
}

void freeTree(BSTNode* root) {
    if (root == NULL) return;
    freeTree(root->left);
    freeTree(root->right);
    free(root);
}

int main(void) {
    printf("=== BST Search (BST 탐색) ===\n\n");

    // Build BST
    BSTNode* root = NULL;
    int values[] = {50, 30, 70, 20, 40, 60, 80};
    for (int i = 0; i < 7; i++) {
        root = insert(root, values[i]);
    }

    printf("BST Structure:\n");
    printf("        50\n");
    printf("       /  \\\n");
    printf("      30   70\n");
    printf("     /  \\  /  \\\n");
    printf("    20  40 60  80\n\n");

    // Search tests
    searchWithPath(root, 40);
    searchWithPath(root, 60);
    searchWithPath(root, 25);
    searchWithPath(root, 80);

    printf("\nMinimum value: %d\n", findMin(root)->data);
    printf("Maximum value: %d\n", findMax(root)->data);

    freeTree(root);
    return 0;
}
```

**Search Visualization:**
```
BST:
        50
       /  \
      30   70
     /  \  /  \
    20  40 60  80

Search for 40:
  Step 1: 40 < 50 -> go left
          [50]
          /
         30

  Step 2: 40 > 30 -> go right
          50
         /
        [30]
           \
           40

  Step 3: 40 == 40 -> FOUND!
          50
         /
        30
          \
         [40]

Path: 50 -> 30 -> 40 (3 comparisons)

Search for 25:
  50 -> 30 -> 20 -> NULL (NOT FOUND)
```

---

## BST Delete (BST 삭제)

### Concept (개념)

Deleting a node from BST has **three cases**:

**Case 1: Leaf Node (No children)**
- Simply remove the node

**Case 2: Node with One Child**
- Replace node with its child

**Case 3: Node with Two Children**
- Find inorder successor (or predecessor)
- Replace node's data with successor's data
- Delete the successor

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct BSTNode {
    int data;
    struct BSTNode* left;
    struct BSTNode* right;
} BSTNode;

BSTNode* createNode(int data) {
    BSTNode* newNode = (BSTNode*)malloc(sizeof(BSTNode));
    newNode->data = data;
    newNode->left = NULL;
    newNode->right = NULL;
    return newNode;
}

BSTNode* insert(BSTNode* root, int data) {
    if (root == NULL) return createNode(data);
    if (data < root->data) root->left = insert(root->left, data);
    else if (data > root->data) root->right = insert(root->right, data);
    return root;
}

// Find minimum value node in subtree
BSTNode* findMin(BSTNode* root) {
    BSTNode* current = root;
    while (current && current->left != NULL) {
        current = current->left;
    }
    return current;
}

// Delete node from BST
// BST에서 노드 삭제
BSTNode* deleteNode(BSTNode* root, int data) {
    // Base case: empty tree
    if (root == NULL) {
        return root;
    }

    // Find the node to delete
    if (data < root->data) {
        root->left = deleteNode(root->left, data);
    } else if (data > root->data) {
        root->right = deleteNode(root->right, data);
    } else {
        // Found node to delete

        // Case 1: Leaf node (no children)
        // 경우 1: 리프 노드 (자식 없음)
        if (root->left == NULL && root->right == NULL) {
            free(root);
            return NULL;
        }

        // Case 2: Node with one child
        // 경우 2: 자식이 하나인 노드
        if (root->left == NULL) {
            BSTNode* temp = root->right;
            free(root);
            return temp;
        }
        if (root->right == NULL) {
            BSTNode* temp = root->left;
            free(root);
            return temp;
        }

        // Case 3: Node with two children
        // 경우 3: 자식이 둘인 노드
        // Find inorder successor (smallest in right subtree)
        BSTNode* successor = findMin(root->right);

        // Copy successor's data to this node
        root->data = successor->data;

        // Delete the successor
        root->right = deleteNode(root->right, successor->data);
    }

    return root;
}

void inorder(BSTNode* root) {
    if (root == NULL) return;
    inorder(root->left);
    printf("%d ", root->data);
    inorder(root->right);
}

void printTree(BSTNode* root, int level) {
    if (root == NULL) return;
    printTree(root->right, level + 1);
    for (int i = 0; i < level; i++) printf("    ");
    printf("%d\n", root->data);
    printTree(root->left, level + 1);
}

void freeTree(BSTNode* root) {
    if (root == NULL) return;
    freeTree(root->left);
    freeTree(root->right);
    free(root);
}

int main(void) {
    printf("=== BST Delete - Three Cases (BST 삭제 - 세 가지 경우) ===\n\n");

    // Build BST
    BSTNode* root = NULL;
    int values[] = {50, 30, 70, 20, 40, 60, 80, 35, 45};
    for (int i = 0; i < 9; i++) {
        root = insert(root, values[i]);
    }

    printf("Initial BST:\n");
    printf("          50\n");
    printf("         /  \\\n");
    printf("       30    70\n");
    printf("      /  \\   /  \\\n");
    printf("    20   40 60   80\n");
    printf("        /  \\\n");
    printf("       35  45\n\n");

    printf("Inorder: ");
    inorder(root);
    printf("\n\n");

    // Case 1: Delete leaf node (20)
    printf("=== Case 1: Delete Leaf Node (20) ===\n");
    printf("Node 20 has no children -> Simply remove\n");
    root = deleteNode(root, 20);
    printf("After deletion, Inorder: ");
    inorder(root);
    printf("\n\n");

    // Case 2: Delete node with one child (70 now has one child 80)
    // First delete 60 to set up the case
    root = deleteNode(root, 60);
    printf("=== Case 2: Delete Node with One Child (70) ===\n");
    printf("Node 70 has one child (80) -> Replace 70 with 80\n");
    root = deleteNode(root, 70);
    printf("After deletion, Inorder: ");
    inorder(root);
    printf("\n\n");

    // Rebuild tree for Case 3
    freeTree(root);
    root = NULL;
    for (int i = 0; i < 9; i++) {
        root = insert(root, values[i]);
    }

    printf("=== Case 3: Delete Node with Two Children (30) ===\n");
    printf("Node 30 has two children (20, 40)\n");
    printf("Find inorder successor: 35 (smallest in right subtree)\n");
    printf("Replace 30 with 35, then delete original 35\n");
    root = deleteNode(root, 30);
    printf("After deletion, Inorder: ");
    inorder(root);
    printf("\n\n");

    printf("Final BST Structure:\n");
    printTree(root, 0);

    freeTree(root);
    return 0;
}
```

**Delete Visualization - Three Cases:**
```
CASE 1: Delete Leaf Node (자식 없음)
========================================
Delete 20:

Before:         After:
    50            50
   /  \          /  \
  30   70       30   70
 / \            / \
20  40         X  40

Simply remove node 20


CASE 2: Delete Node with One Child (자식 하나)
========================================
Delete 70 (has only right child 80):

Before:         After:
    50            50
   /  \          /  \
  30   70       30   80
        \
        80

Replace 70 with its child 80


CASE 3: Delete Node with Two Children (자식 둘)
========================================
Delete 30:

Before:
        50
       /  \
      30   70
     /  \
    20   40
        /  \
       35  45

Step 1: Find inorder successor of 30
        -> Go to right child (40)
        -> Go left until NULL
        -> Successor is 35

Step 2: Copy 35 to 30's position

        50
       /  \
      35   70
     /  \
    20   40
        /  \
       35  45

Step 3: Delete original 35 (Case 1 - leaf)

After:
        50
       /  \
      35   70
     /  \
    20   40
          \
          45
```

---

## Time Complexity Analysis (시간 복잡도 분석)

### BST Operations Complexity

```c
#include <stdio.h>

int main(void) {
    printf("=== BST Time Complexity Analysis (BST 시간 복잡도 분석) ===\n\n");

    printf("| Operation | Average Case | Worst Case |\n");
    printf("|-----------|--------------|------------|\n");
    printf("| Search    | O(log n)     | O(n)       |\n");
    printf("| Insert    | O(log n)     | O(n)       |\n");
    printf("| Delete    | O(log n)     | O(n)       |\n");
    printf("| Min/Max   | O(log n)     | O(n)       |\n");
    printf("| Traversal | O(n)         | O(n)       |\n\n");

    printf("Space Complexity:\n");
    printf("  - O(n) for storing n nodes\n");
    printf("  - O(h) for recursive calls (h = height)\n\n");

    return 0;
}
```

**Balanced vs Unbalanced BST:**
```
Balanced BST (균형 BST):
Height = O(log n)

        50
       /  \
      25   75
     / \   / \
    10 30 60  90

Search for 60: 50 -> 75 -> 60 (3 comparisons)
For n = 7 nodes, height = 2, max comparisons = 3


Unbalanced/Skewed BST (편향 BST):
Height = O(n)

Insert in order: 10, 20, 30, 40, 50

    10
     \
      20
       \
        30
         \
          40
           \
            50

Search for 50: 10 -> 20 -> 30 -> 40 -> 50 (5 comparisons)
For n = 5 nodes, height = 4, max comparisons = 5
This is as bad as a linked list!
```

**When Does Worst Case Occur?**
```
Worst case O(n) occurs when tree becomes skewed:

1. Inserting sorted data:
   Insert: 1, 2, 3, 4, 5

   1 -> 1    -> 1      -> 1        -> 1
         \      \         \           \
          2      2         2           2
                  \         \           \
                   3         3           3
                              \           \
                               4           4
                                            \
                                             5

2. Inserting reverse sorted data:
   Insert: 5, 4, 3, 2, 1

   5 -> 5    -> 5      -> 5        -> 5
       /       /         /           /
      4       4         4           4
             /         /           /
            3         3           3
                     /           /
                    2           2
                               /
                              1

Solution: Use Self-Balancing BSTs (AVL, Red-Black Tree)
```

---

## Complete BST Implementation

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct BSTNode {
    int data;
    struct BSTNode* left;
    struct BSTNode* right;
} BSTNode;

// Create new node
BSTNode* createNode(int data) {
    BSTNode* newNode = (BSTNode*)malloc(sizeof(BSTNode));
    if (!newNode) return NULL;
    newNode->data = data;
    newNode->left = newNode->right = NULL;
    return newNode;
}

// Insert
BSTNode* insert(BSTNode* root, int data) {
    if (!root) return createNode(data);
    if (data < root->data)
        root->left = insert(root->left, data);
    else if (data > root->data)
        root->right = insert(root->right, data);
    return root;
}

// Search
BSTNode* search(BSTNode* root, int data) {
    if (!root || root->data == data) return root;
    if (data < root->data)
        return search(root->left, data);
    return search(root->right, data);
}

// Find minimum
BSTNode* findMin(BSTNode* root) {
    while (root && root->left)
        root = root->left;
    return root;
}

// Find maximum
BSTNode* findMax(BSTNode* root) {
    while (root && root->right)
        root = root->right;
    return root;
}

// Delete
BSTNode* deleteNode(BSTNode* root, int data) {
    if (!root) return NULL;

    if (data < root->data)
        root->left = deleteNode(root->left, data);
    else if (data > root->data)
        root->right = deleteNode(root->right, data);
    else {
        // Case 1 & 2
        if (!root->left) {
            BSTNode* temp = root->right;
            free(root);
            return temp;
        }
        if (!root->right) {
            BSTNode* temp = root->left;
            free(root);
            return temp;
        }
        // Case 3
        BSTNode* successor = findMin(root->right);
        root->data = successor->data;
        root->right = deleteNode(root->right, successor->data);
    }
    return root;
}

// Traversals
void inorder(BSTNode* root) {
    if (!root) return;
    inorder(root->left);
    printf("%d ", root->data);
    inorder(root->right);
}

void preorder(BSTNode* root) {
    if (!root) return;
    printf("%d ", root->data);
    preorder(root->left);
    preorder(root->right);
}

void postorder(BSTNode* root) {
    if (!root) return;
    postorder(root->left);
    postorder(root->right);
    printf("%d ", root->data);
}

// Count nodes
int countNodes(BSTNode* root) {
    if (!root) return 0;
    return 1 + countNodes(root->left) + countNodes(root->right);
}

// Calculate height
int height(BSTNode* root) {
    if (!root) return -1;
    int leftH = height(root->left);
    int rightH = height(root->right);
    return 1 + (leftH > rightH ? leftH : rightH);
}

// Free tree
void freeTree(BSTNode* root) {
    if (!root) return;
    freeTree(root->left);
    freeTree(root->right);
    free(root);
}

// Print tree visualization
void printTreeHelper(BSTNode* root, int space, int height) {
    if (!root) return;

    space += height;
    printTreeHelper(root->right, space, height);

    printf("\n");
    for (int i = height; i < space; i++)
        printf(" ");
    printf("%d\n", root->data);

    printTreeHelper(root->left, space, height);
}

void printTree(BSTNode* root) {
    printTreeHelper(root, 0, 5);
}

int main(void) {
    printf("=== Complete BST Implementation ===\n\n");

    BSTNode* root = NULL;

    // Insert values
    int values[] = {50, 30, 70, 20, 40, 60, 80, 10, 25, 35, 45};
    int n = sizeof(values) / sizeof(values[0]);

    printf("Inserting: ");
    for (int i = 0; i < n; i++) {
        printf("%d ", values[i]);
        root = insert(root, values[i]);
    }
    printf("\n\n");

    printf("Tree Structure:\n");
    printTree(root);

    printf("\nTraversals:\n");
    printf("  Inorder:   "); inorder(root); printf("\n");
    printf("  Preorder:  "); preorder(root); printf("\n");
    printf("  Postorder: "); postorder(root); printf("\n");

    printf("\nTree Properties:\n");
    printf("  Total nodes: %d\n", countNodes(root));
    printf("  Height: %d\n", height(root));
    printf("  Min value: %d\n", findMin(root)->data);
    printf("  Max value: %d\n", findMax(root)->data);

    // Search test
    printf("\nSearch Tests:\n");
    int targets[] = {40, 25, 100};
    for (int i = 0; i < 3; i++) {
        BSTNode* result = search(root, targets[i]);
        printf("  Search %d: %s\n", targets[i],
               result ? "Found" : "Not Found");
    }

    // Delete test
    printf("\nDelete 30 (node with two children):\n");
    root = deleteNode(root, 30);
    printf("  Inorder after deletion: ");
    inorder(root);
    printf("\n");

    freeTree(root);
    return 0;
}
```

---

## Summary (요약)

| Topic | Key Points |
|-------|------------|
| BST Concept | Left < Root < Right, efficient ordered data storage |
| BST Insert | Navigate tree based on comparison, O(log n) average |
| BST Search | Binary search in tree form, O(log n) average |
| BST Delete | Three cases: leaf, one child, two children |
| Time Complexity | O(log n) average, O(n) worst case (skewed) |

### BST Operation Summary (BST 연산 요약)

```
BST Operations Flow:

INSERT:              SEARCH:              DELETE:
   Compare            Compare              Find node
      |                  |                    |
   < node?           == node?            Has children?
    /   \             /   \               /    |    \
  Yes   No         Yes   No            None  One   Two
   |     |          |     |              |     |     |
  Left  Right    Found  Continue      Remove Replace Use
                          |                    |   Successor
                      < node?              with child
                       /   \
                     Left  Right
```

---

## Practice Exercises (연습 문제)

1. Implement a function to find the kth smallest element in a BST.
2. Write a function to convert a sorted array to a balanced BST.
3. Implement a function to find the lowest common ancestor (LCA) of two nodes in a BST.
4. Write a function to check if a given binary tree is a valid BST.
5. Implement BST using iterative methods for all operations (no recursion).
