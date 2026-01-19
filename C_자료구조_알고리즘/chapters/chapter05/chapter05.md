# Chapter 05. Arrays (배열)

> Understanding array memory structure, static vs dynamic arrays, and their operations.

---

## Memory Structure (메모리 구조)

### Concept (개념)

An **array** (배열) is a collection of elements of the same data type stored in contiguous (연속적인) memory locations. The elements are accessed using an index (인덱스), starting from 0.

**Key Properties:**
- Fixed size (for static arrays)
- Contiguous memory allocation
- Constant-time access O(1) using index
- Elements are of the same type

```c
#include <stdio.h>

int main(void) {
    // Array declaration and initialization
    int arr[5] = {10, 20, 30, 40, 50};

    printf("Array Memory Layout:\n");
    printf("====================\n\n");

    // Show memory addresses (메모리 주소 확인)
    printf("Index | Value | Address      | Offset\n");
    printf("------|-------|--------------|-------\n");

    for (int i = 0; i < 5; i++) {
        printf("  %d   |  %2d   | %p | +%lu bytes\n",
               i,
               arr[i],
               (void*)&arr[i],
               (unsigned long)(&arr[i] - &arr[0]) * sizeof(int));
    }

    // Base address (기본 주소)
    printf("\nBase address (arr): %p\n", (void*)arr);
    printf("Size of each element: %zu bytes\n", sizeof(int));
    printf("Total array size: %zu bytes\n", sizeof(arr));

    // Address calculation (주소 계산)
    // Address of arr[i] = Base address + (i * sizeof(element))
    printf("\nAddress calculation example:\n");
    printf("arr[3] address = %p\n", (void*)&arr[3]);
    printf("Calculated: %p + (3 * %zu) = %p\n",
           (void*)arr, sizeof(int), (void*)(arr + 3));

    return 0;
}
```

**Memory Visualization:**
```
Memory Address:  1000   1004   1008   1012   1016
                +------+------+------+------+------+
Array:          |  10  |  20  |  30  |  40  |  50  |
                +------+------+------+------+------+
Index:            [0]    [1]    [2]    [3]    [4]
```

### Example
> See `examples/` folder

---

## Static vs Dynamic Arrays (정적 배열 vs 동적 배열)

### Concept (개념)

**Static Array** (정적 배열):
- Size determined at compile time
- Allocated on the stack
- Cannot be resized
- Automatically deallocated when out of scope

**Dynamic Array** (동적 배열):
- Size determined at runtime
- Allocated on the heap using malloc/calloc
- Can be resized using realloc
- Must be manually freed

```c
#include <stdio.h>
#include <stdlib.h>

// Print array helper
void printArray(const char* name, int* arr, int size) {
    printf("%s: [", name);
    for (int i = 0; i < size; i++) {
        printf("%d", arr[i]);
        if (i < size - 1) printf(", ");
    }
    printf("]\n");
}

int main(void) {
    // ===== Static Array (정적 배열) =====
    printf("=== Static Array ===\n");

    // Size must be constant (compile-time)
    int staticArr[5] = {1, 2, 3, 4, 5};

    printArray("Static", staticArr, 5);
    printf("Size: %zu bytes\n", sizeof(staticArr));

    // Cannot resize static array!
    // staticArr = realloc(staticArr, 10 * sizeof(int)); // ERROR

    // ===== Dynamic Array (동적 배열) =====
    printf("\n=== Dynamic Array ===\n");

    // Size can be determined at runtime
    int n;
    printf("Enter array size: ");
    scanf("%d", &n);

    // Allocate dynamic array
    int* dynamicArr = (int*)malloc(n * sizeof(int));

    if (dynamicArr == NULL) {
        printf("Memory allocation failed!\n");
        return 1;
    }

    // Initialize
    for (int i = 0; i < n; i++) {
        dynamicArr[i] = (i + 1) * 10;
    }

    printArray("Dynamic", dynamicArr, n);

    // Resize dynamic array (동적 배열 크기 조정)
    printf("\nResizing array to %d elements...\n", n * 2);

    int newSize = n * 2;
    int* temp = (int*)realloc(dynamicArr, newSize * sizeof(int));

    if (temp == NULL) {
        printf("Reallocation failed!\n");
        free(dynamicArr);
        return 1;
    }

    dynamicArr = temp;

    // Initialize new elements
    for (int i = n; i < newSize; i++) {
        dynamicArr[i] = (i + 1) * 10;
    }

    printArray("Resized", dynamicArr, newSize);

    // Must free dynamic array (동적 배열 해제 필수)
    free(dynamicArr);
    printf("\nDynamic array freed.\n");

    // ===== Comparison =====
    printf("\n=== Comparison ===\n");
    printf("| Feature        | Static Array    | Dynamic Array     |\n");
    printf("|----------------|-----------------|-------------------|\n");
    printf("| Size           | Compile-time    | Runtime           |\n");
    printf("| Memory         | Stack           | Heap              |\n");
    printf("| Resizable      | No              | Yes (realloc)     |\n");
    printf("| Deallocation   | Automatic       | Manual (free)     |\n");

    return 0;
}
```

