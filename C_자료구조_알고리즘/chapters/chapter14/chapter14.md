# Chapter 14. Heap (힙)

> Understanding Heap data structure, Max/Min Heap operations, Heap Sort, and Priority Queue implementation.

---

## Heap Concept (힙 개념)

### Concept (개념)

A **Heap** (힙) is a specialized tree-based data structure that satisfies the **heap property**. It is a **Complete Binary Tree** where:

- **Max Heap** (최대 힙): Parent >= Children (Root has maximum value)
- **Min Heap** (최소 힙): Parent <= Children (Root has minimum value)

```
Max Heap Property:
Every parent node is GREATER than or equal to its children

        90        <- Root (Maximum)
       /  \
      70   80
     / \   /
    40 50 60

Parent >= Left Child AND Parent >= Right Child


Min Heap Property:
Every parent node is SMALLER than or equal to its children

        10        <- Root (Minimum)
       /  \
      20   30
     / \   /
    40 50 60

Parent <= Left Child AND Parent <= Right Child
```

### Complete Binary Tree (완전 이진 트리)

A heap must be a **Complete Binary Tree**:
- All levels are fully filled except possibly the last
- Last level is filled from left to right

```
Complete Binary Tree (Valid for Heap):
        1
       / \
      2   3
     / \ /
    4  5 6

NOT Complete (NOT valid):
        1
       / \
      2   3
     /     \
    4       6    <- Gap at position 5
```

### Array Representation (배열 표현)

Heaps are typically stored in arrays for efficiency:

```
For a node at index i (0-indexed):
  - Parent:      (i - 1) / 2
  - Left child:  2 * i + 1
  - Right child: 2 * i + 2

Tree:           Array:
     90         Index: [0]  [1]  [2]  [3]  [4]  [5]
    /  \        Value: [90] [70] [80] [40] [50] [60]
   70   80
  / \   /
 40 50 60

Node 70 (index 1):
  - Parent: (1-1)/2 = 0 (value 90)
  - Left child: 2*1+1 = 3 (value 40)
  - Right child: 2*1+2 = 4 (value 50)
```

```c
#include <stdio.h>

int main(void) {
    printf("=== Heap Array Index Formulas (힙 배열 인덱스 공식) ===\n\n");

    printf("For 0-indexed array:\n");
    printf("  Parent of i:      (i - 1) / 2\n");
    printf("  Left child of i:  2 * i + 1\n");
    printf("  Right child of i: 2 * i + 2\n\n");

    printf("Example:\n");
    printf("       90[0]\n");
    printf("      /    \\\n");
    printf("   70[1]   80[2]\n");
    printf("   / \\      /\n");
    printf("40[3] 50[4] 60[5]\n\n");

    // Demonstrate formulas
    int heap[] = {90, 70, 80, 40, 50, 60};
    int n = 6;

    printf("Array: ");
    for (int i = 0; i < n; i++) {
        printf("%d ", heap[i]);
    }
    printf("\n\n");

    for (int i = 0; i < n; i++) {
        printf("Node %d (value=%d):\n", i, heap[i]);
        int parent = (i - 1) / 2;
        int left = 2 * i + 1;
        int right = 2 * i + 2;

        if (i > 0)
            printf("  Parent: index %d (value=%d)\n", parent, heap[parent]);
        if (left < n)
            printf("  Left child: index %d (value=%d)\n", left, heap[left]);
        if (right < n)
            printf("  Right child: index %d (value=%d)\n", right, heap[right]);
        printf("\n");
    }

    return 0;
}
```

---

## Max Heap / Min Heap (최대 힙 / 최소 힙)

### Max Heap Structure

