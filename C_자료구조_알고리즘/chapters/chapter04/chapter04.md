# Chapter 04. Algorithm Analysis (알고리즘 분석)

> Understanding how to measure and compare algorithm efficiency using time and space complexity.

---

## Algorithm (알고리즘)

### Concept (개념)

An **algorithm** (알고리즘) is a step-by-step procedure for solving a problem or performing a computation. A good algorithm should be:

1. **Correct** (정확성): Produces the expected output for all valid inputs
2. **Efficient** (효율성): Uses minimal time and memory
3. **Clear** (명확성): Easy to understand and implement
4. **Finite** (유한성): Terminates after a finite number of steps

```c
#include <stdio.h>

// Algorithm 1: Find the maximum value in an array (배열의 최댓값 찾기)
int findMax(int arr[], int n) {
    int max = arr[0];       // Step 1: Initialize max
    for (int i = 1; i < n; i++) {  // Step 2: Iterate through array
        if (arr[i] > max) {  // Step 3: Compare each element
            max = arr[i];    // Step 4: Update max if larger found
        }
    }
    return max;              // Step 5: Return result
}

// Algorithm 2: Calculate sum of integers from 1 to n
// Method A: Loop (반복문)
int sumLoop(int n) {
    int sum = 0;
    for (int i = 1; i <= n; i++) {
        sum += i;
    }
    return sum;
}

// Method B: Mathematical formula (수학 공식)
int sumFormula(int n) {
    return n * (n + 1) / 2;
}

int main(void) {
    int arr[] = {3, 7, 2, 9, 1, 5};
    int n = sizeof(arr) / sizeof(arr[0]);

    printf("Array: ");
    for (int i = 0; i < n; i++) {
        printf("%d ", arr[i]);
    }
    printf("\nMaximum value: %d\n", findMax(arr, n));

    int num = 100;
    printf("\nSum from 1 to %d:\n", num);
    printf("  Using loop: %d\n", sumLoop(num));
    printf("  Using formula: %d\n", sumFormula(num));

    return 0;
}
```

### Example
> See `examples/` folder

---

## Time Complexity (시간 복잡도)

### Concept (개념)

**Time complexity** (시간 복잡도) measures how the running time of an algorithm grows as the input size increases. We count the number of basic operations (comparisons, assignments, arithmetic) as a function of input size n.

**Types of Analysis:**
- **Best Case** (최선의 경우): Minimum operations needed
- **Average Case** (평균적인 경우): Expected operations for typical input
- **Worst Case** (최악의 경우): Maximum operations needed (most commonly used)

```c
#include <stdio.h>

// Example 1: Constant time - O(1)
// Time does not depend on input size
int getFirst(int arr[], int n) {
    return arr[0];  // Always 1 operation
}

// Example 2: Linear time - O(n)
// Time grows linearly with input size
int linearSearch(int arr[], int n, int target) {
    for (int i = 0; i < n; i++) {  // n iterations at worst
        if (arr[i] == target) {
            return i;
        }
    }
    return -1;
}

// Example 3: Quadratic time - O(n^2)
// Time grows with square of input size
void bubbleSort(int arr[], int n) {
    for (int i = 0; i < n - 1; i++) {        // n-1 iterations
        for (int j = 0; j < n - i - 1; j++) { // n-i-1 iterations
            if (arr[j] > arr[j + 1]) {
                // Swap
                int temp = arr[j];
                arr[j] = arr[j + 1];
                arr[j + 1] = temp;
            }
        }
    }
}

// Example 4: Logarithmic time - O(log n)
// Time grows logarithmically (very efficient)
int binarySearch(int arr[], int n, int target) {
    int left = 0, right = n - 1;

    while (left <= right) {
        int mid = left + (right - left) / 2;

        if (arr[mid] == target) {
            return mid;
        } else if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid - 1;
        }
    }
    return -1;
}

// Example 5: Linearithmic time - O(n log n)
// Common in efficient sorting algorithms (merge sort, quick sort)

int main(void) {
    int arr[] = {64, 34, 25, 12, 22, 11, 90};
    int n = sizeof(arr) / sizeof(arr[0]);

    // O(1) - Constant
    printf("First element (O(1)): %d\n", getFirst(arr, n));

    // O(n) - Linear
    int target = 25;
    int index = linearSearch(arr, n, target);
    printf("Linear search for %d (O(n)): index %d\n", target, index);

    // O(n^2) - Quadratic
    printf("\nBubble Sort (O(n^2)):\n");
    printf("Before: ");
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);

    bubbleSort(arr, n);

    printf("\nAfter:  ");
    for (int i = 0; i < n; i++) printf("%d ", arr[i]);
    printf("\n");

    // O(log n) - Logarithmic (requires sorted array)
    target = 34;
    index = binarySearch(arr, n, target);
    printf("\nBinary search for %d (O(log n)): index %d\n", target, index);

    return 0;
}
```