### Example
> See `examples/` folder

---

## Insert and Delete Operations (삽입과 삭제 연산)

### Concept (개념)

**Insert Operation** (삽입 연산):
- At end: O(1)
- At beginning or middle: O(n) - must shift elements

**Delete Operation** (삭제 연산):
- At end: O(1)
- At beginning or middle: O(n) - must shift elements

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    int* data;
    int size;
    int capacity;
} ArrayList;

// Create ArrayList
ArrayList* createArrayList(int initialCapacity) {
    ArrayList* list = (ArrayList*)malloc(sizeof(ArrayList));
    list->data = (int*)malloc(initialCapacity * sizeof(int));
    list->size = 0;
    list->capacity = initialCapacity;
    return list;
}

// Ensure capacity
void ensureCapacity(ArrayList* list) {
    if (list->size >= list->capacity) {
        list->capacity *= 2;
        list->data = (int*)realloc(list->data, list->capacity * sizeof(int));
        printf("  [Capacity doubled to %d]\n", list->capacity);
    }
}

// Insert at end - O(1) amortized
void insertEnd(ArrayList* list, int value) {
    ensureCapacity(list);
    list->data[list->size++] = value;
}

// Insert at beginning - O(n)
void insertBeginning(ArrayList* list, int value) {
    ensureCapacity(list);

    // Shift all elements right (오른쪽으로 이동)
    for (int i = list->size; i > 0; i--) {
        list->data[i] = list->data[i - 1];
    }

    list->data[0] = value;
    list->size++;
}

// Insert at index - O(n)
void insertAt(ArrayList* list, int index, int value) {
    if (index < 0 || index > list->size) {
        printf("Invalid index!\n");
        return;
    }

    ensureCapacity(list);

    // Shift elements from index to right
    for (int i = list->size; i > index; i--) {
        list->data[i] = list->data[i - 1];
    }

    list->data[index] = value;
    list->size++;
}

// Delete at end - O(1)
int deleteEnd(ArrayList* list) {
    if (list->size == 0) {
        printf("List is empty!\n");
        return -1;
    }
    return list->data[--list->size];
}

// Delete at beginning - O(n)
int deleteBeginning(ArrayList* list) {
    if (list->size == 0) {
        printf("List is empty!\n");
        return -1;
    }

    int value = list->data[0];

    // Shift all elements left (왼쪽으로 이동)
    for (int i = 0; i < list->size - 1; i++) {
        list->data[i] = list->data[i + 1];
    }

    list->size--;
    return value;
}

// Delete at index - O(n)
int deleteAt(ArrayList* list, int index) {
    if (index < 0 || index >= list->size) {
        printf("Invalid index!\n");
        return -1;
    }

    int value = list->data[index];

    // Shift elements from index+1 to left
    for (int i = index; i < list->size - 1; i++) {
        list->data[i] = list->data[i + 1];
    }

    list->size--;
    return value;
}