```c
#include <stdio.h>
#include <stdlib.h>

#define MAX_SIZE 100

typedef struct MaxHeap {
    int arr[MAX_SIZE];
    int size;
} MaxHeap;

// Initialize heap
void initHeap(MaxHeap* heap) {
    heap->size = 0;
}

// Check if heap is empty
int isEmpty(MaxHeap* heap) {
    return heap->size == 0;
}

// Check if heap is full
int isFull(MaxHeap* heap) {
    return heap->size == MAX_SIZE;
}

// Get parent index
int parent(int i) {
    return (i - 1) / 2;
}

// Get left child index
int leftChild(int i) {
    return 2 * i + 1;
}

// Get right child index
int rightChild(int i) {
    return 2 * i + 2;
}

// Swap two elements
void swap(int* a, int* b) {
    int temp = *a;
    *a = *b;
    *b = temp;
}

// Print heap as array
void printHeap(MaxHeap* heap) {
    printf("Heap Array: [");
    for (int i = 0; i < heap->size; i++) {
        printf("%d%s", heap->arr[i], i < heap->size - 1 ? ", " : "");
    }
    printf("]\n");
}

// Visualize heap as tree
void printHeapTree(MaxHeap* heap) {
    if (heap->size == 0) {
        printf("Empty heap\n");
        return;
    }

    int level = 0;
    int levelSize = 1;
    int printed = 0;

    printf("Heap Tree:\n");

    while (printed < heap->size) {
        // Print spacing for this level
        int spaces = (1 << (4 - level)) - 1;
        for (int i = 0; i < spaces; i++) printf(" ");

        // Print nodes at this level
        for (int i = 0; i < levelSize && printed < heap->size; i++) {
            printf("%2d", heap->arr[printed++]);
            // Spacing between nodes at same level
            for (int j = 0; j < 2 * spaces + 1; j++) printf(" ");
        }
        printf("\n");

        level++;
        levelSize *= 2;
    }
}

int main(void) {
    printf("=== Max Heap vs Min Heap (최대 힙 vs 최소 힙) ===\n\n");

    printf("Max Heap (최대 힙):\n");
    printf("  - Root contains MAXIMUM value\n");
    printf("  - Parent >= Children\n");
    printf("  - Used for: Max Priority Queue, Heap Sort (descending)\n\n");

    printf("       90       <- Maximum\n");
    printf("      /  \\\n");
    printf("     70   80\n");
    printf("    / \\   /\n");
    printf("   40 50 60\n\n");

    printf("Min Heap (최소 힙):\n");
    printf("  - Root contains MINIMUM value\n");
    printf("  - Parent <= Children\n");
    printf("  - Used for: Min Priority Queue, Dijkstra's Algorithm\n\n");

    printf("       10       <- Minimum\n");
    printf("      /  \\\n");
    printf("     20   30\n");
    printf("    / \\   /\n");
    printf("   40 50 60\n\n");

    return 0;
}
```

---

## Insert - Heapify Up (삽입 - 상향식 힙화)

### Concept (개념)

To insert into a Max Heap:
1. Add new element at the end (maintain complete tree)
2. **Heapify Up**: Compare with parent and swap if needed
3. Repeat until heap property is restored

```c
#include <stdio.h>
#include <stdlib.h>

#define MAX_SIZE 100

typedef struct MaxHeap {
    int arr[MAX_SIZE];
    int size;
} MaxHeap;

void initHeap(MaxHeap* heap) { heap->size = 0; }
int parent(int i) { return (i - 1) / 2; }
int leftChild(int i) { return 2 * i + 1; }
int rightChild(int i) { return 2 * i + 2; }

void swap(int* a, int* b) {
    int temp = *a; *a = *b; *b = temp;
}

void printHeapArray(MaxHeap* heap) {
    printf("[");
    for (int i = 0; i < heap->size; i++) {
        printf("%d%s", heap->arr[i], i < heap->size - 1 ? ", " : "");
    }
    printf("]");
}

// Heapify Up (상향식 힙화)
// Move newly inserted element up to maintain heap property
void heapifyUp(MaxHeap* heap, int index) {
    // While not at root and parent is smaller
    while (index > 0 && heap->arr[parent(index)] < heap->arr[index]) {
        printf("  Swap %d with parent %d\n",
               heap->arr[index], heap->arr[parent(index)]);

        swap(&heap->arr[index], &heap->arr[parent(index)]);
        index = parent(index);
    }
}

// Insert into Max Heap
void insert(MaxHeap* heap, int value) {
    if (heap->size >= MAX_SIZE) {
        printf("Heap overflow!\n");
        return;
    }

    printf("\nInserting %d:\n", value);

    // Step 1: Add at end
    heap->arr[heap->size] = value;
    int insertIndex = heap->size;
    heap->size++;

    printf("  Added at index %d: ", insertIndex);
    printHeapArray(heap);
    printf("\n");

    // Step 2: Heapify up
    heapifyUp(heap, insertIndex);

    printf("  After heapify: ");
    printHeapArray(heap);
    printf("\n");
}

int main(void) {
    printf("=== Heap Insert - Heapify Up (힙 삽입 - 상향식 힙화) ===\n");

    MaxHeap heap;
    initHeap(&heap);

    // Insert sequence: 40, 50, 60, 70, 80, 90
    int values[] = {40, 50, 60, 70, 80, 90};
    int n = sizeof(values) / sizeof(values[0]);

    for (int i = 0; i < n; i++) {
        insert(&heap, values[i]);
    }

    printf("\n=== Final Max Heap ===\n");
    printf("Array: ");
    printHeapArray(&heap);
    printf("\n\nTree structure:\n");
    printf("       90\n");
    printf("      /  \\\n");
    printf("     70   80\n");
    printf("    / \\   /\n");
    printf("   40 50 60\n");

    return 0;
}
```

