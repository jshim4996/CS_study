# Chapter 11. Searching (탐색)

> Understanding fundamental search algorithms: linear search, binary search, and their implementations in C.

---

## Linear Search (선형 탐색)

### Concept (개념)

**Linear Search** (선형 탐색) is the simplest search algorithm that sequentially checks each element in a list until the target element is found or the entire list has been traversed.

**Key Characteristics:**
- Works on both sorted and unsorted arrays
- Time Complexity: O(n)
- Space Complexity: O(1)
- No preprocessing required

```c
#include <stdio.h>

// Linear Search - Returns index of target, or -1 if not found
// 선형 탐색 - 찾으면 인덱스 반환, 없으면 -1 반환
int linearSearch(int arr[], int n, int target) {
    for (int i = 0; i < n; i++) {
        if (arr[i] == target) {
            return i;  // Found at index i
        }
    }
    return -1;  // Not found
}

// Linear Search with count of comparisons
// 비교 횟수를 세는 선형 탐색
int linearSearchWithCount(int arr[], int n, int target, int* comparisons) {
    *comparisons = 0;
    for (int i = 0; i < n; i++) {
        (*comparisons)++;
        if (arr[i] == target) {
            return i;
        }
    }
    return -1;
}

int main(void) {
    int arr[] = {64, 34, 25, 12, 22, 11, 90, 45};
    int n = sizeof(arr) / sizeof(arr[0]);
    int comparisons;

    printf("=== Linear Search (선형 탐색) ===\n\n");

    printf("Array: [");
    for (int i = 0; i < n; i++) {
        printf("%d%s", arr[i], i < n - 1 ? ", " : "");
    }
    printf("]\n\n");

    // Search for existing element
    int target = 22;
    int result = linearSearchWithCount(arr, n, target, &comparisons);
    printf("Searching for %d:\n", target);
    printf("  Result: Found at index %d\n", result);
    printf("  Comparisons: %d\n\n", comparisons);

    // Search for non-existing element
    target = 100;
    result = linearSearchWithCount(arr, n, target, &comparisons);
    printf("Searching for %d:\n", target);
    printf("  Result: %s\n", result == -1 ? "Not found" : "Found");
    printf("  Comparisons: %d\n", comparisons);

    return 0;
}
```

**Visualization of Linear Search:**
```
Array: [64, 34, 25, 12, 22, 11, 90, 45]
        [0] [1] [2] [3] [4] [5] [6] [7]

Searching for 22:

Step 1: Compare arr[0]=64 with 22 -> Not equal
        [64] 34  25  12  22  11  90  45
         ^

Step 2: Compare arr[1]=34 with 22 -> Not equal
         64 [34] 25  12  22  11  90  45
             ^

Step 3: Compare arr[2]=25 with 22 -> Not equal
         64  34 [25] 12  22  11  90  45
                 ^

Step 4: Compare arr[3]=12 with 22 -> Not equal
         64  34  25 [12] 22  11  90  45
                     ^

Step 5: Compare arr[4]=22 with 22 -> FOUND!
         64  34  25  12 [22] 11  90  45
                         ^

Result: Found at index 4 (5 comparisons)
```

### Time Complexity Analysis

| Case | Comparisons | Time Complexity |
|------|-------------|-----------------|
| Best Case | 1 | O(1) |
| Worst Case | n | O(n) |
| Average Case | n/2 | O(n) |

---

## Binary Search (이진 탐색)

### Concept (개념)

**Binary Search** (이진 탐색) is an efficient search algorithm that works on **sorted arrays** by repeatedly dividing the search interval in half.

**Key Characteristics:**
- **Requires sorted array** (정렬된 배열 필요)
- Time Complexity: O(log n)
- Space Complexity: O(1) iterative, O(log n) recursive
- Much faster than linear search for large datasets

### Binary Search Condition (이진 탐색 조건)

**IMPORTANT:** Binary search only works on **sorted arrays**!

```
Prerequisites for Binary Search:
1. Array must be SORTED (ascending or descending)
2. Random access to elements (array, not linked list)
3. Comparison operation must be defined for elements
```

### Iterative Implementation (반복적 구현)