// Print ArrayList
void printArrayList(ArrayList* list) {
    printf("ArrayList [size=%d, capacity=%d]: [", list->size, list->capacity);
    for (int i = 0; i < list->size; i++) {
        printf("%d", list->data[i]);
        if (i < list->size - 1) printf(", ");
    }
    printf("]\n");
}

// Free ArrayList
void freeArrayList(ArrayList* list) {
    free(list->data);
    free(list);
}

int main(void) {
    ArrayList* list = createArrayList(4);

    printf("=== Insert Operations ===\n\n");

    // Insert at end - O(1)
    printf("Insert at end: 10, 20, 30\n");
    insertEnd(list, 10);
    insertEnd(list, 20);
    insertEnd(list, 30);
    printArrayList(list);

    // Insert at beginning - O(n)
    printf("\nInsert 5 at beginning:\n");
    insertBeginning(list, 5);
    printArrayList(list);

    // Insert at index - O(n)
    printf("\nInsert 15 at index 2:\n");
    insertAt(list, 2, 15);
    printArrayList(list);

    printf("\n=== Delete Operations ===\n\n");

    // Delete at end - O(1)
    printf("Delete from end: %d\n", deleteEnd(list));
    printArrayList(list);

    // Delete at beginning - O(n)
    printf("\nDelete from beginning: %d\n", deleteBeginning(list));
    printArrayList(list);

    // Delete at index - O(n)
    printf("\nDelete at index 1: %d\n", deleteAt(list, 1));
    printArrayList(list);

    printf("\n=== Time Complexity Summary ===\n");
    printf("| Operation          | Time Complexity |\n");
    printf("|--------------------|----------------|\n");
    printf("| Access by index    | O(1)           |\n");
    printf("| Insert at end      | O(1) amortized |\n");
    printf("| Insert at beginning| O(n)           |\n");
    printf("| Insert at middle   | O(n)           |\n");
    printf("| Delete at end      | O(1)           |\n");
    printf("| Delete at beginning| O(n)           |\n");
    printf("| Delete at middle   | O(n)           |\n");
    printf("| Search (unsorted)  | O(n)           |\n");

    freeArrayList(list);

    return 0;
}
```

### Example
> See `examples/` folder

---

## 2D Arrays (2차원 배열)

### Concept (개념)

A **2D array** (2차원 배열) is an array of arrays, often used to represent matrices (행렬), tables, or grids.

**Memory Layout:**
- Row-major order (행 우선 순서): Elements of each row are stored contiguously
- Column-major order (열 우선 순서): Elements of each column are stored contiguously
- C uses row-major order

```c
#include <stdio.h>
#include <stdlib.h>

// Print 2D array
void print2DArray(int rows, int cols, int arr[rows][cols]) {
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            printf("%3d ", arr[i][j]);
        }
        printf("\n");
    }
}