**Insert Visualization:**
```
Inserting into Max Heap: 40, 50, 60, 70, 80, 90

Step 1: Insert 40
        [40]
        40

Step 2: Insert 50
        [40, 50]
        40          50 > 40, swap
       /     =>    /
      50          40

        [50, 40]

Step 3: Insert 60
        [50, 40, 60]
        50          60 > 50, swap
       / \    =>   / \
      40  60      40  50

        [60, 40, 50]

Step 4: Insert 70
        [60, 40, 50, 70]
           60             70 > 40, swap
          / \      =>    / \
         40  50         70  50
        /              /
       70             40

        Then: 70 > 60, swap
           70
          / \
         60  50
        /
       40

        [70, 60, 50, 40]

Step 5: Insert 80
        [70, 60, 50, 40, 80]
           70             80 > 60, swap
          / \      =>    / \
         60  50         80  50
        / \            / \
       40 80          40 60

        Then: 80 > 70, swap
           80
          / \
         70  50
        / \
       40 60

        [80, 70, 50, 40, 60]

Step 6: Insert 90
        Final: [90, 70, 80, 40, 60, 50]
           90
          /  \
         70   80
        / \   /
       40 60 50
```

---

## Delete - Heapify Down (삭제 - 하향식 힙화)

### Concept (개념)

To delete root (max/min element) from heap:
1. Replace root with last element
2. Remove last element
3. **Heapify Down**: Compare with children and swap with larger child
4. Repeat until heap property is restored

```c
#include <stdio.h>
#include <stdlib.h>

#define MAX_SIZE 100

typedef struct MaxHeap {
    int arr[MAX_SIZE];
    int size;
} MaxHeap;

void initHeap(MaxHeap* heap) { heap->size = 0; }
int parent(int i) { return (i - 1) / 2; }
int leftChild(int i) { return 2 * i + 1; }
int rightChild(int i) { return 2 * i + 2; }

void swap(int* a, int* b) {
    int temp = *a; *a = *b; *b = temp;
}

void printHeapArray(MaxHeap* heap) {
    printf("[");
    for (int i = 0; i < heap->size; i++) {
        printf("%d%s", heap->arr[i], i < heap->size - 1 ? ", " : "");
    }
    printf("]");
}

// Heapify Up for insert
void heapifyUp(MaxHeap* heap, int index) {
    while (index > 0 && heap->arr[parent(index)] < heap->arr[index]) {
        swap(&heap->arr[index], &heap->arr[parent(index)]);
        index = parent(index);
    }
}

// Heapify Down (하향식 힙화)
// Move element down to maintain heap property
void heapifyDown(MaxHeap* heap, int index) {
    int largest = index;
    int left = leftChild(index);
    int right = rightChild(index);

    // Find largest among node and its children
    if (left < heap->size && heap->arr[left] > heap->arr[largest]) {
        largest = left;
    }
    if (right < heap->size && heap->arr[right] > heap->arr[largest]) {
        largest = right;
    }

    // If largest is not the current node, swap and continue
    if (largest != index) {
        printf("  Swap %d with larger child %d\n",
               heap->arr[index], heap->arr[largest]);

        swap(&heap->arr[index], &heap->arr[largest]);
        heapifyDown(heap, largest);  // Recursively heapify
    }
}

// Insert
void insert(MaxHeap* heap, int value) {
    if (heap->size >= MAX_SIZE) return;
    heap->arr[heap->size] = value;
    heapifyUp(heap, heap->size);
    heap->size++;
}

// Extract Max (삭제)
int extractMax(MaxHeap* heap) {
    if (heap->size == 0) {
        printf("Heap underflow!\n");
        return -1;
    }

    int maxValue = heap->arr[0];
    printf("\nExtracting Max: %d\n", maxValue);

    // Step 1: Replace root with last element
    heap->arr[0] = heap->arr[heap->size - 1];
    heap->size--;

    printf("  After replacing root with last element: ");
    printHeapArray(heap);
    printf("\n");

    // Step 2: Heapify down
    if (heap->size > 0) {
        heapifyDown(heap, 0);
    }

    printf("  After heapify down: ");
    printHeapArray(heap);
    printf("\n");

    return maxValue;
}

// Peek Max (without removing)
int peekMax(MaxHeap* heap) {
    if (heap->size == 0) return -1;
    return heap->arr[0];
}

int main(void) {
    printf("=== Heap Delete - Heapify Down (힙 삭제 - 하향식 힙화) ===\n");

    MaxHeap heap;
    initHeap(&heap);

    // Build heap: 90, 70, 80, 40, 60, 50
    int values[] = {90, 70, 80, 40, 60, 50};
    for (int i = 0; i < 6; i++) {
        insert(&heap, values[i]);
    }

    printf("\nInitial Max Heap: ");
    printHeapArray(&heap);
    printf("\n\nTree:\n");
    printf("       90\n");
    printf("      /  \\\n");
    printf("     70   80\n");
    printf("    / \\   /\n");
    printf("   40 60 50\n");

    // Extract max three times
    printf("\n=== Extracting Maximum Elements ===\n");

    for (int i = 0; i < 3; i++) {
        int max = extractMax(&heap);
        printf("Extracted: %d\n", max);
    }

    printf("\nRemaining heap: ");
    printHeapArray(&heap);
    printf("\n");

    return 0;
}
```