### Example
> See `examples/` folder

---

## Space Complexity (공간 복잡도)

### Concept (개념)

**Space complexity** (공간 복잡도) measures the amount of memory an algorithm uses as a function of input size. This includes:

1. **Fixed space**: Memory for constants, simple variables, program code
2. **Variable space**: Memory that depends on input size (arrays, dynamic allocation, recursion stack)

```c
#include <stdio.h>
#include <stdlib.h>

// O(1) space - Constant space
// Only uses a fixed number of variables regardless of n
int sumConstantSpace(int arr[], int n) {
    int sum = 0;  // Only one extra variable
    for (int i = 0; i < n; i++) {
        sum += arr[i];
    }
    return sum;
}

// O(n) space - Linear space
// Creates a new array proportional to input size
int* copyArray(int arr[], int n) {
    int* copy = (int*)malloc(n * sizeof(int));  // O(n) space
    for (int i = 0; i < n; i++) {
        copy[i] = arr[i];
    }
    return copy;
}

// O(n) space due to recursion (재귀로 인한 O(n) 공간)
// Each recursive call adds a frame to the call stack
int factorialRecursive(int n) {
    if (n <= 1) return 1;
    return n * factorialRecursive(n - 1);  // O(n) stack frames
}

// O(1) space - Iterative version
int factorialIterative(int n) {
    int result = 1;
    for (int i = 2; i <= n; i++) {
        result *= i;
    }
    return result;  // Only uses fixed variables
}

// O(n^2) space - Quadratic space
int** create2DArray(int n) {
    int** matrix = (int**)malloc(n * sizeof(int*));
    for (int i = 0; i < n; i++) {
        matrix[i] = (int*)malloc(n * sizeof(int));
    }
    return matrix;  // n * n = n^2 space
}

void free2DArray(int** matrix, int n) {
    for (int i = 0; i < n; i++) {
        free(matrix[i]);
    }
    free(matrix);
}

int main(void) {
    int arr[] = {1, 2, 3, 4, 5};
    int n = 5;

    // O(1) space example
    printf("Sum (O(1) space): %d\n", sumConstantSpace(arr, n));

    // O(n) space example
    int* copied = copyArray(arr, n);
    printf("Copied array (O(n) space): ");
    for (int i = 0; i < n; i++) {
        printf("%d ", copied[i]);
    }
    printf("\n");
    free(copied);

    // Factorial comparison
    printf("\nFactorial of 5:\n");
    printf("  Recursive (O(n) space): %d\n", factorialRecursive(5));
    printf("  Iterative (O(1) space): %d\n", factorialIterative(5));

    // O(n^2) space example
    int size = 3;
    int** matrix = create2DArray(size);
    printf("\n2D Array (O(n^2) space) created: %dx%d\n", size, size);
    free2DArray(matrix, size);

    return 0;
}
```