int main(void) {
    // ===== Static 2D Array =====
    printf("=== Static 2D Array (정적 2차원 배열) ===\n\n");

    // Declaration and initialization
    int matrix[3][4] = {
        {1, 2, 3, 4},
        {5, 6, 7, 8},
        {9, 10, 11, 12}
    };

    printf("3x4 Matrix:\n");
    print2DArray(3, 4, matrix);

    // Memory layout (메모리 레이아웃)
    printf("\nMemory layout (Row-major order):\n");
    printf("Address     | Index   | Value\n");
    printf("------------|---------|------\n");

    for (int i = 0; i < 3; i++) {
        for (int j = 0; j < 4; j++) {
            printf("%p | [%d][%d]  | %d\n",
                   (void*)&matrix[i][j], i, j, matrix[i][j]);
        }
    }

    // Accessing elements (요소 접근)
    printf("\nAccessing matrix[1][2]: %d\n", matrix[1][2]);

    // ===== Dynamic 2D Array =====
    printf("\n=== Dynamic 2D Array (동적 2차원 배열) ===\n\n");

    int rows = 3, cols = 4;

    // Method 1: Array of pointers (포인터 배열)
    printf("Method 1: Array of pointers\n");
    int** dynamic2D = (int**)malloc(rows * sizeof(int*));

    for (int i = 0; i < rows; i++) {
        dynamic2D[i] = (int*)malloc(cols * sizeof(int));
        for (int j = 0; j < cols; j++) {
            dynamic2D[i][j] = i * cols + j + 1;
        }
    }

    printf("Dynamic 3x4 Matrix:\n");
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            printf("%3d ", dynamic2D[i][j]);
        }
        printf("\n");
    }

    // Free Method 1
    for (int i = 0; i < rows; i++) {
        free(dynamic2D[i]);
    }
    free(dynamic2D);

    // Method 2: Single contiguous block (연속 메모리 블록)
    printf("\nMethod 2: Single contiguous block\n");
    int* contiguous = (int*)malloc(rows * cols * sizeof(int));

    // Initialize
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            // Calculate index: i * cols + j
            contiguous[i * cols + j] = (i + 1) * 10 + (j + 1);
        }
    }

    printf("Contiguous 3x4 Matrix:\n");
    for (int i = 0; i < rows; i++) {
        for (int j = 0; j < cols; j++) {
            printf("%3d ", contiguous[i * cols + j]);
        }
        printf("\n");
    }

    free(contiguous);

    // ===== Matrix Operations =====
    printf("\n=== Matrix Operations (행렬 연산) ===\n\n");

    int A[2][3] = {{1, 2, 3}, {4, 5, 6}};
    int B[3][2] = {{7, 8}, {9, 10}, {11, 12}};
    int C[2][2] = {0};  // Result matrix

    printf("Matrix A (2x3):\n");
    print2DArray(2, 3, A);

    printf("\nMatrix B (3x2):\n");
    print2DArray(3, 2, B);

    // Matrix multiplication (행렬 곱셈): C = A * B
    // C[i][j] = sum of A[i][k] * B[k][j] for all k
    for (int i = 0; i < 2; i++) {
        for (int j = 0; j < 2; j++) {
            C[i][j] = 0;
            for (int k = 0; k < 3; k++) {
                C[i][j] += A[i][k] * B[k][j];
            }
        }
    }

    printf("\nMatrix C = A * B (2x2):\n");
    print2DArray(2, 2, C);

    return 0;
}
```

### 2D Array Memory Visualization

```
Static 2D Array (Row-major order):
matrix[3][4] stored as:
+----+----+----+----+----+----+----+----+----+----+----+----+
| 1  | 2  | 3  | 4  | 5  | 6  | 7  | 8  | 9  | 10 | 11 | 12 |
+----+----+----+----+----+----+----+----+----+----+----+----+
|    Row 0      |    Row 1      |    Row 2      |
```

```
Dynamic 2D Array (Array of pointers):
dynamic2D
    +-------+     +----+----+----+----+
    |   *---+---> | 1  | 2  | 3  | 4  |  Row 0
    +-------+     +----+----+----+----+
    |   *---+---> | 5  | 6  | 7  | 8  |  Row 1
    +-------+     +----+----+----+----+
    |   *---+---> | 9  | 10 | 11 | 12 |  Row 2
    +-------+     +----+----+----+----+