**Delete Visualization:**
```
Delete Max from: [90, 70, 80, 40, 60, 50]

           90       <- Remove this
          /  \
         70   80
        / \   /
       40 60 50

Step 1: Save max (90)
Step 2: Replace root with last element (50)

           50       <- Out of place!
          /  \
         70   80
        / \
       40 60

        [50, 70, 80, 40, 60]

Step 3: Heapify Down
        Compare 50 with children (70, 80)
        80 > 70, so swap with 80

           80
          /  \
         70   50    <- Continue heapifying
        / \
       40 60

        [80, 70, 50, 40, 60]

        50 has no children larger than it
        DONE!

Result: [80, 70, 50, 40, 60]

           80
          /  \
         70   50
        / \
       40 60
```

---

## Heap Sort (힙 정렬)

### Concept (개념)

**Heap Sort** (힙 정렬) uses heap data structure to sort elements:
1. Build a Max Heap from the array
2. Repeatedly extract max and place at end

**Time Complexity**: O(n log n)
**Space Complexity**: O(1) - in-place sorting

```c
#include <stdio.h>

void swap(int* a, int* b) {
    int temp = *a; *a = *b; *b = temp;
}

void printArray(int arr[], int n) {
    printf("[");
    for (int i = 0; i < n; i++) {
        printf("%d%s", arr[i], i < n - 1 ? ", " : "");
    }
    printf("]\n");
}

// Heapify subtree rooted at index i
// n is size of heap
void heapify(int arr[], int n, int i) {
    int largest = i;
    int left = 2 * i + 1;
    int right = 2 * i + 2;

    if (left < n && arr[left] > arr[largest]) {
        largest = left;
    }

    if (right < n && arr[right] > arr[largest]) {
        largest = right;
    }

    if (largest != i) {
        swap(&arr[i], &arr[largest]);
        heapify(arr, n, largest);
    }
}

// Build Max Heap from array
void buildMaxHeap(int arr[], int n) {
    // Start from last non-leaf node
    // Last non-leaf node = (n/2) - 1
    for (int i = n / 2 - 1; i >= 0; i--) {
        heapify(arr, n, i);
    }
}

// Heap Sort
void heapSort(int arr[], int n) {
    printf("=== Heap Sort Process ===\n\n");

    printf("Original array: ");
    printArray(arr, n);

    // Step 1: Build Max Heap
    printf("\nStep 1: Build Max Heap\n");
    buildMaxHeap(arr, n);
    printf("Max Heap: ");
    printArray(arr, n);

    // Step 2: Extract elements one by one
    printf("\nStep 2: Extract max and rebuild\n");

    for (int i = n - 1; i > 0; i--) {
        // Move current root (max) to end
        printf("  Swap root %d with arr[%d]=%d: ", arr[0], i, arr[i]);
        swap(&arr[0], &arr[i]);
        printArray(arr, n);

        // Heapify reduced heap
        heapify(arr, i, 0);
        printf("  After heapify (heap size=%d): ", i);
        printArray(arr, n);
        printf("\n");
    }

    printf("Sorted array: ");
    printArray(arr, n);
}

int main(void) {
    printf("=== Heap Sort (힙 정렬) ===\n\n");

    int arr[] = {12, 11, 13, 5, 6, 7};
    int n = sizeof(arr) / sizeof(arr[0]);

    heapSort(arr, n);

    printf("\n=== Heap Sort Complexity ===\n");
    printf("Time Complexity:\n");
    printf("  - Build Heap: O(n)\n");
    printf("  - Extract n times: O(n log n)\n");
    printf("  - Total: O(n log n)\n\n");
    printf("Space Complexity: O(1) - In-place\n");
    printf("Stability: Not stable\n");

    return 0;
}
```