### Example
> See `examples/` folder

---

## Big-O Notation (빅오 표기법)

### Concept (개념)

**Big-O notation** (빅오 표기법) is a mathematical notation that describes the upper bound of an algorithm's growth rate. It focuses on the dominant term and ignores constants and lower-order terms.

**Common Time Complexities (Ordered from fastest to slowest):**

| Big-O | Name | Example |
|-------|------|---------|
| O(1) | Constant | Array access, hash table lookup |
| O(log n) | Logarithmic | Binary search |
| O(n) | Linear | Linear search, single loop |
| O(n log n) | Linearithmic | Merge sort, quick sort (average) |
| O(n^2) | Quadratic | Bubble sort, nested loops |
| O(n^3) | Cubic | Matrix multiplication |
| O(2^n) | Exponential | Recursive Fibonacci |
| O(n!) | Factorial | Permutations |

**Rules for Big-O:**
1. Drop constants: O(2n) = O(n)
2. Drop lower-order terms: O(n^2 + n) = O(n^2)
3. Different inputs use different variables: O(a + b) not O(n)

```c
#include <stdio.h>

// O(1) - Constant: Does not depend on n
int constantTime(int n) {
    int x = n + 1;      // 1 operation
    int y = n * 2;      // 1 operation
    return x + y;       // 1 operation
    // Total: 3 operations = O(1)
}

// O(n) - Linear: Grows linearly with n
int linearTime(int n) {
    int sum = 0;
    for (int i = 0; i < n; i++) {  // n iterations
        sum += i;                   // 1 operation per iteration
    }
    return sum;
    // Total: n operations = O(n)
}

// O(n^2) - Quadratic: Grows with square of n
int quadraticTime(int n) {
    int count = 0;
    for (int i = 0; i < n; i++) {       // n iterations
        for (int j = 0; j < n; j++) {   // n iterations each
            count++;                     // 1 operation
        }
    }
    return count;
    // Total: n * n = n^2 operations = O(n^2)
}

// O(log n) - Logarithmic: Halves problem size each step
int logarithmicTime(int n) {
    int count = 0;
    while (n > 1) {
        n = n / 2;  // Halve n each iteration
        count++;
    }
    return count;
    // For n=16: 16->8->4->2->1 = 4 steps = log2(16)
    // Total: log2(n) operations = O(log n)
}

// O(n log n) - Linearithmic
int linearithmicTime(int n) {
    int count = 0;
    for (int i = 0; i < n; i++) {       // n iterations
        int temp = n;
        while (temp > 1) {               // log n iterations
            temp = temp / 2;
            count++;
        }
    }
    return count;
    // Total: n * log(n) operations = O(n log n)
}

// O(2^n) - Exponential (avoid for large n!)
int fibonacci(int n) {
    if (n <= 1) return n;
    return fibonacci(n - 1) + fibonacci(n - 2);
    // Creates 2 recursive calls at each level
    // Total: approximately 2^n operations = O(2^n)
}

int main(void) {
    int n = 16;

    printf("Input size n = %d\n\n", n);

    printf("O(1) - Constant Time\n");
    printf("  Result: %d\n", constantTime(n));
    printf("  Operations: ~3 (constant)\n\n");

    printf("O(n) - Linear Time\n");
    printf("  Sum 0 to %d: %d\n", n-1, linearTime(n));
    printf("  Operations: ~%d\n\n", n);

    printf("O(n^2) - Quadratic Time\n");
    printf("  Count: %d\n", quadraticTime(n));
    printf("  Operations: ~%d\n\n", n * n);

    printf("O(log n) - Logarithmic Time\n");
    printf("  Steps to halve %d to 1: %d\n", n, logarithmicTime(n));
    printf("  Operations: ~%d (log2(%d))\n\n", logarithmicTime(n), n);

    printf("O(n log n) - Linearithmic Time\n");
    printf("  Count: %d\n", linearithmicTime(n));
    printf("  Operations: ~%d\n\n", n * logarithmicTime(n));

    printf("O(2^n) - Exponential Time (using small n=10)\n");
    printf("  Fibonacci(10): %d\n", fibonacci(10));
    printf("  Operations: ~%d (2^10)\n", 1024);

    return 0;
}
```