```c
#include <stdio.h>

// Binary Search - Iterative Version
// 이진 탐색 - 반복 버전
int binarySearchIterative(int arr[], int n, int target) {
    int left = 0;
    int right = n - 1;

    while (left <= right) {
        // Prevent integer overflow: use left + (right - left) / 2
        int mid = left + (right - left) / 2;

        if (arr[mid] == target) {
            return mid;  // Found
        } else if (arr[mid] < target) {
            left = mid + 1;  // Search right half
        } else {
            right = mid - 1;  // Search left half
        }
    }

    return -1;  // Not found
}

// Binary Search with detailed trace
int binarySearchWithTrace(int arr[], int n, int target) {
    int left = 0;
    int right = n - 1;
    int step = 1;

    printf("Searching for %d in sorted array:\n", target);
    printf("Array indices: ");
    for (int i = 0; i < n; i++) printf("%3d ", i);
    printf("\nArray values:  ");
    for (int i = 0; i < n; i++) printf("%3d ", arr[i]);
    printf("\n\n");

    while (left <= right) {
        int mid = left + (right - left) / 2;

        printf("Step %d: left=%d, right=%d, mid=%d\n", step, left, right, mid);
        printf("        arr[mid]=%d, target=%d\n", arr[mid], target);

        if (arr[mid] == target) {
            printf("        -> FOUND at index %d!\n", mid);
            return mid;
        } else if (arr[mid] < target) {
            printf("        -> %d < %d, search RIGHT half\n", arr[mid], target);
            left = mid + 1;
        } else {
            printf("        -> %d > %d, search LEFT half\n", arr[mid], target);
            right = mid - 1;
        }
        step++;
        printf("\n");
    }

    printf("-> Not found!\n");
    return -1;
}

int main(void) {
    // Array MUST be sorted for binary search!
    int arr[] = {11, 12, 22, 25, 34, 45, 64, 78, 90, 95};
    int n = sizeof(arr) / sizeof(arr[0]);

    printf("=== Binary Search - Iterative (반복적 이진 탐색) ===\n\n");

    // Test case 1: Element exists
    binarySearchWithTrace(arr, n, 45);

    printf("\n-----------------------------------\n\n");

    // Test case 2: Element does not exist
    binarySearchWithTrace(arr, n, 50);

    return 0;
}
```

**Visualization of Binary Search:**
```
Sorted Array: [11, 12, 22, 25, 34, 45, 64, 78, 90, 95]
Indices:       [0] [1] [2] [3] [4] [5] [6] [7] [8] [9]

Searching for 45:

Step 1: left=0, right=9, mid=4
        [11  12  22  25  34] 45  64  78  90  95
         L           M                       R
        arr[4]=34 < 45 -> Search RIGHT half

Step 2: left=5, right=9, mid=7
         11  12  22  25  34 [45  64  78] 90  95
                             L      M        R
        arr[7]=78 > 45 -> Search LEFT half

Step 3: left=5, right=6, mid=5
         11  12  22  25  34 [45  64] 78  90  95
                             L=M R
        arr[5]=45 == 45 -> FOUND!

Result: Found at index 5 (3 comparisons)

Comparison: Linear search would need 6 comparisons!
```

### Recursive Implementation (재귀적 구현)

```c
#include <stdio.h>

// Binary Search - Recursive Version
// 이진 탐색 - 재귀 버전
int binarySearchRecursive(int arr[], int left, int right, int target) {
    // Base case: element not found
    if (left > right) {
        return -1;
    }

    int mid = left + (right - left) / 2;

    // Found the target
    if (arr[mid] == target) {
        return mid;
    }

    // Target is in left half
    if (arr[mid] > target) {
        return binarySearchRecursive(arr, left, mid - 1, target);
    }

    // Target is in right half
    return binarySearchRecursive(arr, mid + 1, right, target);
}

// Wrapper function for cleaner interface
int binarySearch(int arr[], int n, int target) {
    return binarySearchRecursive(arr, 0, n - 1, target);
}

// Recursive with depth tracking
int binarySearchRecursiveTrace(int arr[], int left, int right, int target, int depth) {
    // Print current state
    for (int i = 0; i < depth; i++) printf("  ");
    printf("Depth %d: left=%d, right=%d", depth, left, right);

    if (left > right) {
        printf(" -> NOT FOUND\n");
        return -1;
    }

    int mid = left + (right - left) / 2;
    printf(", mid=%d, arr[mid]=%d\n", mid, arr[mid]);

    if (arr[mid] == target) {
        for (int i = 0; i < depth; i++) printf("  ");
        printf("-> FOUND at index %d\n", mid);
        return mid;
    }

    if (arr[mid] > target) {
        for (int i = 0; i < depth; i++) printf("  ");
        printf("-> Search LEFT (arr[mid]=%d > target=%d)\n", arr[mid], target);
        return binarySearchRecursiveTrace(arr, left, mid - 1, target, depth + 1);
    }

    for (int i = 0; i < depth; i++) printf("  ");
    printf("-> Search RIGHT (arr[mid]=%d < target=%d)\n", arr[mid], target);
    return binarySearchRecursiveTrace(arr, mid + 1, right, target, depth + 1);
}

int main(void) {
    int arr[] = {11, 12, 22, 25, 34, 45, 64, 78, 90, 95};
    int n = sizeof(arr) / sizeof(arr[0]);

    printf("=== Binary Search - Recursive (재귀적 이진 탐색) ===\n\n");

    printf("Array: [");
    for (int i = 0; i < n; i++) {
        printf("%d%s", arr[i], i < n - 1 ? ", " : "");
    }
    printf("]\n\n");

    printf("Searching for 64:\n");
    binarySearchRecursiveTrace(arr, 0, n - 1, 64, 0);

    printf("\nSearching for 30:\n");
    binarySearchRecursiveTrace(arr, 0, n - 1, 30, 0);

    return 0;
}
```