**Heap Sort Visualization:**
```
Original: [12, 11, 13, 5, 6, 7]

Step 1: Build Max Heap
        Start from last non-leaf (index 2)

        Index 2 (13): Already largest
        Index 1 (11): Children are 5, 6. 11 is largest
        Index 0 (12): Children are 11, 13. Swap with 13

        Result: [13, 11, 12, 5, 6, 7]

              13
             /  \
            11   12
           / \   /
          5   6 7

Step 2: Extract and Rebuild

        Extract 13:
        [7, 11, 12, 5, 6 | 13]
        Heapify: [12, 11, 7, 5, 6 | 13]

        Extract 12:
        [6, 11, 7, 5 | 12, 13]
        Heapify: [11, 6, 7, 5 | 12, 13]

        Extract 11:
        [5, 6, 7 | 11, 12, 13]
        Heapify: [7, 6, 5 | 11, 12, 13]

        Extract 7:
        [5, 6 | 7, 11, 12, 13]
        Heapify: [6, 5 | 7, 11, 12, 13]

        Extract 6:
        [5 | 6, 7, 11, 12, 13]

        Final Sorted: [5, 6, 7, 11, 12, 13]
```

---

## Priority Queue (우선순위 큐)

### Concept (개념)

A **Priority Queue** (우선순위 큐) is an abstract data type where each element has a priority. Elements with higher priority are served before elements with lower priority.

**Implementation using Heap:**
- Max Heap: Higher value = Higher priority
- Min Heap: Lower value = Higher priority

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define MAX_SIZE 100

// Priority Queue using Max Heap
typedef struct {
    int priority;
    char task[50];
} PQElement;

typedef struct {
    PQElement arr[MAX_SIZE];
    int size;
} PriorityQueue;

void initPQ(PriorityQueue* pq) {
    pq->size = 0;
}

int isEmpty(PriorityQueue* pq) {
    return pq->size == 0;
}

void swap(PQElement* a, PQElement* b) {
    PQElement temp = *a;
    *a = *b;
    *b = temp;
}

void heapifyUp(PriorityQueue* pq, int index) {
    int parent = (index - 1) / 2;
    while (index > 0 && pq->arr[parent].priority < pq->arr[index].priority) {
        swap(&pq->arr[index], &pq->arr[parent]);
        index = parent;
        parent = (index - 1) / 2;
    }
}

void heapifyDown(PriorityQueue* pq, int index) {
    int largest = index;
    int left = 2 * index + 1;
    int right = 2 * index + 2;

    if (left < pq->size &&
        pq->arr[left].priority > pq->arr[largest].priority) {
        largest = left;
    }
    if (right < pq->size &&
        pq->arr[right].priority > pq->arr[largest].priority) {
        largest = right;
    }

    if (largest != index) {
        swap(&pq->arr[index], &pq->arr[largest]);
        heapifyDown(pq, largest);
    }
}