### Growth Rate Comparison (성장률 비교)

```
n        | O(1)  | O(log n) | O(n)   | O(n log n) | O(n^2)    | O(2^n)
---------|-------|----------|--------|------------|-----------|------------
10       | 1     | 3        | 10     | 33         | 100       | 1,024
100      | 1     | 7        | 100    | 664        | 10,000    | 10^30
1,000    | 1     | 10       | 1,000  | 9,966      | 1,000,000 | 10^301
10,000   | 1     | 13       | 10,000 | 132,877    | 10^8      | ...
```

### Example
> See `examples/` folder

---

## Analyzing Algorithm Complexity (알고리즘 복잡도 분석)

### Concept (개념)

To analyze an algorithm's complexity:

1. Identify the basic operations
2. Count how many times each operation executes
3. Express the count as a function of input size
4. Simplify using Big-O rules

```c
#include <stdio.h>

// Analysis Example 1: Sequential statements
void example1(int n) {
    int a = 1;                    // O(1)
    int b = 2;                    // O(1)

    for (int i = 0; i < n; i++) { // O(n)
        printf("%d ", i);
    }

    for (int i = 0; i < n; i++) { // O(n)
        printf("%d ", i * 2);
    }

    // Total: O(1) + O(1) + O(n) + O(n) = O(2n + 2) = O(n)
    printf("\nExample 1 complexity: O(n)\n");
}

// Analysis Example 2: Nested loops with different bounds
void example2(int n) {
    // Outer loop: n times
    for (int i = 0; i < n; i++) {
        // Inner loop: i times (varies: 0, 1, 2, ..., n-1)
        for (int j = 0; j < i; j++) {
            printf(".");
        }
        printf("\n");
    }

    // Total iterations: 0 + 1 + 2 + ... + (n-1) = n(n-1)/2 = O(n^2)
    printf("Example 2 complexity: O(n^2)\n");
}

// Analysis Example 3: Early termination
int example3(int arr[], int n, int target) {
    // Best case: O(1) - found at first position
    // Average case: O(n/2) = O(n)
    // Worst case: O(n) - found at last position or not found

    for (int i = 0; i < n; i++) {
        if (arr[i] == target) {
            return i;  // Early termination possible
        }
    }
    return -1;

    // We use worst case: O(n)
}

// Analysis Example 4: Multiple sequences
void example4(int n, int m) {
    // First loop: O(n)
    for (int i = 0; i < n; i++) {
        printf("N");
    }

    // Second loop: O(m)
    for (int j = 0; j < m; j++) {
        printf("M");
    }

    // Total: O(n + m) - cannot simplify because n and m are independent
    printf("\nExample 4 complexity: O(n + m)\n");
}

// Analysis Example 5: Dependent loops
void example5(int n) {
    // Outer: O(n)
    for (int i = 0; i < n; i++) {
        // Inner: O(n) for each outer iteration
        for (int j = 0; j < n; j++) {
            // Innermost: O(n) for each inner iteration
            for (int k = 0; k < n; k++) {
                printf(".");
            }
        }
    }

    // Total: O(n * n * n) = O(n^3)
    printf("\nExample 5 complexity: O(n^3)\n");
}

int main(void) {
    int n = 5;

    printf("=== Example 1: Sequential O(n) ===\n");
    example1(n);

    printf("\n=== Example 2: Triangular O(n^2) ===\n");
    example2(n);

    printf("\n=== Example 3: Linear Search O(n) ===\n");
    int arr[] = {1, 2, 3, 4, 5};
    printf("Search result: %d\n", example3(arr, 5, 3));

    printf("\n=== Example 4: Two variables O(n + m) ===\n");
    example4(3, 4);

    printf("\n=== Example 5: Cubic O(n^3) ===\n");
    example5(2);

    return 0;
}
```