**Recursion Tree Visualization:**
```
Searching for 64 in [11, 12, 22, 25, 34, 45, 64, 78, 90, 95]:

binarySearch(arr, 0, 9, 64)
    mid = 4, arr[4] = 34
    34 < 64, search right
    |
    +-> binarySearch(arr, 5, 9, 64)
            mid = 7, arr[7] = 78
            78 > 64, search left
            |
            +-> binarySearch(arr, 5, 6, 64)
                    mid = 5, arr[5] = 45
                    45 < 64, search right
                    |
                    +-> binarySearch(arr, 6, 6, 64)
                            mid = 6, arr[6] = 64
                            FOUND! Return 6
```

---

## Recursive vs Iterative Implementation (재귀 vs 반복 구현)

### Comparison (비교)

```c
#include <stdio.h>
#include <time.h>
#include <stdlib.h>

// Iterative Binary Search
int binarySearchIterative(int arr[], int n, int target) {
    int left = 0, right = n - 1;
    while (left <= right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] == target) return mid;
        else if (arr[mid] < target) left = mid + 1;
        else right = mid - 1;
    }
    return -1;
}

// Recursive Binary Search
int binarySearchRecursive(int arr[], int left, int right, int target) {
    if (left > right) return -1;
    int mid = left + (right - left) / 2;
    if (arr[mid] == target) return mid;
    if (arr[mid] > target)
        return binarySearchRecursive(arr, left, mid - 1, target);
    return binarySearchRecursive(arr, mid + 1, right, target);
}

int main(void) {
    printf("=== Recursive vs Iterative Binary Search ===\n\n");

    printf("| Aspect           | Iterative      | Recursive       |\n");
    printf("|------------------|----------------|------------------|\n");
    printf("| Time Complexity  | O(log n)       | O(log n)         |\n");
    printf("| Space Complexity | O(1)           | O(log n) stack   |\n");
    printf("| Stack Overflow   | Not possible   | Possible (deep)  |\n");
    printf("| Code Clarity     | More verbose   | More elegant     |\n");
    printf("| Performance      | Slightly faster| Function overhead|\n");
    printf("| Tail Recursion   | N/A            | Can optimize     |\n\n");

    // Performance comparison
    int n = 10000000;  // 10 million elements
    int* arr = (int*)malloc(n * sizeof(int));

    // Initialize sorted array
    for (int i = 0; i < n; i++) {
        arr[i] = i * 2;  // Even numbers: 0, 2, 4, 6, ...
    }

    int target = 9999998;  // Near end of array
    int iterations = 1000000;

    // Test iterative
    clock_t start = clock();
    for (int i = 0; i < iterations; i++) {
        binarySearchIterative(arr, n, target);
    }
    clock_t end = clock();
    double iterTime = (double)(end - start) / CLOCKS_PER_SEC;

    // Test recursive
    start = clock();
    for (int i = 0; i < iterations; i++) {
        binarySearchRecursive(arr, 0, n - 1, target);
    }
    end = clock();
    double recTime = (double)(end - start) / CLOCKS_PER_SEC;

    printf("Performance Test (n=%d, iterations=%d):\n", n, iterations);
    printf("  Iterative: %.4f seconds\n", iterTime);
    printf("  Recursive: %.4f seconds\n", recTime);

    free(arr);
    return 0;
}
```

### When to Use Which (언제 무엇을 사용할까)

```
Iterative Binary Search 사용:
  - Memory is limited (스택 공간 제한)
  - Very large arrays (매우 큰 배열)
  - Performance-critical applications (성능 중요)
  - Embedded systems (임베디드 시스템)

Recursive Binary Search 사용:
  - Code readability is priority (가독성 중요)
  - Teaching/learning purposes (교육 목적)
  - Compiler supports tail call optimization (꼬리 재귀 최적화)
  - Problem naturally fits recursion (자연스러운 재귀 구조)
```

---

## Finding Boundaries (경계 찾기)

### Lower Bound and Upper Bound