// Enqueue with priority
void enqueue(PriorityQueue* pq, int priority, const char* task) {
    if (pq->size >= MAX_SIZE) {
        printf("Priority Queue is full!\n");
        return;
    }

    pq->arr[pq->size].priority = priority;
    strcpy(pq->arr[pq->size].task, task);
    heapifyUp(pq, pq->size);
    pq->size++;

    printf("Enqueued: [Priority %d] %s\n", priority, task);
}

// Dequeue highest priority element
PQElement dequeue(PriorityQueue* pq) {
    PQElement empty = {-1, ""};
    if (isEmpty(pq)) {
        printf("Priority Queue is empty!\n");
        return empty;
    }

    PQElement maxElement = pq->arr[0];
    pq->arr[0] = pq->arr[pq->size - 1];
    pq->size--;

    if (pq->size > 0) {
        heapifyDown(pq, 0);
    }

    return maxElement;
}

// Peek highest priority
PQElement peek(PriorityQueue* pq) {
    PQElement empty = {-1, ""};
    if (isEmpty(pq)) return empty;
    return pq->arr[0];
}

void printPQ(PriorityQueue* pq) {
    printf("\nPriority Queue Contents:\n");
    printf("+----------+------------------------+\n");
    printf("| Priority | Task                   |\n");
    printf("+----------+------------------------+\n");

    // Create a copy to not modify original
    PriorityQueue temp = *pq;
    while (!isEmpty(&temp)) {
        PQElement elem = dequeue(&temp);
        printf("| %8d | %-22s |\n", elem.priority, elem.task);
    }
    printf("+----------+------------------------+\n");
}

int main(void) {
    printf("=== Priority Queue using Heap (힙 기반 우선순위 큐) ===\n\n");

    PriorityQueue pq;
    initPQ(&pq);

    // Add tasks with priorities
    printf("Adding tasks:\n");
    enqueue(&pq, 3, "Do laundry");
    enqueue(&pq, 10, "Emergency meeting");
    enqueue(&pq, 1, "Watch TV");
    enqueue(&pq, 5, "Finish homework");
    enqueue(&pq, 8, "Doctor appointment");

    printPQ(&pq);

    printf("\nProcessing tasks by priority:\n");
    printf("==============================\n");

    while (!isEmpty(&pq)) {
        PQElement task = dequeue(&pq);
        printf("Processing: [Priority %d] %s\n", task.priority, task.task);
    }

    return 0;
}
```

**Priority Queue Example:**
```
Tasks with Priority:
+----------+------------------------+
| Priority | Task                   |
+----------+------------------------+
|       10 | Emergency meeting      |  <- Highest
|        8 | Doctor appointment     |
|        5 | Finish homework        |
|        3 | Do laundry             |
|        1 | Watch TV               |  <- Lowest
+----------+------------------------+

Heap Structure (Max Heap):
           (10, Emergency)
              /       \
    (8, Doctor)     (5, Homework)
        /    \
 (3, Laundry) (1, TV)

Processing Order:
1. Emergency meeting (Priority 10)
2. Doctor appointment (Priority 8)
3. Finish homework (Priority 5)
4. Do laundry (Priority 3)
5. Watch TV (Priority 1)
```

---

## Complete Heap Implementation

```c
#include <stdio.h>
#include <stdlib.h>

#define MAX_SIZE 100

typedef struct Heap {
    int arr[MAX_SIZE];
    int size;
    int isMaxHeap;  // 1 for max heap, 0 for min heap
} Heap;

// Initialize heap
Heap* createHeap(int isMaxHeap) {
    Heap* heap = (Heap*)malloc(sizeof(Heap));
    heap->size = 0;
    heap->isMaxHeap = isMaxHeap;
    return heap;
}

// Compare based on heap type
int compare(Heap* heap, int a, int b) {
    if (heap->isMaxHeap) {
        return a > b;  // Max heap: parent should be greater
    }
    return a < b;  // Min heap: parent should be smaller
}

void swap(int* a, int* b) {
    int temp = *a; *a = *b; *b = temp;
}

void heapifyUp(Heap* heap, int index) {
    int parent = (index - 1) / 2;
    while (index > 0 && compare(heap, heap->arr[index], heap->arr[parent])) {
        swap(&heap->arr[index], &heap->arr[parent]);
        index = parent;
        parent = (index - 1) / 2;
    }
}