### Example
> See `examples/` folder

---

## Amortized Analysis (분할 상환 분석)

### Concept (개념)

**Amortized analysis** (분할 상환 분석) considers the average time per operation over a sequence of operations. Some operations may be expensive, but if they occur rarely, the average cost per operation is low.

```c
#include <stdio.h>
#include <stdlib.h>

typedef struct {
    int* data;
    int size;
    int capacity;
} DynamicArray;

DynamicArray* createArray(void) {
    DynamicArray* arr = (DynamicArray*)malloc(sizeof(DynamicArray));
    arr->capacity = 1;
    arr->size = 0;
    arr->data = (int*)malloc(arr->capacity * sizeof(int));
    return arr;
}

void append(DynamicArray* arr, int value) {
    // If full, double the capacity - O(n) operation
    if (arr->size == arr->capacity) {
        arr->capacity *= 2;
        arr->data = (int*)realloc(arr->data, arr->capacity * sizeof(int));
        printf("  [Resized to capacity %d]\n", arr->capacity);
    }

    // Add element - O(1) operation
    arr->data[arr->size++] = value;
}

void freeArray(DynamicArray* arr) {
    free(arr->data);
    free(arr);
}

int main(void) {
    printf("Dynamic Array - Amortized Analysis\n");
    printf("===================================\n\n");

    DynamicArray* arr = createArray();

    printf("Appending 10 elements:\n");
    for (int i = 1; i <= 10; i++) {
        printf("Append %d: ", i);
        append(arr, i);
        printf("size=%d, capacity=%d\n", arr->size, arr->capacity);
    }

    /*
     * Analysis (분석):
     * - Most appends are O(1)
     * - Resize happens at sizes 1, 2, 4, 8, ... (powers of 2)
     * - Cost of resize at size n: O(n)
     *
     * For n appends:
     * - Total resize cost: 1 + 2 + 4 + 8 + ... + n = O(2n) = O(n)
     * - Total simple append cost: O(n)
     * - Total cost: O(n) + O(n) = O(n)
     *
     * Amortized cost per append: O(n) / n = O(1)
     */

    printf("\nAmortized Analysis Result:\n");
    printf("- Worst case per operation: O(n) when resizing\n");
    printf("- Amortized cost per operation: O(1)\n");

    freeArray(arr);

    return 0;
}
```

### Example
> See `examples/` folder

---

## Summary (요약)

| Concept | Description |
|---------|-------------|
| Algorithm | Step-by-step procedure to solve a problem |
| Time Complexity | Measures running time growth vs input size |
| Space Complexity | Measures memory usage vs input size |
| Big-O Notation | Upper bound of algorithm growth rate |
| Best/Average/Worst Case | Different scenarios for analysis |
| Amortized Analysis | Average cost over sequence of operations |

### Common Complexities Quick Reference

| Complexity | Name | Example Algorithms |
|------------|------|-------------------|
| O(1) | Constant | Array access, hash lookup |
| O(log n) | Logarithmic | Binary search |
| O(n) | Linear | Linear search, single pass |
| O(n log n) | Linearithmic | Merge sort, heap sort |
| O(n^2) | Quadratic | Bubble sort, selection sort |
| O(2^n) | Exponential | Naive recursive solutions |

---

## Practice Exercises (연습 문제)

1. Analyze the time complexity of finding duplicates in an array using nested loops.
2. Compare the time complexity of different approaches to calculate power(x, n).
3. Analyze the space complexity of recursive vs iterative tree traversal.
4. Calculate the amortized time complexity of a stack with dynamic array.
5. Determine the Big-O complexity of common string operations (reverse, concatenate, search).