```c
#include <stdio.h>

// Lower Bound: First element >= target
// 하한: target 이상인 첫 번째 요소
int lowerBound(int arr[], int n, int target) {
    int left = 0, right = n;

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] < target) {
            left = mid + 1;
        } else {
            right = mid;
        }
    }
    return left;
}

// Upper Bound: First element > target
// 상한: target 초과인 첫 번째 요소
int upperBound(int arr[], int n, int target) {
    int left = 0, right = n;

    while (left < right) {
        int mid = left + (right - left) / 2;
        if (arr[mid] <= target) {
            left = mid + 1;
        } else {
            right = mid;
        }
    }
    return left;
}

// Count occurrences of target using bounds
int countOccurrences(int arr[], int n, int target) {
    return upperBound(arr, n, target) - lowerBound(arr, n, target);
}

int main(void) {
    int arr[] = {1, 2, 2, 2, 3, 4, 4, 5, 5, 5, 5, 6};
    int n = sizeof(arr) / sizeof(arr[0]);

    printf("=== Lower Bound and Upper Bound ===\n\n");

    printf("Array: [");
    for (int i = 0; i < n; i++) {
        printf("%d%s", arr[i], i < n - 1 ? ", " : "");
    }
    printf("]\n");
    printf("Index:  ");
    for (int i = 0; i < n; i++) {
        printf("%d  ", i);
    }
    printf("\n\n");

    int target = 2;
    printf("Target: %d\n", target);
    printf("  Lower Bound (first >= %d): index %d\n", target, lowerBound(arr, n, target));
    printf("  Upper Bound (first > %d):  index %d\n", target, upperBound(arr, n, target));
    printf("  Count of %d: %d\n\n", target, countOccurrences(arr, n, target));

    target = 5;
    printf("Target: %d\n", target);
    printf("  Lower Bound (first >= %d): index %d\n", target, lowerBound(arr, n, target));
    printf("  Upper Bound (first > %d):  index %d\n", target, upperBound(arr, n, target));
    printf("  Count of %d: %d\n", target, countOccurrences(arr, n, target));

    return 0;
}
```

**Visualization:**
```
Array: [1, 2, 2, 2, 3, 4, 4, 5, 5, 5, 5, 6]
Index:  0  1  2  3  4  5  6  7  8  9 10 11

For target = 2:
        [1, 2, 2, 2, 3, 4, 4, 5, 5, 5, 5, 6]
            ^        ^
            |        |
      lower_bound  upper_bound
         (1)          (4)

Count = upper_bound - lower_bound = 4 - 1 = 3

For target = 5:
        [1, 2, 2, 2, 3, 4, 4, 5, 5, 5, 5, 6]
                              ^           ^
                              |           |
                        lower_bound   upper_bound
                             (7)        (11)

Count = 11 - 7 = 4
```

---

## Time Complexity Comparison (시간 복잡도 비교)

```
Number of Elements vs Maximum Comparisons:

n (elements) | Linear O(n) | Binary O(log n)
-------------|-------------|----------------
        10   |         10  |              4
       100   |        100  |              7
     1,000   |      1,000  |             10
    10,000   |     10,000  |             14
   100,000   |    100,000  |             17
 1,000,000   |  1,000,000  |             20

Visual comparison:
Linear Search:  n comparisons
|================================================| 1,000,000

Binary Search: log2(n) comparisons
|==| 20

Binary search is exponentially faster for large datasets!
```

---

## Summary (요약)

| Topic | Key Points |
|-------|------------|
| Linear Search | O(n), works on unsorted arrays, simple implementation |
| Binary Search | O(log n), requires sorted array, divide and conquer |
| Binary Search Condition | Array must be sorted! |
| Iterative vs Recursive | Both O(log n) time; iterative uses O(1) space, recursive uses O(log n) stack |
| Lower/Upper Bound | Useful for counting and range queries |

### Algorithm Selection Guide (알고리즘 선택 가이드)

```
Is the array sorted?
        |
       No -----> Use Linear Search O(n)
        |           OR
       Yes          Sort first (if searching multiple times)
        |
        v
Use Binary Search O(log n)
        |
        |----> Single search? -> Iterative (less overhead)
        |
        |----> Complex condition? -> Recursive (cleaner code)
        |
        +----> Finding range? -> Lower/Upper Bound
```

---

## Practice Exercises (연습 문제)

1. Implement a function to find the first and last occurrence of an element in a sorted array.
2. Write binary search for a rotated sorted array (e.g., [4,5,6,7,0,1,2]).
3. Implement binary search on a 2D sorted matrix.
4. Find the square root of a number using binary search.
5. Search for an element in a nearly sorted array (element may be at i-1, i, or i+1).