void heapifyDown(Heap* heap, int index) {
    int extreme = index;
    int left = 2 * index + 1;
    int right = 2 * index + 2;

    if (left < heap->size && compare(heap, heap->arr[left], heap->arr[extreme])) {
        extreme = left;
    }
    if (right < heap->size && compare(heap, heap->arr[right], heap->arr[extreme])) {
        extreme = right;
    }

    if (extreme != index) {
        swap(&heap->arr[index], &heap->arr[extreme]);
        heapifyDown(heap, extreme);
    }
}

void insert(Heap* heap, int value) {
    if (heap->size >= MAX_SIZE) {
        printf("Heap overflow!\n");
        return;
    }
    heap->arr[heap->size] = value;
    heapifyUp(heap, heap->size);
    heap->size++;
}

int extractTop(Heap* heap) {
    if (heap->size == 0) {
        printf("Heap underflow!\n");
        return -1;
    }
    int top = heap->arr[0];
    heap->arr[0] = heap->arr[heap->size - 1];
    heap->size--;
    if (heap->size > 0) {
        heapifyDown(heap, 0);
    }
    return top;
}

int peek(Heap* heap) {
    if (heap->size == 0) return -1;
    return heap->arr[0];
}

void printHeap(Heap* heap) {
    printf("[");
    for (int i = 0; i < heap->size; i++) {
        printf("%d%s", heap->arr[i], i < heap->size - 1 ? ", " : "");
    }
    printf("]\n");
}

void freeHeap(Heap* heap) {
    free(heap);
}

int main(void) {
    printf("=== Complete Heap Implementation ===\n\n");

    // Max Heap Demo
    printf("Max Heap Demo:\n");
    Heap* maxHeap = createHeap(1);
    int values[] = {35, 33, 42, 10, 14, 19, 27, 44, 26, 31};

    for (int i = 0; i < 10; i++) {
        insert(maxHeap, values[i]);
    }
    printf("Max Heap: ");
    printHeap(maxHeap);
    printf("Max element: %d\n\n", peek(maxHeap));

    // Min Heap Demo
    printf("Min Heap Demo:\n");
    Heap* minHeap = createHeap(0);

    for (int i = 0; i < 10; i++) {
        insert(minHeap, values[i]);
    }
    printf("Min Heap: ");
    printHeap(minHeap);
    printf("Min element: %d\n\n", peek(minHeap));

    // Extract all from max heap
    printf("Extracting from Max Heap (descending order):\n");
    while (maxHeap->size > 0) {
        printf("%d ", extractTop(maxHeap));
    }
    printf("\n\n");

    // Extract all from min heap
    printf("Extracting from Min Heap (ascending order):\n");
    while (minHeap->size > 0) {
        printf("%d ", extractTop(minHeap));
    }
    printf("\n");

    freeHeap(maxHeap);
    freeHeap(minHeap);

    return 0;
}
```

---

## Summary (요약)

| Topic | Key Points |
|-------|------------|
| Heap Concept | Complete binary tree with heap property |
| Max Heap | Parent >= Children, root is maximum |
| Min Heap | Parent <= Children, root is minimum |
| Insert (Heapify Up) | Add at end, bubble up to correct position |
| Delete (Heapify Down) | Replace root with last, sink down |
| Heap Sort | Build heap + extract all, O(n log n) |
| Priority Queue | Efficient implementation using heap |

### Time Complexity Summary (시간 복잡도 요약)

| Operation | Time Complexity |
|-----------|-----------------|
| Build Heap | O(n) |
| Insert | O(log n) |
| Extract Max/Min | O(log n) |
| Peek Max/Min | O(1) |
| Heap Sort | O(n log n) |

### Heap vs Other Data Structures

```
| Operation        | Array (unsorted) | Array (sorted) | Heap      |
|------------------|------------------|----------------|-----------|
| Insert           | O(1)             | O(n)           | O(log n)  |
| Find Max/Min     | O(n)             | O(1)           | O(1)      |
| Extract Max/Min  | O(n)             | O(1) or O(n)   | O(log n)  |
| Build            | O(n)             | O(n log n)     | O(n)      |

Heap provides the best balance for priority queue operations!
```

---

## Practice Exercises (연습 문제)

1. Implement a Min Heap with decrease-key operation.
2. Find the kth largest element in an array using a heap.
3. Merge k sorted arrays using a min heap.
4. Implement a median finder using two heaps (max heap + min heap).
5. Build a heap from an array in O(n) time and prove why it works.