```

### Example
> See `examples/` folder

---

## Array Utility Functions (배열 유틸리티 함수)

### Concept (개념)

Common array operations implemented as reusable functions.

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

// Find maximum element
int findMax(int arr[], int n) {
    int max = arr[0];
    for (int i = 1; i < n; i++) {
        if (arr[i] > max) {
            max = arr[i];
        }
    }
    return max;
}

// Find minimum element
int findMin(int arr[], int n) {
    int min = arr[0];
    for (int i = 1; i < n; i++) {
        if (arr[i] < min) {
            min = arr[i];
        }
    }
    return min;
}

// Calculate sum
int sum(int arr[], int n) {
    int total = 0;
    for (int i = 0; i < n; i++) {
        total += arr[i];
    }
    return total;
}

// Calculate average
double average(int arr[], int n) {
    return (double)sum(arr, n) / n;
}

// Reverse array in place
void reverse(int arr[], int n) {
    for (int i = 0; i < n / 2; i++) {
        int temp = arr[i];
        arr[i] = arr[n - 1 - i];
        arr[n - 1 - i] = temp;
    }
}

// Copy array
int* copyArray(int arr[], int n) {
    int* copy = (int*)malloc(n * sizeof(int));
    if (copy != NULL) {
        memcpy(copy, arr, n * sizeof(int));
    }
    return copy;
}

// Rotate array left by k positions
void rotateLeft(int arr[], int n, int k) {
    k = k % n;  // Handle k > n

    int* temp = (int*)malloc(k * sizeof(int));

    // Save first k elements
    memcpy(temp, arr, k * sizeof(int));

    // Shift remaining elements left
    memmove(arr, arr + k, (n - k) * sizeof(int));

    // Put saved elements at end
    memcpy(arr + n - k, temp, k * sizeof(int));

    free(temp);
}

// Find element (linear search)
int find(int arr[], int n, int target) {
    for (int i = 0; i < n; i++) {
        if (arr[i] == target) {
            return i;
        }
    }
    return -1;
}

// Count occurrences
int count(int arr[], int n, int target) {
    int cnt = 0;
    for (int i = 0; i < n; i++) {
        if (arr[i] == target) {
            cnt++;
        }
    }
    return cnt;
}

// Print array
void printArray(int arr[], int n) {
    printf("[");
    for (int i = 0; i < n; i++) {
        printf("%d", arr[i]);
        if (i < n - 1) printf(", ");
    }
    printf("]\n");
}

int main(void) {
    int arr[] = {5, 2, 8, 1, 9, 3, 7, 4, 6, 2};
    int n = sizeof(arr) / sizeof(arr[0]);

    printf("Original array: ");
    printArray(arr, n);

    printf("\n=== Statistics ===\n");
    printf("Max: %d\n", findMax(arr, n));
    printf("Min: %d\n", findMin(arr, n));
    printf("Sum: %d\n", sum(arr, n));
    printf("Average: %.2f\n", average(arr, n));

    printf("\n=== Search ===\n");
    printf("Index of 7: %d\n", find(arr, n, 7));
    printf("Count of 2: %d\n", count(arr, n, 2));

    printf("\n=== Transformations ===\n");

    int* copy = copyArray(arr, n);
    reverse(copy, n);
    printf("Reversed: ");
    printArray(copy, n);
    free(copy);

    copy = copyArray(arr, n);
    rotateLeft(copy, n, 3);
    printf("Rotated left by 3: ");
    printArray(copy, n);
    free(copy);

    return 0;
}
```

### Example
> See `examples/` folder

---

## Summary (요약)

| Topic | Key Points |
|-------|------------|
| Memory Structure | Contiguous memory, O(1) index access |
| Static Array | Fixed size, stack allocated, automatic deallocation |
| Dynamic Array | Runtime size, heap allocated, must free manually |
| Insert/Delete | O(1) at end, O(n) at beginning/middle |
| 2D Array | Array of arrays, row-major order in C |

### Time Complexity Summary (시간 복잡도 요약)

| Operation | Array |
|-----------|-------|
| Access | O(1) |
| Search (unsorted) | O(n) |
| Search (sorted) | O(log n) |
| Insert at end | O(1) |
| Insert at position | O(n) |
| Delete at end | O(1) |
| Delete at position | O(n) |

---

## Practice Exercises (연습 문제)

1. Implement a function to merge two sorted arrays into one sorted array.
2. Write a function to remove duplicates from a sorted array in place.
3. Implement matrix transpose operation.
4. Create a function to find the second largest element in an array.
5. Implement a circular buffer using a static array.
